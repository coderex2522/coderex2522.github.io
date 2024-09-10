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
> Web services rely on caching at nearly every layer of the system architecture. Commonly, each cache is implemented and maintained independently by a distinct team and is highly specialized to its function. For example, an application-data cache would be independent from a CDN cache. However, this approach ignores the difficult challenges that different caching systems have in common, greatly increasing the overall effort required to deploy, maintain, and scale each cache.

* Web services 依赖系统结构中每层的cache进行加速
* 每层cache 是由不同的团队独立开发，并且是高度定制的
* 不同缓存系统的挑战存在共性
* 不同缓存系统导致开发、维护、扩展成本高

> This paper presents a different approach to cache development, successfully employed at Facebook, which extracts a core set of common requirements and functionality from otherwise disjoint caching systems. CacheLib is a generalpurpose caching engine, designed based on experiences with a range of caching use cases at Facebook, that facilitates the easy development and maintenance of caches. CacheLib was first deployed at Facebook in 2017 and today powers over 70 services including CDN, storage, and application-data caches. This paper describes our experiences during the transition from independent, specialized caches to the widespread adoption of CacheLib. We explain how the characteristics of production workloads and use cases at Facebook drove important design decisions. We describe how caches at Facebook have evolved over time, including the significant benefits seen from deploying CacheLib. We also discuss the implications our experiences have for future caching design and research.

* 论文呈现了一个新的缓存系统设计，成功应用在Facebook上，该系统将`提取`出`通用`的`需求和功能`，并将其应用到`不同`的`缓存系统`中。
* CacheLib 是一个通用的缓存引擎，基于Facebook的`大量的缓存使用case`

# 4. Design and Implementation
> CacheLib enables the construction of fast, stable caches for a broad set of use cases. To address common challenges across these use cases as described in Sections 2 and 3, we identify `the following features` as necessary requirements for a `generalpurpose` caching engine.   

 > `Thread-safe cache primitives`: To simplify programming for applications that handle highly bursty traffic, CacheLib provides a thread-safe API for reads, writes, and deletes. In addition, thread-safety simplifies the implementation of consistency and invalidation protocols. Concurrent calls to the CacheLib API leave the cache in a valid state, respect linearizablility [47] if referencing a common key, and incur minimal resource contention.

 * CacheLib 提供了线程安全的缓存接口(read/write/delete)，简化了应用程序应对突发峰值流量的编程
 * 线程安全接口简化了一致性和失效协议
 * 对 CacheLib API 的并发调用不会导致cache状态不一致，针对同一键的操作遵循线性化性，并且造成的资源竞争最小

 > `Transparent hybrid caching`: To achieve high hit ratios while caching large working sets, CacheLib supports caches composed of both DRAM and flash, known as hybrid caches. Hybrid caches enable large-scale deployment of caches with terabytes of cache capacity per node. CacheLib hides the `intricacies`(复杂性) of the flash caching system from application programmers by providing the same `byte-addressable interface` (Section 4.1) regardless of the underlying storage media. This transparency allows application programmers to ignore when and where objects get written across different storage media. It also increases the `portability`(可移植性) of caching applications, allowing applications to easily run on a new hardware configurations as they become available.

 * CacheLib 支持混合缓存，能够实现大容量的部署，拥有TB级缓存容量
 * CacheLib 隐藏了flash缓存系统的复杂性，通过提供`相同的字节地址接口`，无论底层存储介质是什么，都为应用程序提供一致的接口
 * 增加应用程序可移植性，使应用程序可以在新硬件配置上运行，而无需修改代码

 > `Low resource overhead`: CacheLib achieves `high throughput and low memory and CPU usage` for a broad range of workloads (Section 2). This makes CacheLib suitable for in-process use cases where the cache must share resources with an application. Low resource overheads allow CacheLib to support use cases with many small objects.   

* CacheLib 具备`高吞吐量和低内存和CPU使用量`，使其适用于进程内使用场景，缓存必须与应用程序共享资源
 
 > `Structured items`: CacheLib provides a native implementation of arrays and hashmaps that can be cached and mutated efficiently without incurring any serialization overhead. Caching structured data makes it easy for programmers to efficiently integrate a cache with application logic. Dynamic resource monitoring, allocation, and OOM protection: To prevent crashes from temporary spikes in system memory usage, CacheLib monitors total system memory usage. CacheLib dynamically allocates and frees memory used by the cache to control the overall system memory usage. 

