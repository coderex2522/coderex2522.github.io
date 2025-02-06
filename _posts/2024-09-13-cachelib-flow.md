---
layout:     post
title:      "【CacheLib实现】Cache Flow"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:

    - CacheLib
    - 代码学习
---

整理examples/single_tier_cache/main.cpp的代码，理清Cache的大致流程

```cpp
using Cache = cachelib::LruAllocator;
// - CacheAllocator.h
using LruAllocator = CacheAllocator<LruCacheTrait>; 
// CacheTraits.h
struct LruCacheTrait {
  using MMType = MMLru;
  using AccessType = ChainedHashTable;
  using AccessTypeLocks = SharedMutexBuckets;
};

// 配置
using CacheConfig = typename Cache::Config;
// CacheAllocator.h
using CacheT = CacheAllocator<CacheTrait>;
using Config = CacheAllocatorConfig<CacheT>;


