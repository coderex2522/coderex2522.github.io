


Baton与信号量不同的地方，仅支持单词post/wait操作



# std::is_invocable 

std::is_invocable 是 C++17 中引入的一个类型特性（定义在 <type_traits> 头文件中），用于在编译时检查某个对象或表达式是否可以被调用（即是否可以通过给定参数类型进行调用）。它在模板元编程中非常有用，尤其是与 SFINAE（替换失败不是错误）结合时。

语法
cpp
#include <type_traits>

template<class Fn, class... Args>
struct is_invocable;

// 或直接使用辅助变量模板（C++17 起）
template<class Fn, class... Args>
inline constexpr bool is_invocable_v = std::is_invocable<Fn, Args...>::value;
参数
Fn：要检查的可调用对象类型（例如函数、函数指针、lambda、仿函数等）。
Args...：参数类型列表，表示调用时传入的参数类型。
返回值
若 Fn 可以用 Args... 类型的参数调用（即 std::invoke 合法），则 value 为 true。
否则，value 为 false。


# folly::invoke_result_t
folly::invoke_result_t 是 Facebook 的 Folly 库中提供的一个类型萃取（type trait），用于确定调用某个可调用对象（如函数、函数指针、成员函数指针、函数对象等）后的返回类型。它的功能类似于 C++17 标准库中的 std::invoke_result_t，但可能在实现细节或兼容性上有所扩展。

语法解析
cpp
template <typename Fn, typename... Args>
using invoke_result_t = typename invoke_result<Fn, Args...>::type;
作用：推断调用 Fn 并传入参数 Args... 后的返回类型。
参数：
Fn：可调用对象的类型（如函数、成员函数指针、函数对象等）。
Args...：调用时传入的参数类型列表。
返回值：通过 invoke_result<Fn, Args...>::type 获取最终的返回类型。
