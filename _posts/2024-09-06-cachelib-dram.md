---
layout:     post
title:      "【CacheLib】Ram Cache"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:

    - CacheLib
---
# CacheTraits
```cpp
struct LruCacheTrait {
  using MMType = MMLru;
  using AccessType = ChainedHashTable;
  using AccessTypeLocks = SharedMutexBuckets;
};

struct LruCacheWithSpinBucketsTrait {
  using MMType = MMLru;
  using AccessType = ChainedHashTable;
  using AccessTypeLocks = SpinBuckets;
};

struct Lru2QCacheTrait {
  using MMType = MM2Q;
  using AccessType = ChainedHashTable;
  using AccessTypeLocks = SharedMutexBuckets;
};

struct TinyLFUCacheTrait {
  using MMType = MMTinyLFU;
  using AccessType = ChainedHashTable;
  using AccessTypeLocks = SharedMutexBuckets;
};

```

# AccessContainer
The AccessContainer is `an intrusive hook based container` and supports a standard interface that works with CacheAllocator. 

It could be viewed as `a large hash table` that serves as the index of the entire CacheLib cache.
- An item's accessibility is controlled only by the access container. So an item is accessible (can be returned by find request) only after it is `inserted into the access container`.
- It is no longer accessible once it is `removed from the access container`.    

There is `only one access container` for the entire CacheLib cache (see `accessContainer_` in `cachelib/allocator/CacheAllocator.h`).

## Major API functions
- find
- insert
- remove

## ChainedHashTtable
This is our `only implementation of access container`.