* CacheLib 提供了数组和哈希表的原生实现，能够在不产生任何序列化开销的情况下高效缓存和修改
* CacheLib 提供了结构化数据的原生实现，可以轻松的将缓存集成到应用程序逻辑中
* 监控系统内存使用情况，动态分配和释放内存，防止因临时内存使用峰值导致系统崩溃

> `Warm restarts`: To handle code updates seamlessly, CacheLib can perform warm restarts that retain the state of the cache. This overcomes the need to warm up caches every time they are restarted.

* 为了无缝处理代码更新，CacheLib 可以执行`热重启`，保持缓存的状态

## 4.1. CacheLib API
> The CacheLib API is designed to be simple enough to allow application programmers to quickly build in-process caching layers with little need for cache tuning and configuration. At the same time, CacheLib must scale to support complex application-level consistency protocols, as well as zero-copy access to data for high performance. Choosing an API which is both simple and powerful was an important concern in the design of CacheLib. 

* CacheLib 提供简单的API，方便上层应用快速构建`in-process caching layers`, 并且不需要缓存调优和配置
* CacheLib 具备可扩展性，支持复杂的`应用级一致性协议`，以及`零拷贝的访问`
* CacheLib设计过程中一个重要的考虑因素是`简单和强大`的API

> The API centers around the concept of an Item, an abstract representation of a cached object. The Item enables byte-addressable access to an underlying object, independent of whether the object is stored in DRAM or flash. Access to cached Items is controlled via an ItemHandle which enables reference counting for cached Items. When an ItemHandle object is constructed or destroyed, a reference counter for the corresponding Item is incremented or decremented, respectively. An Item cannot be evicted from the cache unless its reference count is 0. If an Item with a non-zero reference count expires or is deleted, existing ItemHandles will remain valid, but no new ItemHandles will be issued for the Item.

* Item 是Cache Object抽象表示，Item 封装了底层对象，支持 `Byte-Addressable`, 与底层存储介质无关
* Item 通过ItemHandle 控制访问，ItemHandle 支持引用计数
* 只有 Item 的引用计数为0时，Item 才可以被驱逐
* 如果Item引用计数不为0但是已经expired或者deleted，Item将不会被驱逐，ItemHandle仍然有效，但是`不能为Item创建新的ItemHandle`

> Figure 7 shows the basic CacheLib API. To insert a new object into the cache, allocate may first evict another Item (according to an eviction policy) as long as there are no outstanding ItemHandles that reference it. The new Item can be configured with an expiration time (TTL). It is created within the given memory “pool” (see below), which can be individually configured to provide strong isolation guarantees. Any new Items only become visible after an insertOrReplace operation completes on a corresponding ItemHandle. To access cached Items, find creates an ItemHandle from a key, after which getMemory allows unsynchronized, zero-copy access to the memory associated with an Item. To atomically update an Item, one would allocate a new ItemHandle for the key they wish to update, perform the update using getMemory, and then make the update visible calling insertOrReplace with the new ItemHandle. Because CacheLib clients access raw memory for performance, CacheLib trusts users to faithfully indicate any mutations using the method markNvmUnclean. Finally, remove deletes the Item identified by a key, indicating invalidation or deletion of the underlying object.

* `insertOrReplace`操作会先驱逐另一个Item，如果Item的引用计数不为0，那么ItemHandle将不会被驱逐，ItemHandle仍然有效，但是不能为Item创建新的ItemHandle
* `find`操作会创建一个ItemHandle，通过`getMemory`方法可以获取Item的底层内存，通过`markNvmUnclean`方法可以标记Item的底层内存是否被修改
* `remove`操作会删除Item，通过`markNvmUnclean`方法可以标记Item的底层内存是否被修改
* `markNvmUnclean`方法可以标记Item的底层内存是否被修改
* `remove`操作会删除Item，通过`markNvmUnclean`方法可以标记Item的底层内存是否被修改

> Figure 8 shows a simple example of CacheLib’s native support for structured data. Structured Items are accessed through a TypedHandle, which offers the same methods as an ItemHandle. TypedHandles enable low-overhead access to user-defined data structures which can be cached and evicted just like normal Items. In addition to statically sized data structures, CacheLib also supports variably-sized data structures; for example, CacheLib implements a simple hashmap that supports range queries, arrays, and iterable buffers.

