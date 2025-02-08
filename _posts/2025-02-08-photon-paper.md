---
layout:     post
title:      "【论文笔记】Photon: A Fast Query Engine for Lakehouse Systems"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:
    - 论文笔记
---

论文笔记 《[Photon: A Fast Query Engine for Lakehouse Systems](https://people.eecs.berkeley.edu/~matei/papers/2022/sigmod_photon.pdf)》

# ABSTRACT
> Many organizations are shifting to a data management paradigm called the “Lakehouse,” which implements the functionality of structured data warehouses on top of unstructured data lakes. This presents new challenges for query execution engines. The execution engine needs to provide good performance on the <u>raw uncurated datasets</u>原始的，未经整理的数据集 that are <u>ubiquitous</u>普遍存在的 in data lakes, and excellent performance on structured data stored in popular columnar file formats like Apache Parquet. Toward these goals, `we present Photon, a vectorized query engine for Lakehouse environments that we developed at Databricks. Photon can outperform existing cloud data warehouses in SQL workloads, but implements a more general execution framework that enables efficient processing of raw data and also enables Photon to support the Apache Spark API`. We discuss the design choices we made in Photon (e.g., vectorization vs. code generation) and describe its integration with our existing SQL and Apache Spark runtimes, its task model, and its memory manager. Photon has accelerated some customer workloads by over 10× and has recently allowed Databricks to set a new audited performance record for the official 100TB TPC-DS benchmark.
- Photon 在 SQL 工作负载上可以超越现有的云数据仓库，但它实现了一个更通用的执行框架，能够高效处理原始数据，并且支持 Apache Spark API
- Photon 已将一些客户的工作负载加速超过 10 倍，并且最近使 Databricks 创下了官方 100TB TPC-DS 基准的最新审计性能记录。