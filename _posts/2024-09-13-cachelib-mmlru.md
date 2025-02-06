---
layout:     post
title:      "【CacheLib实现】MMLru"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:

    - CacheLib
    - 代码学习
---

MMLru 只包含了两个类型的定义
* `Config`
* `Container`

# Config
```cpp
struct Config {
    // threshold value in seconds to compare with a node's update time to
    // determine if we need to update the position of the node in the linked
    // list. By default this is 60s to reduce the contention on the lru lock.
    uint32_t defaultLruRefreshTime{60};
    uint32_t lruRefreshTime{defaultLruRefreshTime};

    // ratio of LRU refresh time to the tail age. If a refresh time computed
    // according to this ratio is larger than lruRefreshtime, we will adopt
    // this one instead of the lruRefreshTime set.
    double lruRefreshRatio{0.};

    // whether the lru needs to be updated on writes for recordAccess. If
    // false, accessing the cache for writes does not promote the cached item
    // to the head of the lru.
    bool updateOnWrite{false};

    // whether the lru needs to be updated on reads for recordAccess. If
    // false, accessing the cache for reads does not promote the cached item
    // to the head of the lru.
    bool updateOnRead{true};

    // whether to tryLock or lock the lru lock when attempting promotion on
    // access. If set, and tryLock fails, access will not result in promotion.
    bool tryLockUpdate{false};

    // By default insertions happen at the head of the LRU. If we need
    // insertions at the middle of lru we can adjust this to be a non-zero.
    // Ex: lruInsertionPointSpec = 1, we insert at the middle (1/2 from end)
    //     lruInsertionPointSpec = 2, we insert at a point 1/4th from tail
    uint8_t lruInsertionPointSpec{0};

    // Minimum interval between reconfigurations. If 0, reconfigure is never
    // called.
    std::chrono::seconds mmReconfigureIntervalSecs{};

    // Whether to use combined locking for withEvictionIterator.
    bool useCombinedLockForIterators{false};
}
```
* 

* lruInsertionPointSpec: 表示新进节点的插入位置
  * 会 1/(2 ^ lruInsertionPointSpec) 的位置插入节点
  * 只对新加入的节点生效，如果节点在list中被命中时，默认还是添加到head of list


# Container
```cpp
template <typename T>
using Hook = DListHook<T>;

template <typename T, Hook<T> T::*HookPtr>
struct Container {...}
