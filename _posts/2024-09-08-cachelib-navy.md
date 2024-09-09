---
layout:     post
title:      "【CacheLib】Navy"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:

    - CacheLib
---

# 特性
- Manages terabytes of data 管理TB级别数据
- Efficiently supports both `small` (~100 bytes) and `large` (~10KBs to ~1 MBs) objects
- Sync and async API (supports custom executors for async ops) 同步或者异步的API
- Cache persistence on safe shutdown 缓存持久化
- Comprehensive cache stats 丰富的stats
- Different admission policies to optimize
  - write endurance
  - hit ratio
  - IOPS
- Supports Direct IO and raw devices
- Supports async IO with either `io_uring` or `libaio`
- Supports Flexible Data Placement (FDP) to improve `write endurance of the device`
  
# 设计目标
Navy 首要(over-arching)的设计目标
1. efficient caching for billions of `small` (\<1KB) and millions of `large` objects (1KB - 16MB) on SSDs. 高效地缓存小item / 大item
2. read optimized point lookups 点查优化
3. low DRAM overhead
Since Navy is designed for a cache, it chooses to sacrifice the durability of data when it enables the accomplishment of the goals above.
由于写入耐久性是非易失性存储（NVM）的一个限制，Navy的设计也针对写入耐久性进行了优化。


# 架构
![1](/img/cachelib/navy.png)



# Engine Driver

## NVM Admission Policy

## Navy Admission Policy


# Engine
The Engine
- is the core of Navy
- implements actual NVM caching algorithms

![1](/img/cachelib/engine.png)

为了支持大范围的item size, Navy 支持两种类型的Engine
- BigHash
- BlockCache
两种类型的实例会配套成 `EnginePair`


