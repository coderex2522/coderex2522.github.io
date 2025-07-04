---
layout:     post
title:      "Folly future promise"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:
    - folly
---

## 从promise中获取future
promise可以获取future，有两种future
- SemiFuture
- future

其中二者区别主要是在是否设置了executor。SemiFuture未设置executor，SemiFuture通过via设置executor后就变成了future。设置executor含义是指定callback在哪个线程池中执行。

```cpp
 template <class T>
SemiFuture<T> Promise<T>::getSemiFuture() {
  if (retrieved_) {
    throw_exception<FutureAlreadyRetrieved>();
  }
  retrieved_ = true;
  return SemiFuture<T>(&getCore());
}

template <class T>
Future<T> Promise<T>::getFuture() {
  // An InlineExecutor approximates the old behaviour of continuations
  // running inine on setting the value of the promise.
  return getSemiFuture().via(&InlineExecutor::instance());
}
```

## 添加callback
添加callback主要有四个方法，分别是
- thenValue
- thenTry
- thenValueInline
- thenTryInline

区别在于
- Value和Try主要区别是callback参数不同，value是普通类型，try是对value的封装，同时有可能是还有异常。
- Inline和非inline区别时设置callback时是允许立即执行，还是放到线程池中执行，可参考执行方式。


```cpp
template <class T>
template <typename F>
Future<typename futures::detail::tryCallableResult<T, F>::value_type>
Future<T>::thenTry(F&& func) && {
  auto lambdaFunc = [f = static_cast<F&&>(func)](
                        folly::Executor::KeepAlive<>&&,
                        folly::Try<T>&& t) mutable {
    return static_cast<F&&>(f)(std::move(t));
  };
  using R = futures::detail::tryExecutorCallableResult<T, decltype(lambdaFunc)>;
  return this->thenImplementation(
      std::move(lambdaFunc), R{}, futures::detail::InlineContinuation::forbid);
}

template <class T>
template <typename F>
Future<typename futures::detail::tryCallableResult<T, F>::value_type>
Future<T>::thenTryInline(F&& func) && {
  auto lambdaFunc = [f = static_cast<F&&>(func)](
                        folly::Executor::KeepAlive<>&&,
                        folly::Try<T>&& t) mutable {
    return static_cast<F&&>(f)(std::move(t));
  };
  using R = futures::detail::tryExecutorCallableResult<T, decltype(lambdaFunc)>;
  return this->thenImplementation(
      std::move(lambdaFunc), R{}, futures::detail::InlineContinuation::permit);
}

template <class T>
template <typename F>
Future<typename futures::detail::valueCallableResult<T, F>::value_type>
Future<T>::thenValue(F&& func) && {
  auto lambdaFunc = [f = static_cast<F&&>(func)](
                        Executor::KeepAlive<>&&, folly::Try<T>&& t) mutable {
    return futures::detail::wrapInvoke(std::move(t), static_cast<F&&>(f));
  };
  using R = futures::detail::tryExecutorCallableResult<T, decltype(lambdaFunc)>;
  return this->thenImplementation(
      std::move(lambdaFunc), R{}, futures::detail::InlineContinuation::forbid);
}

template <class T>
template <typename F>
Future<typename futures::detail::valueCallableResult<T, F>::value_type>
Future<T>::thenValueInline(F&& func) && {
  auto lambdaFunc = [f = static_cast<F&&>(func)](
                        Executor::KeepAlive<>&&, folly::Try<T>&& t) mutable {
    return futures::detail::wrapInvoke(std::move(t), static_cast<F&&>(f));
  };
  using R = futures::detail::tryExecutorCallableResult<T, decltype(lambdaFunc)>;
  return this->thenImplementation(
      std::move(lambdaFunc), R{}, futures::detail::InlineContinuation::permit);
}
```

对于thenImplementation来说，根据callback返回值类型不同，有两个重载：

- 当返回普通类型时，执行callback返回结果时，直接设置下一层promise的result。
- 当返回类型为future时，执行完callback，替换下一层的promise的core到callback返回的future的core，同时将下一层的future中设置的callback传递给这一层返回的future，这样相当于在执行时，对返回的future进行了替换，替换为callback返回的future。 

下面针对上面两个类型的callback详细介绍


# collect
collect方法参数是一个future的list，返回是一个SemiFuture，其result所有future的result构成的元组。其实现的功能是，新创建一个future，以参数中的所有future的结果作为其输入，即新建的future仅在参数中所有future获取到结果才执行自身的callback。这是DAG中重要的一环，即某个算子依赖多个算子，在多个算子执行完成时才能够执行，该功能即由collect来实现。