* CacheLib 支持结构化数据的原生实现，通过`TypedHandle`可以访问结构化数据
* TypedHandle 提供了与ItemHandle相同的方法
* CacheLib 支持可变大小的结构化数据，例如，CacheLib实现了一个简单的hashmap，支持范围查询，数组，可迭代缓冲区

> CacheLib implements these APIs in C++, with binding to other languages such as Rust.

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


> To guarantee atomic metadata operations, CacheLib relies on a variety of known optimization techniques [35, 62, 64], including `fine-grained locking`, `user-space mutexes`, and `C++ atomics`. This is particularly important for eviction policies, where naive implementations lead to lock contention and limit throughput [6, 9, 10, 61]. For example, under LRU, popular Items frequently compete to be reset to the most-recently used (MRU) position. This is particularly common at Facebook due to our high request rates (see Figure 6). CacheLib adopts a simple solution to reduce contention: Items that were recently reset to the MRU position are not reset again for some time T [9,87]. As long as T is much shorter than the time it takes an object to percolate through the LRU list (i.e., eviction age), this simplification does not affect hit ratios in practice. CacheLib also uses advanced locking mechanisms such as flat combining [45] to reduce resource contention.

* 为了保证原子的metadata操作，CacheLib依赖一些已知的优化技术，包括
  * 细粒度的锁
  * 用户空间mutex
  * C++的atomic
* 这些优化技术对eviction policies 非常重要，因为 naive 实现会导致`锁冲突`，从而限制`throughput`
* 例如
  * 问题：在LRU中，popular Items会频繁地被reset到MRU的位置， 解决办法是
    * 在reset到MRU位置后，对象在T时间内不会被reset
    * 只要T比对象在LRU列表中维持的时间要短，那么这个简化方案不会影响hit ratio
* CacheLib 使用 advanced locking mechanisms(例如: flat combining) 来减少资源竞争

>  `Flash cache`. When Items are evicted from the DRAM cache, they can optionally be written to a flash cache. Due to high popularity churn (Section 3), the content cached on flash changes continually. Hence, in addition to maintaining low per-object overhead, CacheLib must contend with the limited write endurance of flash cells.

* Item 从 DRAM cache 被evicted，可以选择写入flash cache
* 由于popularity churn(Section 3)，flash cache的内容会持续改变
* 因此，除了保持每个object的overhead，CacheLib还必须考虑闪存单元有限的写入耐久性问题

> To reduce the rate of writes to flash, CacheLib `selectively` writes objects to the flash cache. If an object exists on flash and was not changed while in DRAM, it is not written back to flash. Otherwise, CacheLib admits objects to flash according to a configurable admission policy. CacheLib’s default admission policy is to admit objects to the flash cache with a fixed probability p [57]. Adjusting the probability p allows finegrained control over write rate to flash. Section 5 describes our experience with more complex admission policies.

* 为了减少flash的写入频率，CacheLib会`选择性的`将object写入Flash cache.
* 如果object在Flash中存在，并且在DRAM中没有被修改，那么不会写入Flash.
* 否则，CacheLib会根据一个可配置的admission policy 允许objects 写入 flash
* CacheLib的默认admission policy 是根据`固定的概率 p`
* 调整概率 p的值 可以提供更细粒度的控制写入flash的频率 (section 5会描述更复杂的admission policies)


> Another consideration for flash endurance is write amplification which happens when the number of bytes written to the device is larger than the number of bytes inserted into the cache. For instance, CacheLib performs extra writes to store metadata and is forced to write at block granularities. We distinguish between application-level write amplification, which occurs when CacheLib itself writes more bytes to flash than the size of the inserted object, and device-level write amplification, which is caused by the flash device firmware. CacheLib’s flash caches are carefully designed to balance both sources of write amplification and DRAM overhead. 

* 另一个与flash耐用性相关的考虑因素是`写放大` (当写入flash的bytes数量比插入到cache的bytes数量多时，就会发生写放大. 例如，CacheLib会多写入一些bytes来存储metadata，并且被强制写入block granularities)
* 需要区分以下两种写放大
  * `application-level write amplification`: 发生于CacheLib自身写入flash的bytes数量比插入到cache的object的bytes数量多
  * `device-level write amplification`: 由flash设备固件引起的
* CacheLib的flash cache通过精心设计，以平衡两种`写放大`和`DRAM overhead`

