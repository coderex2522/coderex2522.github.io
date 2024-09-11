---
layout:     post
title:      "【论文笔记】Flashield: a Hybrid Key-value Cache that Controls Flash Write Amplification"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:

    - CacheLib
    - 论文笔记
---

论文笔记 《[Flashield: a Hybrid Key-value Cache that Controls Flash Write Amplification](https://www.usenix.org/system/files/nsdi19spring_eisenman_prepub.pdf)》

# Abstract
> As its price per bit drops, SSD is increasingly becoming the default storage medium for hot data in cloud application databases. Even though SSD’s price per bit is more than 10× lower, and it provides sufficient performance (when accessed over a network) compared to DRAM, the durability of flash has limited its adoption in write-heavy use cases, such as key-value caching. This is because key-value caches need to frequently insert, update and evict small objects. This causes excessive writes and erasures on flash storage, which significantly shortens the lifetime of flash. We present Flashield, a hybrid key-value cache that uses DRAM as a “filter” to control and limit writes to SSD. Flashield performs lightweight machine learning admission control to predict which objects are likely to be read frequently without getting updated; these objects, which are prime candidates to be stored on SSD, are written to SSD in large chunks sequentially. In order to efficiently utilize the cache’s available memory, we design a novel in-memory index for the variable-sized objects stored on flash that requires only 4 bytes per object in DRAM. We describe Flashield’s design and implementation, and evaluate it on real-world traces from a widely used caching service, Memcachier. Compared to state-ofthe-art systems that suffer a write amplification of 2.5× or more, Flashield maintains a median write amplification of 0.5× (since many filtered objects are never written to flash at all), without any loss of hit rate or throughput.

* flash/ssd price per bit 比 DRAM价格低10倍+
* flash/ssd  durability(耐用性) 限制了它在write-heavy use cases(比如kv cache)中的采用
* kv cache 需要频繁的插入，更新，eviction, 这会导致flash 过量的 `write and erasures`
* Flashield，一个混合键值缓存
  * 使用DRAM作为“过滤器”来控制和限制对SSD的写入
  * 利用轻量级的机器学习进行`admission control`，预测哪些对象可能会频繁读取而不被更新
  * 这些对象是存储在SSD上的理想候选者，以大块数据的形式顺序写入SSD
  * 为了有效利用缓存的可用DRAM内存，我们设计了一种`新颖的内存索引`，针对存储在闪存上的可变大小对象，在DRAM中仅需`4字节`来存储每个对象
* 与那些遭受2.5倍或更多写入放大效应的现有系统相比，Flashield保持了中位写入放大率为0.5倍（因为许多过滤的对象根本不会写入闪存），并且在命中率和吞吐量上没有任何损失。