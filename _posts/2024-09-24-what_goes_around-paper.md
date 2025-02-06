---
layout:     post
title:      "【论文笔记】What Goes Around Comes Around... And Around..."
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:
    - 论文笔记
---

论文笔记 《[What Goes Around Comes Around... And Around...](https://db.cs.cmu.edu/papers/2024/whatgoesaround-sigmodrec2024.pdf)》

# Abstract
> Two decades ago, one of us co-authored a paper commenting on the previous 40 years of data modelling research and development [188]. That paper demonstrated that the relational model (RM) and SQL are the prevailing choice for database management systems (DBMSs), despite efforts to replace either them. Instead, SQL absorbed the best ideas from these alternative approaches.
> We revisit this issue and argue that this same evolution has continued since 2005. Once again there have been repeated efforts to replace either SQL or the RM. But the RM continues to be the dominant data model and SQL has been extended to capture the good ideas from others. As such, we expect more of the same in the future, namely the continued evolution of SQL and relational DBMSs (RDBMSs). We also discuss DBMS implementations and argue that the major advancements have been in the RM systems, primarily driven by changing hardware characteristics.

# Introduction
> In this paper, we analyze the last 20 years of data model and query language activity in databases. We structure our commentary into the following areas: (1) MapReduce Systems, (2) Key-value Stores, (3) Document Databases, (4) Column Family / Wide-Column, (5) Text Search Engines, (6) Array Databases, (7) Vector Databases, and (8) Graph Databases.

按下列领域 structure our commentary
1. MapReduce Systems
2. Key-value Stores
3. Document Databases
4. Column Family / Wide-Column
5. Text Search Engines
6. Array Databases
7. Vector Databases
8. Graph Databases

> We <u>contend</u>认为 that most systems that deviated from SQL or the RM have not dominated the DBMS landscape and often only serve <u>niche markets</u>小众市场. Many systems that started out rejecting the RM with much fanfare (think NoSQL) now expose a SQL-like interface for RM databases. Such systems are now on a path to convergence with RDBMSs. Meanwhile, SQL incorporated the best query language ideas to expand its support for modern applications and remain relevant.

- 大多数偏离SQL或RM的系统没有主导DBMS发展
- 有些系统(NoSQL)一开始拒绝RM，但后来暴露了SQL-like接口，使其可以支持RM数据库。
- 与此同时，SQL吸收了最佳查询语言理念，以扩展对现代应用的支持，保持其相关性。

> Although there has not been much change in RM fundamentals, there were dramatic changes in RM system implementations. The second part of this paper discusses advancements in DBMS architectures that address modern applications and hardware: (1) Columnar Systems, (2) Cloud Databases, (3) Data Lakes / Lakehouses, (4) NewSQL Systems, (5) Hardware Accelerators, and (6) Blockchain Databases. Some of these are <u>profound</u>深远的 changes to DBMS implementations, while others are merely trends based on faulty <u>premises</u>前提.

- 虽然关系模型的基本原则未发生太大变化，但关系模型系统的实现发生了剧烈变化

> We finish with `a discussion of important considerations for the next generation of DBMSs` and provide parting comments on our hope for the future of databases in both research and commercial settings.


# Data Models & Query Languages
> For our discussion here, we group the research and <u>development thrusts</u>发展推力，发展重点 in data models and query languages for database into eight categories.

## MapReduce Systems
> Google constructed their MapReduce (MR) framework in 2003 as a “`point solution`” for processing its periodic crawl of the internet [122]. At the time, Google had <u>little expertis</u>经验有限 in DBMS technology, and they built MR to meet their crawl needs. In database terms, `Map is a user-defined function` (UDF) that performs computation and/or filtering while `Reduce is a GROUP BY operation`. <u>To a first approximation</u>从初步的角度来看, MR runs a single query:  
> `SELECT map() FROM crawl_table GROUP BY reduce()`  
> Google’s MR approach did not <u>prescribe</u>规定 a specific data model or query language. Rather, it was up to the Map and Reduce functions written in a procedural MR program to <u>parse and decipher</u>分析和解读 the contents of data files.

- 谷歌的MR方法并未规定特定的数据模型或查询语言。相反，解析和解读数据文件内容的任务落在了作为程序运行的Map和Reduce函数上。

> There was a lot of interest in MR-based systems at other companies in the late 2000s. Yahoo! developed an open-source version of MR in 2005, called Hadoop. It ran on top of a distributed file system HDFS that was a clone of the Google File System [134]. Several startups were formed to support Hadoop in the commercial marketplace. We will use MR to refer to the Google implementation and Hadoop to refer to the open-source version. They are functionally similar.  
> There was a <u>controversy</u>争论，争议 about the value of Hadoop compared to RDBMSs designed for OLAP workloads. This <u>culminated</u>达到顶点/巅峰 in a 2009 study that showed that data warehouse DBMSs outperformed Hadoop [172]. This generated <u>dueling</u>决斗，争论 articles from Google and the DBMS community [123, 190]. Google argued that with careful engineering, a MR system will beat DBMSs, and a user does not have to load data with a schema before running queries on it. Thus, MR is better for “one shot” tasks, such as text processing and ETL operations. The DBMS community argued that MR incurs performance problems due to its design that existing parallel DBMSs already solved. Furthermore, the use of higher-level languages (SQL) operating over partitioned tables has proven to be a good programming model [127].  

- google认为: 通过精心的工程设计，混合响应（MR）系统将优于数据库管理系统（DBMS），用户在运行查询之前不必按照模式加载数据。因此，MR更适合“单次”任务，比如文本处理和ETL操作。
- DBMS社区则认为: MR由于其设计而造成的性能问题是现有的并行DBMS已经解决的。此外，使用高阶语言（如SQL）对分区表进行操作已被证明是一种良好的编程模型。


> A lot of the discussion in the two papers was on implementation issues (e.g., indexing, parsing, push vs. pull query processing, failure recovery). `From reading both papers a reasonable conclusion would be that there is a place for both kinds of systems.` However, two changes in the technology world <u>rendered</u>使成为 the debate <u>moot</u>无意义.  

- 从阅读这两篇论文可以得出一个合理的结论，即这两种系统都有其存在的价值。然而，科技界的两项变化让这一争论变得无关紧要。

> `The first event was that the Hadoop technology and services market cratered in the 2010s.` Many enterprises spent a lot of money on Hadoop clusters, only to find there was little interest in this functionality. Developers found it difficult to <u>shoehorn</u>把...硬塞 their application into the restricted MR/Hadoop paradigm. There were considerable efforts to provide a SQL and RM interface on top of Hadoop, most notable was Meta’s Hive [30, 197].  

- 第一个事件是Hadoop技术和 服务市场在2010年代崩溃

> The next event occurred eight months after the CACM article when Google announced that they were moving their crawl processing from MR to BigTable [164]. The reason was that Google needed to interactively update its crawl database in real time but MR was a batch system. Google finally announced in 2014 that MR had no place in their technology stack and killed it off [194].  
> The first event left the three leading Hadoop vendors (Cloudera, Hortonworks, MapR) without a viable product to sell. Cloudera rebranded Hadoop to mean the whole stack (application, Hadoop, HDFS). In a further sleight-of-hand, Cloudera built a RDBMS, Impala [150], on top of HDFS but not using Hadoop. They realized that Hadoop had no place as an internal interface in a SQL DBMS, and they configured it out of their stack with software built directly on HDFS. In a similar vein, MapR built Drill [22] directly on HDFS, and Meta created Presto [185] to replace Hive.  

- 通过进一步的手法，Cloudera在HDFS之上构建了一个关系数据库管理系统（RDBMS）Impala [150]，但并没有使用Hadoop。他们意识到Hadoop在SQL DBMS的内部接口中没有存在的必要，因此将其从他们的技术栈中配置了出去，转而使用直接构建在HDFS上的软件。
- 类似地，MapR在HDFS之上直接构建了Drill [22]，
- Meta则创建了Presto [185]来替代Hive。
  
> **Discussion**: MR’s deficiencies were so significant that it could not be saved despite the adoption and enthusiasm from the developer community. Hadoop died about a decade ago, leaving a legacy of HDFS clusters in enterprises and a collection of companies dedicated to making money from them. At present, HDFS has lost its <u>luster</u>光彩, as enterprises realize that there are **better distributed storage alternatives** [124]. Meanwhile, distributed RDBMSs are <u>thriving</u>欣欣向荣, especially in the cloud.  
> Some aspects of MR system implementations related to scalability, elasticity, and fault tolerance are <u>carried over</u>延续，保留 into distributed RDBMSs. MR also brought about the revival of shared-disk architectures with disaggregated storage, subsequently giving rise to open-source file formats and data lakes (see Sec. 3.3). Hadoop’s limitations opened the door for other data processing platforms, namely Spark [201] and Flink [109]. Both systems started as better implementations of MR with procedural APIs but have since added support for SQL [105].

- 与可扩展性、弹性和容错性相关的某些MR系统实施方面被引入到分布式关系数据库管理系统（RDBMS）中。MR还带来了共享磁盘架构的复兴，伴随着去耦合存储的发展，这随后催生了开源文件格式和数据湖（见第3.3节）。Hadoop的局限性为其他数据处理平台打开了大门，即Spark [201]和Flink [109]。这两个系统最初是作为MR的更好实现，拥有过程API，但后来增加了对SQL的支持。

## 2.2 Key/Value Stores
> The key/value (KV) data model is the simplest model possible. It represents the following binary relation: (key,value) A KV DBMS represents a collection of data as an associative array that maps a key to a value. The value is typically an untyped array of bytes (i.e., a blob), and the DBMS is unaware of its contents. It is up to the application to maintain the schema and parse the value into its corresponding parts. Most KV DBMSs only provide **get/set/delete operations on a single value**.  
> In the 2000s, several new Internet companies built their own shared-nothing, distributed KV stores for <u>narrowly focused applications</u>特定的应用程序, like caching and storing session data. For caching, Memcached [131] is the most well-known example of this approach. Redis [67] markets itself as a Memcached replacement, offering a more robust query API with checkpointing support. For more persistent application data, Amazon created the Dynamo KV store in 2007 [125]. Such systems offer higher and more predictable performance, compared to a RDBMS, in exchange for **more limited functionality**.  

- Memcached是这种方法中最著名的例子
- Redis则自称是Memcached的替代品，提供更强大的查询API以及检查点支持。
- 对于更持久的应用数据，亚马逊在2007年创建了Dynamo KV存储

> The second KV DBMS category are embedded storage managers designed to run in the same address space as a higher-level application. One of the first standalone embedded KV DBMSs was BerkeleyDB from the early 1990s [170]. Recent notable entries include Google’s LevelDB [37], which Meta later forked as RocksDB [68].  

- 第二类KV DBMS是嵌入式存储管理器，旨在与更高层次的应用程序在同一地址空间中运行。第一个独立的嵌入式KV DBMS是早期的BerkeleyDB。最近的显著条目包括谷歌的LevelDB，后来Meta将其分叉为RocksDB。

> Discussion: Key/value stores provide a quick “`out-of-the-box`开箱即用” way for developers to store data, compared to the more <u>laborious</u>耗时费力的 effort required to set up a table in a RDBMS. Of course, it is dangerous to use a KV store in a complex application that requires <u>more than just a binary relation</u>不仅仅是二元关系. `If an application requires multiple fields in a record, then KV stores are probably a bad idea.` Not only must the application parse record fields, but also there are no secondary indexes to retrieve other fields by value. Likewise, developers must implement joins or multi-get operations in their application.  

- 如果一个应用程序需要记录中的多个字段，那么使用KV存储可能并不是一个好主意
- 原因： 应用程序不仅必须解析记录字段，而且没有二级索引来通过值检索其他字段。同样，开发人员必须在其应用程序中实现联接或多次获取操作。
  
> To deal with these issues, several systems began as a KV store and then <u>morphed</u>变换 into a more feature-rich `record store`. Such systems replace the opaque value with a semi-structured value, such as a JSON document. Examples of this transition are Amazon’s DynamoDB [129] and Aerospike [9]. It is <u>not trivial</u>不平凡，不轻松的 to reengineer a KV store to make it support a complex data model, whereas RDBMSs easily emulates KV stores without any changes. If an application needs an embedded DBMS, there are full-featured choices available today, including SQLite [71] and DuckDB [180]. Hence, a RDBMS may be a better choice, even for simple applications, <u>because they offer a path forward if the application’s complexity increases</u>因为它们在应用程序复杂性增加时提供了向前发展的路径.  

- 几个系统开始作为KV存储，然后逐渐转变为更具功能丰富的记录存储。这些系统用半结构化值（如JSON文档）替换不透明值。这种转变的例子包括亚马逊的DynamoDB和Aerospike。

> One new architecture trend from the last 20 years is using embedded KV stores as the underlying storage manager for full-featured DBMSs. Prior to this, building a new DBMS requires engineers to build a custom storage manager that is natively integrated in the DBMS. `MySQL was the first DBMS to expose an API that allowed developers to replace its default KV storage manager`. This API enabled Meta to build RocksDB to replace **InnoDB** for its massive fleet of MySQL databases. Similarly, MongoDB discarded their <u>ill-fated</u>不幸的，厄运的 MMAP-based storage manager in favor of **WiredTiger’s KV store** in 2014 [120, 138]. Using an existing KV store allows developers to write a new DBMS in less time.

- MySQL是第一个公开API，允许开发人员替换其默认KV存储管理器的DBMS.
- WiredTiger作为mongodb的默认存储引擎
- 使用现有的KV存储使得开发人员可以在更短的时间内编写新的DBMS

