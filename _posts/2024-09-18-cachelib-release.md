---
layout:     post
title:      "【CacheLib实现】Cache Release Flow"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:

    - CacheLib
    - 代码学习
---

整理cachelib/allocator/CacheAllocator.h的release代码

# findChainedItem
```cpp
```




# releaseBackToAllocator
```cpp
if (!it.isDrained()) {
    throw std::runtime_error(
        folly::sformat("cannot release this item: {}", it.toString()));
  }
// Chained items can only end up in this place if the user has allocated
  // memory for a chained item but has decided not to insert the chained item
  // to a parent item and instead drop the chained item handle. In this case,
  // we free the chained item directly without calling remove callback.
  if (it.isChainedItem()) {
    if (toRecycle) {
      throw std::runtime_error(
          folly::sformat("Can not recycle a chained item {}, toRecyle",
                         it.toString(), toRecycle->toString()));
    }

    allocator_->free(&it);
    return ReleaseRes::kReleased;
  }

if (it.hasChainedItem()) {
    // At this point, the parent is only accessible within this thread
    // and thus no one else can add or remove any chained items associated
    // with this parent. So we're free to go through the list and free
    // chained items one by one.
    auto headHandle = findChainedItem(it);
    ChainedItem* head = &headHandle.get()->asChainedItem();
    headHandle.reset();

    if (head == nullptr || &head->getParentItem(compressor_) != &it) {
      throw exception::ChainedItemInvalid(folly::sformat(
          "Mismatch parent pointer. This should not happen. Key: {}",
          it.getKey()));
    }

    if (!chainedItemAccessContainer_->remove(*head)) {
      throw exception::ChainedItemInvalid(folly::sformat(
          "Chained item associated with {} cannot be removed from hash table "
          "This should not happen here.",
          it.getKey()));
    }

    while (head) {
      auto next = head->getNext(compressor_);

      const auto childInfo =
          allocator_->getAllocInfo(static_cast<const void*>(head));
      (*stats_.fragmentationSize)[childInfo.poolId][childInfo.classId].sub(
          util::getFragmentation(*this, *head));

      removeFromMMContainer(*head);

      // No other thread can access any of the chained items by this point,
      // so the refcount for each chained item must be equal to 1. Since
      // we use 1 to mark an item as being linked to a parent item.
      const auto childRef = head->decRef();
      XDCHECK_EQ(0u, childRef);

      if (head == toRecycle) {
        XDCHECK(ReleaseRes::kReleased != res);
        res = ReleaseRes::kRecycled;
      } else {
        allocator_->free(head);
      }

      stats_.numChainedChildItems.dec();
      head = next;
    }
    stats_.numChainedParentItems.dec();
  }

  if (&it == toRecycle) {
    XDCHECK(ReleaseRes::kReleased != res);
    res = ReleaseRes::kRecycled;
  } else {
    XDCHECK(it.isDrained());
    allocator_->free(&it);
  }
```
1. 判断it是否为drained，不为drained直接抛异常
2. chained item只有在用户已为item分配内存但决定不将其插入父item并直接放弃chained item的句柄时，才能最终进入此处。在这种情况下，我们会直接释放chained item，而不调用移除回调。”

 


# release
```cpp
template <typename CacheTrait>
void CacheAllocator<CacheTrait>::release(Item* it, bool isNascent) {
  // decrement the reference and if it drops to 0, release it back to the
  // memory allocator, and invoke the removal callback if there is one.
  if (UNLIKELY(!it)) {
    return;
  }

  const auto ref = decRef(*it);

  if (UNLIKELY(ref == 0)) {
    const auto res =
        releaseBackToAllocator(*it, RemoveContext::kNormal, isNascent);
    XDCHECK(res == ReleaseRes::kReleased);
  }
}
```
1. 首先判断it是否为空，为空直接返回
2. 获取it的引用计数，如果为0，则调用releaseBackToAllocator释放it

# getNextCandidate
```cpp
```

# findEviction
```cpp
unsigned int searchTries = 0;
while (config_.evictionSearchTries == 0 ||
         config_.evictionSearchTries > searchTries) {
auto [candidate, toRecycle] = getNextCandidate(pid, cid, searchTries);
auto ret = releaseBackToAllocator(*candidate, RemoveContext::kEviction,
                                      /* isNascent */ false, toRecycle);
    if (ret == ReleaseRes::kRecycled) {
      return toRecycle;
    }
}
```

1. 循环调用getNextCandidate，直到找到一个可以释放的item
2. 调用releaseBackToAllocator释放item