下面来看一下其具体实现：
```cpp
template <typename... Fs>
SemiFuture<std::tuple<typename remove_cvref_t<Fs>::value_type...>> collect(
    Fs&&... fs) {
  using Result = std::tuple<typename remove_cvref_t<Fs>::value_type...>;
  struct Context {
    ~Context() {
      if (!threw.load(std::memory_order_relaxed)) {
        // if any of the input futures were off the end of a weakRef(), the
        // logic added in setCallback_ will not execute as an executor
        // weakRef() drops all callbacks added silently without executing them
        auto brokenPromise = false;
        folly::for_each(results, [&](auto& result) {
          if (!result.hasValue() && !result.hasException()) {
            brokenPromise = true;
          }
        });

        if (brokenPromise) {
          p.setException(BrokenPromise{pretty_name<Result>()});
        } else {
          p.setValue(unwrapTryTuple(std::move(results)));
        }
      }
    }
    Promise<Result> p;
    std::tuple<Try<typename remove_cvref_t<Fs>::value_type>...> results;
    std::atomic<bool> threw{false};
  };
  
  // 该步骤时获取执行器中所有defer执行器，这里不考虑该逻辑
  std::vector<futures::detail::DeferredWrapper> executors;
  futures::detail::stealDeferredExecutorsVariadic(executors, fs...);

  auto ctx = std::make_shared<Context>();
  futures::detail::foreach(
      [&](auto i, auto&& f) {
        f.setCallback_([i, ctx](Executor::KeepAlive<>&&, auto&& t) {
          if (t.hasException()) {
            if (!ctx->threw.exchange(true, std::memory_order_relaxed)) {
              ctx->p.setException(std::move(t.exception()));
            }
          } else if (!ctx->threw.load(std::memory_order_relaxed)) {
            std::get<i.value>(ctx->results) = std::move(t);
          }
        });
      },
      static_cast<Fs&&>(fs)...);

  auto future = ctx->p.getSemiFuture();
  // 不考虑该逻辑，不考虑defer执行器
  if (!executors.empty()) {
    auto work = [](Try<typename decltype(future)::value_type>&& t) {
      return std::move(t).value();
    };
    future = std::move(future).defer(work);
    const auto& deferredExecutor = futures::detail::getDeferredExecutor(future);
    deferredExecutor->setNestedExecutors(std::move(executors));
  }
  return future;
}
```
通过看代码，我们可以看到，其实现就是利用shared_ptr的特性，即只在引用计数为0时，才执行析构函数。这样，我们让参数中所有folly均通过callback持有shared_ptr，让所有future执行callback结束时，就会执行析构函数，自动将shared_ptr的引用计数减一，直到所有的future均执行完成callback，就会最终析构shared_ptr，这时在析构函数中对新建的promise设置result，这样新建的promise就依赖所有参数中future执行结果，保证在所有future的callback执行·完成才执行。

其中Context就充当该shared_ptr的数据。对每个传递的future设置callback为将结果写到Context中对应的位置（或者抛出异常）。

在参数中所有future的callback执行完成后，就获取到了所有folly的结果，执行Context的析构函数，判断是否有异常产生，如果没有异常产生，则设置下一层的promise，完成计算。对应返回的future来说，如果后面还有别的链式执行逻辑，则会在这里被设置result后继续执行，如果没有，则用户直接获取到结果。


# class SharedPromise
上面介绍的collect解决了一个算子依赖多个算子的情况，但还有另一个情况，就是多个算了依赖了同一个算子。这部分则是通过SharedPromise类来实现，其实现逻辑也相对简单，就是其会持有一个promise的list，当某个算子要依赖该算子时，就会将list增加一个promise，这样依赖其的算子就会获得一个future，当该算子执行完成后，会对该list中所有promise设置result，这样持有该算子future的·所有算子都可以继续执行了。

我们来看其具体实现，仅看核心数据和接口：
```cpp
template <class T>
class SharedPromise {
 public:
 	SemiFuture<T> getSemiFuture() const;
  void setTry(Try<T>&& t);
  mutable Mutex mutex_;
  mutable Defaulted<size_t> size_;
  Defaulted<Try<T>> try_;
  mutable std::vector<Promise<T>> promises_;
  std::function<void(exception_wrapper const&)> interruptHandler_;
};

template <class T>
SemiFuture<T> SharedPromise<T>::getSemiFuture() const {
  std::lock_guard<std::mutex> g(mutex_);
  size_.value++;
  if (hasResult()) {
    return makeFuture<T>(Try<T>(try_.value));
  } else {
    promises_.emplace_back();
    if (interruptHandler_) {
      promises_.back().setInterruptHandler(interruptHandler_);
    }
    return promises_.back().getSemiFuture();
  }
}

template <class T>
void SharedPromise<T>::setTry(Try<T>&& t) {
  std::vector<Promise<T>> promises;
  
  // 不能重复设置result
  {
    std::lock_guard<std::mutex> g(mutex_);
    if (hasResult()) {
      throw_exception<PromiseAlreadySatisfied>();
    }
    try_.value = std::move(t);
    promises.swap(promises_);
  }

  for (auto& p : promises) {
    p.setTry(Try<T>(try_.value));
  }
}
```