> The Large Object Cache (LOC) is used to store objects larger than 2KB on flash. Because LOC objects are larger than 2KB, the number of unique objects in a LOC will only number in the millions. It is therefore feasible to keep an in-memory index of the LOC. The LOC uses segmented B+ trees [23, 34, 63] in DRAM to store the flash locations of Items. Items are aligned to 4KB flash pages, so the flash location is a 4B, 4KB-aligned address. This allows the LOC to index up to 16TB of flash storage space.

* Large Object Cache (LOC) 用于在flash中存储大于2KB的object. 因为LOC object 大于2KB，所以LOC中object的数量在百万以内，因此可以构建一个DRAM内存中的索引, 用来映射item在Flash中的地址
* Item 在flash中以4KB对齐的方式存储，所以flash地址是一个4B的4KB对齐地址
* 允许LOC索引`16TB`的flash存储空间

> The LOC uses flash to further limit the size of the DRAM index. Keys are hashed to 8B. The first 4B identify the B+- tree segment, and the second 4B are used as a key within in a tree segment to lookup a flash location. A hash collision in the DRAM index will cause CacheLib to believe it has found an object’s flash location. Hence, LOC stores a copy of each object’s full key on flash as part of the object metadata and validates the key after the object is fetched off flash. Each flash device is partitioned into regions which each store objects of a different size range. Hence, the object size can be inferred from where it is located on flash, without explicitly storing object sizes in DRAM. CacheLib can then retrieve the object via a single flash read for the correct number of bytes. To reduce the size of addresses stored in DRAM, every 4KB flash page stores at most a single object and its metadata. This is spaceefficient because LOC only stores objects larger than 2KB. Objects larger than 4KB can span multiple pages. Because the LOC reads and writes at the page level, any fragmentation also causes application-level write amplification.


> To amortize the computational cost of flash erasures, the LOC’s caching policy evicts entire regions rather than individual objects. (Region size is configurable, e.g., 16MB.) By default, FIFO is used so that regions are erased in strictly sequential order [60]. Writing sequentially improves the performance of the flash translation layer and greatly reduces device-level write amplification (see Section 5.2). If FIFO eviction evicts a popular object, it may be readmitted to the cache [86]. Alternatively, LOC supports a pseudo-LRU policy which tracks recency at region granularity. A request for any object in a region logically resets the region to the MRU position. Evictions erase the entire region at the LRU position. The Small Object Cache (SOC) is used to store objects smaller than 2KB on flash. Because billions of small objects can fit on a single 1TB flash device, an exact lookup index (with associated per-object metadata) would use an unreasonably large amount of DRAM [32]. Hence, SOC uses an approximate index that scales with the number of flash pages. SOC hashes keys into sets.Each set identifies a 4KB flash page which is used to store multiple objects. Objects are evicted from sets in FIFO order. A naive implementation of this design would always read a set’s page off flash to determine whether it contains a particular object. As an optimization, CacheLib maintains an 8B Bloom filter in DRAM for each set, each with 4 probes. This filter contains the keys stored on the set’s flash page and prevents unnecessary reads more than 90% of the time [11, 12]. Figure 10 shows the alloc path, and Figure 11 shows the find path. Controlling write amplification in the SOC is particularly challenging. Admitting an object to the SOC requires writing an entire 4KB flash page, and is thus a significant source of application-level write amplification. This also applies to the remove operation, which removes an object from flash. Similarly, because keys are hashed to sets, admitting a stream of objects to the SOC causes random writes that result in higher device-level write amplification. Furthermore, the SOC only supports eviction policies that do not require state updates on hits, such as FIFO, since updating a set on a hit would require a 4KB page write. These challenges highlight the importance of advanced admission policies (see Section 5.2).


## 4.3. Implementation of Advanced Features
> CacheLib supports many applications with demanding requirements. To support these applications efficiently, Cache- Lib implements several advanced features, making them available to all CacheLib-based services under the same, generalpurpose CacheLib API. We describe the implementation of four important features: structured items, caching large and small objects, warm restarts, and resource monitoring, corresponding to challenges already discussed in Section 3.

* 为了高效支持多种应用，CacheLib实现了许多高级功能，使其可以在所有基于CacheLib的服务中通用。我们描述了CacheLib实现的四个重要的功能
  * 结构化对象
  * 缓存大和小对象
  * 热重启
  * 资源监控

