---
layout:     post
title:      "【论文笔记】The CacheLib Caching Engine: Design and Experiences at Scale"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:

    - CacheLib
    - 论文笔记
---
### 背景

论文笔记 《[The CacheLib Caching Engine: Design and Experiences at Scale](https://www.usenix.org/system/files/osdi20-berg.pdf)》

# Abstract
> Web services rely on caching at nearly every layer of the
system architecture. Commonly, each cache is implemented
and maintained independently by a distinct team and is highly
specialized to its function. For example, an application-data
cache would be independent from a CDN cache. However, this
approach ignores the difficult challenges that different caching
systems have in common, greatly increasing the overall effort
required to deploy, maintain, and scale each cache.


> This paper presents a different approach to cache development, successfully employed at Facebook, which extracts
a core set of common requirements and functionality from
otherwise disjoint caching systems. CacheLib is a generalpurpose caching engine, designed based on experiences with
a range of caching use cases at Facebook, that facilitates the
easy development and maintenance of caches. CacheLib was
first deployed at Facebook in 2017 and today powers over 70
services including CDN, storage, and application-data caches.
This paper describes our experiences during the transition
from independent, specialized caches to the widespread adoption of CacheLib. We explain how the characteristics of production workloads and use cases at Facebook drove important
design decisions. We describe how caches at Facebook have
evolved over time, including the significant benefits seen from
deploying CacheLib. We also discuss the implications our experiences have for future caching design and research.

# 4. Design and Implementation
> CacheLib enables the construction of fast, stable caches for a
broad set of use cases. To address common challenges across
these use cases as described in Sections 2 and 3, we identify
`the following features` as necessary requirements for a `generalpurpose`
caching engine.

> Thread-safe cache primitives: To simplify programming
for applications that handle highly bursty traffic, CacheLib
provides a thread-safe API for reads, writes, and deletes. In
addition, thread-safety simplifies the implementation of consistency
and invalidation protocols. Concurrent calls to the
CacheLib API leave the cache in a valid state, respect linearizablility
[47] if referencing a common key, and incur minimal
resource contention.

> Transparent hybrid caching: To achieve high hit ratios
while caching large working sets, CacheLib supports caches
composed of both DRAM and flash, known as hybrid caches. Hybrid caches enable large-scale deployment of caches with
terabytes of cache capacity per node. CacheLib hides the
intricacies of the flash caching system from application programmers
by providing the same byte-addressable interface
(Section 4.1) regardless of the underlying storage media. This
transparency allows application programmers to ignore when
and where objects get written across different storage media.
It also increases the portability of caching applications,
allowing applications to easily run on a new hardware configurations
as they become available.

> Low resource overhead: CacheLib achieves high throughput
and low memory and CPU usage for a broad range of
workloads (Section 2). This makes CacheLib suitable for inprocess
use cases where the cache must share resources with
an application. Low resource overheads allow CacheLib to
support use cases with many small objects.

> Structured items: CacheLib provides a native implementation
of arrays and hashmaps that can be cached and mutated
efficiently without incurring any serialization overhead.
Caching structured data makes it easy for programmers to
efficiently integrate a cache with application logic.
Dynamic resource monitoring, allocation, and OOM protection:
To prevent crashes from temporary spikes in system
memory usage, CacheLib monitors total system memory usage.
CacheLib dynamically allocates and frees memory used
by the cache to control the overall system memory usage.

> `Warm restarts`: To handle code updates seamlessly, CacheLib can perform warm restarts that retain the state of the cache. This overcomes the need to warm up caches every time they are restarted.
* 

## 4.1. CacheLib API
> The CacheLib API is designed to be simple enough to allow
application programmers to quickly build in-process caching
layers with little need for cache tuning and configuration.
At the same time, CacheLib must scale to support complex
application-level consistency protocols, as well as zero-copy
access to data for high performance. Choosing an API which
is both simple and powerful was an important concern in the
design of CacheLib.
* CacheLib 提供简单的API，方便上层应用快速构建`in-process caching layers`
* CacheLib 支持复杂的`应用级一致性协议`，以及`零拷贝的访问`



## 4.2 Architecture Overview
> CacheLib is designed to be scalable enough to accommodate massive working sets (Section 3.1) with highly variable sizes (Section 3.2). To achieve low per-object overhead, a single CacheLib cache is composed of several subsystems, each of which is `tailored to`(针对,为...量身定制) a particular storage medium and object size. Specifically, CacheLib consists of a DRAM cache and a flash cache. The flash cache is composed of two caches: the Large Object Cache (LOC) for Items>=2KB in size and Small Object Cache (SOC) for Items <2KB in size.

* CacheLib 由两个子系统组成，一个用于DRAM，一个用于Flash
  * Flash cache 由两个子系统组成，Large Object Cache(LOC) 和 Small Object Cache(SOC)
  * LOC 用于item >= 2KB
  * SOC 用于item < 2KB

> An allocate request is fulfilled by allocating space in DRAM, evicting Items from DRAM if necessary. Evicted Items are either admitted to a flash cache (potentially causing another eviction) or `discarded`(废弃掉). A find request successively checks for an object in DRAM, then LOC, then SOC. This lookup order minimizes the average memory access time [46] of the cache (see Appendix A). A find call returns an ItemHandle immediately. If the requested object is located on DRAM, this ItemHandle is ready to use. If the requested object is located on flash, it is fetched asynchronously and the ItemHandle becomes ready to use once the object is in DRAM. An empty ItemHandle is returned to signify a cache miss. These data paths are summarized in Figure 9.

* 存在两种请求类型：`allocate request` and `find request`
* `allocate request`
  * 由DRAM分配空间
  * 如果空间不足，则从DRAM中evict item，evicted item可能被admit到flash cache，或者被discard
* `find request`
  * 先检查DRAM，然后LOC，最后SOC. 
  * 查找的原则是 减少平均内存访问时间
  * 查找的返回的结果是一个ItemHandle
    * 如果请求的对象在DRAM中，则ItemHandle就ready to use
    * 如果请求的对象在flash中，则需要异步fetch
    * 如果请求的对象不存在，则返回一个empty ItemHandle

> We now describe CacheLib’s subsystems in more detail. DRAM cache. CacheLib’s DRAM cache uses a chained hash table to perform lookups. The DRAM cache can be partitioned into separate pools, each with its own eviction policy. Programmers select a particular PoolId when calling allocate (see Figure 7), allowing the isolation of different types of traffic within a single cache instance.

* DRAM Cache 使用一个链式哈希表(chained hash table)来执行查找
* DRAM Cache 可以被partition 成一个个独立的pool，每个pool都有自己的eviction policy
* 用户可以select一个特定的poolId来进行item allocate
* Pool的作用可以让不同的流量traffic在单个cache实例中隔离开

> For performance, cache memory is allocated using slab classes [6, 22, 37] which store objects of similar sizes. CacheLib uses 4MB slabs and implements a custom slab allocator. Each slab requires 7B of DRAM (3B of internal metadata + 4B to identify the size of objects in the slab). Because CacheLib workloads often include many objects of a specific size(e.g., 80B), the sizes corresponding to each slab class are configured on a per-workload basis to minimize fragmentation. Further optimizations for objects smaller than 64B or larger than 4MB are described in Section 4.3.

* CacheLib 使用slab class来存储相似大小的objects
* CacheLib 使用4MB的slab，并实现了一个自定义的slab allocator
* 每个slab需要7B的内存开销，3B是内部metadata，4B用于标识slab中objects的大小
* CacheLib 负载包括一些特定大小的objects，因此需要根据负载来配置slab class的大小，以减少fragmentation 内存碎片
* CacheLib 针对size(<64B)或者(>4MB)的对象会进一步优化 (在4.3节描述)

> Each slab class maintains its own eviction policy state. CacheLib is designed to support the continual development of new eviction policies, and currently supports LRU, LRU with multiple insertion points, 2Q [54, 93], and TinyLFU [31]. These eviction policies differ in their overheads and their biases towards either recency or frequency, and are thus configured on a per-workload basis as well. To approximate a global eviction policy, memory is rebalanced between slab classes using known rebalancing algorithms [72]. To support these policies, among other features, CacheLib dedicates 31B of DRAM overhead per item. Table 1 describes the metadata which comprises this DRAM overhead.

* 每个slab class维护自己的eviction policy state
* CacheLib 允许用户定义新的eviction policy，目前支持
  * LRU
  * LRU with multiple insertion points
  * 2Q
  * TinyLFU
* eviction policies 的不同特性使得用户可以配置不同的policy(recency or frequency)
* CacheLib 使用一个全局的rebalancing algorithm来approximate一个全局的eviction policy
![1](/img/cachelib/table1.png)
* CacheLib 为每个item分配31B的DRAM overhead, 用来支持eviction policies 和 其他特性


> To guarantee atomic metadata operations, CacheLib relies on a variety of known optimization techniques [35, 62, 64], including **fine-grained locking**, **user-space mutexes**, and **C++ atomics**. This is particularly important for eviction policies, where naive implementations lead to lock contention and limit throughput [6, 9, 10, 61]. For example, under LRU, popular Items frequently compete to be reset to the most-recentlyused (MRU) position. This is particularly common at Facebook due to our high request rates (see Figure 6). CacheLib adopts a simple solution to reduce contention: Items that were recently reset to the MRU position are not reset again for some time T [9,87]. As long as T is much shorter than the time it takes an object to percolate through the LRU list (i.e., eviction age), this simplification does not affect hit ratios in practice. CacheLib also uses advanced locking mechanisms such as flat combining [45] to reduce resource contention. Flash cache. When Items are evicted from the DRAM cache, they can optionally be written to a flash cache. Due to high popularity churn (Section 3), the content cached on flash changes continually. Hence, in addition to maintaining low per-object overhead, CacheLib must contend with the limited write endurance of flash cells.

* 为了保证原子的metadata操作，CacheLib依赖一些已知的优化技术，包括
  * 细粒度的锁
  * 用户空间mutex
  * C++的atomic
* 
> To reduce the rate of writes to flash, CacheLib selectively writes objects to the flash cache. If an object exists on flash and was not changed while in DRAM, it is not written back to flash. Otherwise, CacheLib admits objects to flash according to a configurable admission policy. CacheLib’s default admission policy is to admit objects to the flash cache with a fixed probability p [57]. Adjusting the probability p allows finegrained control over write rate to flash. Section 5 describes our experience with more complex admission policies.