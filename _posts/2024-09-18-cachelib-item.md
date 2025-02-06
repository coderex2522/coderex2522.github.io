---
layout:     post
title:      "【CacheLib实现】Cache CacheItem"
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

# CacheItem vs ChainedItem
两者的区别: 目前有两种可以被缓存的Item。
- ChainedItem是一个依赖于regular item的item
- ChainedItem没有真实的Key，而是与一个regular item（其父item）相关联
- 多个ChainedItem可以链接到同一个regular item上，它们可以一起缓存比单个Item更大的数据

# CacheItem
```cpp
class CacheItem {
...


using ReadHandle = detail::ReadHandleImpl<CacheItem>;
using WriteHandle = detail::WriteHandleImpl<CacheItem>;


...
};
```

# CacheAllocator
### getMMContainer
```cpp
template <typename CacheTrait>
typename CacheAllocator<CacheTrait>::MMContainer&
CacheAllocator<CacheTrait>::getMMContainer(const Item& item) const noexcept {
  const auto allocInfo =
      allocator_->getAllocInfo(static_cast<const void*>(&item));
  return getMMContainer(allocInfo.poolId, allocInfo.classId);
}

template <typename CacheTrait>
typename CacheAllocator<CacheTrait>::MMContainer&
CacheAllocator<CacheTrait>::getMMContainer(PoolId pid,
                                           ClassId cid) const noexcept {
  XDCHECK_LT(static_cast<size_t>(pid), mmContainers_.size());
  XDCHECK_LT(static_cast<size_t>(cid), mmContainers_[pid].size());
  return *mmContainers_[pid][cid];
}
```

### getItemLruType
```cpp
MMType::LruType getItemLruType(const Item& item) const {
    return getMMContainer(item).getLruType(item);
}

```