> `Structured items`. Because CacheLib provides raw access to cached memory, flat data structures can be easily cached using the CacheLib API. In addition, CacheLib natively supports arrays and maps. CacheLib supports an Array type for fixedsize objects at no additional overhead for each entry in the array. The Map type supports variable object sizes, and comes in ordered and unordered variants. The overhead for each Map entry is 4B to store its size.

* CacheLib提供了对缓存内存的直接访问，所以可以使用CacheLib API来缓存扁平数据结构。
* CacheLib支持`数组`和`MAP`
  * CacheLib支持固定大小的对象数组，`无需为每个entry添加额外的开销`
  * Map类型支持可变对象大小，并支持有序和无序变体。Map的每个entry的开销为4B，用于存储其大小。

> `Caching large and small objects in DRAM`. To store objects larger than 4MB in size, CacheLib chains multiple DRAM Items together into one logical large item. This chaining requires an additional 4B next pointer per object in the chain. The most common use case for large objects is the storage of structured items. While it is uncommon for a single, logical object to be larger than 4MB, we frequently see Arrays or Maps that comprise more than 4MB in aggregate. CacheLib also features compact caches, DRAM caches designed to cache objects smaller than a cache line (typically 64B or 128B). Compact caches store objects with the same key size and object size in a single cache line [18, 29, 46, 80]. Compact caches are `set-associative`(集合关联) caches, where each cache line is a set which is indexed by a key’s hash. LRU eviction is done within each set by repositioning objects within a cache line. Compact caches have no per-object overhead. One prominent example of using compact caches is CacheLib’s support for negative caching. Negative cache objects indicate that a backend query has previously returned an empty result. Negative cache objects are small, fixed-size objects which only require storing a key to identify the empty query. As discussed in Section 3.5, negative caching improves hit ratios drastically in SocialGraph. Negative caching is not used by Lookaside, Storage, or CDN, but it is employed by 4 of the 10 largest CacheLib-based systems. Both of these features reinforce CacheLib’s `overarching`(首要的) design, which is to provide specialized solutions for objects of different sizes in order to keep per-object overheads low.

* CacheLib 允许缓存大于4MB的对象，将多个DRAM Item链接在一起，形成一个逻辑大对象。这需要每个对象链中的附加4B的next指针
* 对于large objects而言，常用的应用场景是存储结构化对象。虽然单个逻辑对象通常不会超过4MB，但通常会看到包含超过4MB的对象的`数组或MAP`
* CacheLib 具备`compact caches`，它是设计用于缓存小于缓存行（通常为 64B 或 128B）对象的 DRAM 缓存.
  * compact caches 存储将具有相同键大小和对象大小的对象在一个缓存行中，通常为 64B 或 128B
  * compact caches 是`集合关联`的缓存，其中每个缓存行都是一个索引的集合，由对象的哈希值索引
  * 每个缓存行是一个被键的哈希索引的集合。LRU eviction在每个集合内通过重新定位缓存行内的对象来实现。compact caches 对于item没有额外开销
* 10个基于CacheLib的系统中有4个应用都使用了 Negative cache


> `Dynamic resource usage and monitoring`. CacheLib monitors the total system memory usage and continuously adapts the DRAM cache size to stay below a specified bound. CacheLib exposes several parameters to control the memory usage mechanism. If the system free memory drops below lowerLimitGB bytes, CacheLib will iteratively free percentPerIteration percent of the difference between upperLimitGB and lowerLimitGB until system free memory rises above upperLimitGB. A maximum of maxLimitPercent of total cache size can be freed by this process, preventing the cache from becoming too small. Although freeing memory may cause evictions, this feature is designed to prevent outright crashes which are far worse for cache hit ratios (see Figure 15). As system free memory increases, CacheLib reclaims memory by an analogous process.

* CacheLib 监控了系统内存使用情况，并持续调整 DRAM 缓存的大小，以保持在指定的边界以下
* CacheLib 提供了 几种参数 来控制`内存使用机制`
  * 

> `Warm restarts`. CacheLib implements warm restarts by allocating DRAM cache space using POSIX shared memory [76]. This allows a cache to shut down while leaving its cache state in shared memory. A new cache can then take ownership of the cache state on start up. The DRAM cache keeps its index permanently in shared memory by default. All other DRAM cache state is serialized into shared memory during shutdown. The LOC B-tree index and SOC Bloom filters are serialized and written in a dedicated section on flash during shutdown.

* CacheLib 通过使用POSIX共享内存来实现`Warm restarts`
* 

