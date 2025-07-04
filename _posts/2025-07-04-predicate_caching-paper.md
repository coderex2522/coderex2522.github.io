---
layout:     post
title:      "【论文笔记】Predicate Caching: Query-Driven Secondary Indexing for Cloud DataWarehouses"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:

    - Predicate Caching
    - 论文笔记
---

论文笔记 《Predicate Caching: Query-Driven Secondary Indexing for Cloud DataWarehouses》

# Abstract
> Cloud data warehouses are today’s standard for analytical query
processing. Multiple cloud vendors offer state-of-the-art systems,
such as Amazon Redshift. We have observed that customer workloads
experience highly repetitive query patterns, i.e., users and
systems frequently send the same queries. In order to improve query
performance on these queries, most systems rely on techniques like
`result caches` or `materialized views`.

- 为了改善重复query的性能，系统一般采用result cache 和物化视图来进行优化


> However, these caches are often stale due to inserts, deletes, or updates that occur between query repetitions. We propose a novel secondary index, `predicate caching`, to improve query latency for repeating scans and joins. Predicate caching stores ranges of qualifying tuples of base table scans. Such an index can be built on the fly, is lightweight, and can be kept online without recomputation.

- result cache和物化视图等，经常会因为插入，删除，更新等操作失效。

> We implemented a prototype of this idea in the cloud data
warehouse Amazon Redshift. Our evaluation shows that predicate
caching improves query runtimes by up to 10x on selected queries
with negligible build overhead.

- 性能：提升10倍，并且构建成本可忽略


