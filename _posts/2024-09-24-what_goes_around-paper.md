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

## 2.3 Document Databases
> The document data model represents a database as a collection of record objects. Each document contains a hierarchy of field/value pairs, where each field is identified by a name and a field’s value can be either a scalar type, an array of values, or another document. The following example in JSON is a customer document that contain a nested list of purchase order records with their corresponding order items.  
> `{ “name”: “First Last”, “orders”: [ { “id”: 123, “items”: [...] }, { “id”: 456, “items”: [...] }, ] } `  
> Document data models have been an active field of effort for several decades. This has <u>given rise to</u>导致，引起 data formats like SGML [117] and XML [118]. Despite the buzz with XML databases in the late 1990s, we correctly predicted in 2005 they would not <u>supplant</u>取代，代替 RDBMSs [188]. `JSON has since overtaken XML to become the standard for data exchange for web-based applications`. JavaScript’s popularity with developers and the accompanying ubiquity of JSON led several companies to create document-oriented systems that natively stored JSON in the 2000s.  
> The <u>inability</u>无力，无能 of OLTP RDBMSs to scale in the 2000s <u>ushered</u>引导，引领，开创 in dozens of document DBMSs that marketed themselves using the <u>catchphrase</u>标语 NoSQL [110]. There were two marketing messages for such systems that <u>resonated</u>充满 with developers. First, SQL and joins are slow, and one should `use a “faster” lower-level, record-at-atime interface`低级逐条记录接口. Second, ACID transactions are unnecessary for modern applications, so the DBMS should only provide weaker notion of it (i.e., BASE [179]).  
> Because of these two <u>thrusts</u>原因, NoSQL came to stand for a DBMS that stored records or documents as JSON, supported a lower-level API, and weak or non-existent transactions. There are dozens of such systems, of which MongoDB [41] is the most popular.

- NoSQL成为了存储记录或文档以JSON格式，支持低级API和弱或不存在事务的DBMS的代名词。其中，MongoDB是最受欢迎的一个例子

> **Discussion**: Document DBMSs are essentially the same as object-oriented DBMSs from the 1980s and XML DBMSs from the late 1990s. Proponents支持者 of document DBMSs <u>make the same argument as</u>做同样的论证 their OO/XML predecessors前者/前身: storing data as documents removes the impedance阻抗 mismatch between how application OO code interacts with data and how relational databases store them. They also claim that denormalizing entries into nested structures is better for performance because it removes the need to dispatch multiple queries to retrieve data related to a given object (i.e., “N+1 problem” in ORMs). The problems with `denormalization/prejoining` is an old topic that dates back to the 1970s [116]: (1) if the join is not one-to-many, then there will be duplicated data, (2) prejoins are not necessarily faster than joins, and (3) there is no data independence.  

- 关于非规范化/预连接的问题是一个源远流长的话题，追溯到1970年代：（1）如果连接不是一对多，则会有重复数据；（2）预连接不一定比连接快；（3）缺乏数据独立性

> Despite strong protestations抗议，反对 that SQL was terrible, by the end of the 2010s, almost every NoSQL DBMS added a SQL interface. Notable examples include DynamoDB PartiQL [56], Cassandra CQL [15], Aerospike AQL [9], and Couchbase SQL++ [72]. The last holdout was MongoDB, but they added SQL for their Atlas service in 2021 [42]. Instead of supporting the SQL standard for DDL and DML operations, NoSQL vendors claim that they support their own proprietary私有的，专有的 query language derived or inspired from SQL. For most applications, these distinctions are without merit. Any language differences between SQL and NoSQL derivatives are mostly due to JSON extensions and maintenance operations.  
> Many of the remaining NoSQL DBMSs also added strongly consistent (ACID) transactions (see Sec. 3.4). As such, the NoSQL message has morphed from “Do not use SQL – it is too slow!” to “Not only SQL” (i.e., SQL is fine for some things).  
> Adding SQL and ACID to a NoSQL DBMS lowers their intellectual distance from RDBMSs. The main differences between them seems to be JSON support and the fact that NoSQL vendors allow “schema later” databases. But the SQL standard added a JSON data type and operations in 2016 [165, 178]. And as RDBMSs continue to improve their “first five minutes” experience for developers, we believe that the two kinds of systems will soon be effectively identical.  
> Higher level languages are almost universally preferred to record-at-a-time notations as they require less code and provide greater data independence. Although we acknowledge that the first SQL optimizers were slow and ineffective, they have improved immensely in the last 50 years. But the optimizer remains the hardest part of building a DBMS. We suspect that this engineering burden was a contributing factor to why NoSQL systems originally chose to not support SQL.
## 2.4 Column-Family Databases
> There is another category of NoSQL systems that uses a data model called column-family (aka wide-column). Despite its name, column-family is not a columnar data model. Instead, it is a reduction of the document data model that only supports one level of nesting instead of arbitrary nesting; it is relation-like, but each record can have optional attributes, and cells can contain an array of values. The following example shows a mapping from user identifier keys to JSON documents that contain each user’s varying profile information:
```SQL
User1000 → { “name”: “Alice”,
“accounts”: [ 123, 456 ],
“email”: "xxx@xxx.edu” }
User1001 → { “name”: “Bob”,
“email”: [ “yyy@yyy.org”, “zzz@zzz.com” ] }
```
> The first column-family model DBMS was Google’s BigTable in 2004 [111]. Instead of adopting SQL and emerging columnar storage, Google used this data model with procedural client APIs. Other systems adopted the column-family model in an attempt to copy Google’s bespoke implementation. Most notable are Cassandra [14] and HBase [28]. They also copied BigTable’s limitations, including the lack of joins and secondary indexes.

- 它们还复制了BigTable的局限性，包括缺乏连接和二级索引。

> **Discussion**: All our comments in Sec. 2.3 about the document model are also applicable here. In the early 2010s, Google built RDBMSs on top of BigTable, including MegaStore [99] and the first version of Spanner. Since then, Google rewrote `Spanner` to remove the BigTable remnants残留，残余 [98], and it is now the primary database for many of its internal applications. Several NoSQL DBMSs deprecated their proprietary APIs in favor of SQL but still retain their non-relational architectures. Cassandra replaced their Thrift-API with a SQL-like language called CQL [15], and HBase now recommends the Phoenix SQL-frontend [57]. Google still offers BigTable as a cloud service, but the columnfamily model is <u>a singular outlier</u>奇异的异类 with the same disadvantages as NoSQL DBMSs.  

## 2.5 Text Search Engines
> Text search engines have existed for a long time, beginning with the seminal开创性的 SMART system in the 1960s [184]. SMART pioneered <u>information retrieval</u>信息检索 and the vector space model向量空间模型, now nearly universal in modern search engines, by tokenizing将...标记为 documents into a “bag of words” and then building full-text indexes (aka inverted indexes) on those tokens to support queries on their contents. The system was also cognizant知道，意识 of noise words (e.g., “the”, “a”), synonyms同义词 (e.g., “The Big Apple” is a synonym for “New York City”), salient突出的，显著的 keywords, and distance (e.g., “drought” often appears close to “climate change”).  
> The leading text search systems today include Elasticsearch [23] and Solr [70], which both use Lucene [38] as their internal search library. These systems offer good support for storing and indexing text data but offer `none-to-limited`没有或者有限的 transaction capabilities. This limitation means that a DBMS has to recover from data corruption by rebuilding the document index from scratch, which results in significant downtime停机时间.  
- 今天领先的文本搜索系统包括Elasticsearch和Solr，它们都使用Lucene作为其内部搜索库。这些系统提供了良好的文本数据存储和索引支持，但提供的事务能力几乎没有。
> All the leading RDBMSs support full-text search indexes, including Oracle [52], Microsoft SQL Server [52], MySQL [43], and PostgreSQL [62]. Their search features have improved recently and are generally <u>on par with</u>与...水平相当 the special-purpose systems above. They also have the advantage of built-in transaction support. But their integration of search operations in SQL is often clunky笨重，沉重 and differs between DBMSs.  
- 它们的搜索功能最近有所改进，并在一般上与上述专用系统平起平坐。它们还具有内置事务支持的优势。然而，在SQL中集成搜索操作往往笨拙，并且在不同的DBMS之间有所差异
> **Discussion**: Text data is inherently固有的 unstructured, which means that there is no data model. Instead, a DBMS seeks to extract structure (i.e., meta-data, indexes) from text to avoid “<u>needle in the haystack</u>大海捞针” sequential searches. There are three ways to manage text data in application. First, one can run multiple systems, such as Elasticsearch for text and a RDBMS for operational workloads. This approach allows one to run “<u>best of breed</u>最佳组合” systems but requires additional ETL plumbing to push data from the operational DBMS to the text DBMS and to rewrite applications to route queries to the right DBMSs based on their needs. Alternatively, one can run a RDBMS with good text-search integration capabilities but with divergent APIs in SQL. This latter issue is often overcome by application frameworks that hide this complexity (e.g., Django Haystack [20]). The third option is a polystore system [187] that masks the system differences via middleware that exposes a unified interface.  
- 首先，可以运行多个系统，如为文本使用Elasticsearch，为操作工作负载使用RDBMS。这种方法允许运行“最佳组合”的系统，但需要额外的ETL管道将数据从操作DBMS推送到文本DBMS，并重写应用程序以根据需求将查询路由到合适的DBMS
- 其次，可以运行具有良好文本搜索集成能力的RDBMS，但其SQL中的API各不相同, 这后一种问题通常通过隐藏这一复杂性的应用框架来解决（例如，Django Haystack）。
- 第三种选择是一个多存储系统，经过中间件掩饰系统差异，提供统一的接口。

> Inverted index-centric search engines based on SMART are used for exact match searches. These methods have been supplanted in recent years by similarity search using ML-generated embeddings (see Sec. 2.7).  

- 基于SMART的倒排索引中心搜索引擎用于精确匹配搜索。这些方法近年来被使用机器学习生成的嵌入的相似搜索所取代。

## 2.6 Array Databases
> There are many areas of computing where arrays are an obvious data representation. We use the term “array” to mean all variants of them [182]: vectors (one dimension – see Sec. 2.7), matrices (two dimensions), and tensors张量 (three or more dimensions). For example, scientific surveys for geographic regions usually represent data as a multi-dimensional array that stores sensor measurements using location/time-based coordinates:  
```
(latitude, longitude, time, [vector-of-values])
```
> Several other data sets look like this, including genomic基因组，染色体 sequencing and computational fluid dynamics. Arrays are also the core of most ML data sets. Although array-based programming languages have existed since the 1960s (APL [142]), the initial work on array DBMSs began in the 1980s. `PICDMS is considered to be the first DBMS implementation using the array data model` [114]. The two oldest array DBMSs still being developed today are Rasdaman [66, 103] and kdb+ [34]. Newer array DBMSs include SciDB [54, 191] and TileDB [76]. `HDF5 [29] and NetCDF [46] are popular array file formats for scientific data`.  
> There are several system challenges with storing and querying real-world array data sets. Foremost is that array data does not always align to a regular integer grid; for example, geospatial地理空间的 data is often split into irregular shapes. An application can map such grids to integer coordinates via metadata describing this mapping [166]. Hence, most applications maintain array and non-array data together in a single database.  
- 绝大多数应用程序在同一数据库中同时维护数组和非数组数据
> Unlike row-or column-based DBMSs, querying array data in arbitrary dimensions presents unique challenges. The difficulty arises from storing multi-dimensional array data on a linear physical storage medium like a disk. To overcome these challenges, array DBMSs must employ indexing and data structures to support efficient traversal across array dimensions.  
- 与基于行或列的DBMS不同，在任意维度上查询数组数据提出了独特的挑战。这是由于多维数组数据在诸如磁盘这样的线性物理存储介质上的存储困难。
- 数组DBMS必须采用索引和数据结构，以支持跨数组维度的高效遍历
> **Discussion**: Array DBMSs are a niche market小众市场 that has only seen adoption in specific verticals (we discuss vector DBMSs next). For example, they have considerable traction in the genomics space. HDF5 is popular for satellite imagery and other gridded scientific data. But business applications rarely use dedicated array DBMSs, which is necessary for any product to survive. No major cloud provider offers a hosted array DBMS service, meaning they do not see a sizable market.  
> The challenge that array DBMS vendors have always faced is that the SQL includes support for ordered arrays as first-class data types (despite this being against the original RM proposal [115]). The first proposal to extend the unordered set-based RM with ordered rasters was in 1993 [155]. An early example of this was Illustra’s temporal (one-dimensional) data plugin [31]. SQL:1999 introduced limited support for single-dimension, fixed-length array data types. SQL:2003 expanded to support nested arrays without a predefined maximum cardinality. Later entrants include Oracle Georaster [4] and Teradata [73]. Data cubes are special-purpose arrays [135], but columnar RDBMSs have eclipsed them for OLAP workloads because of their better flexibility and lower engineering costs [113]. More recently, the SQL:2023 standard includes support for true multi-dimensional arrays (SQL/MDA) that is heavily inspired by Rasdaman’s RQL [166]. This update allows SQL to represent arrays with arbitrary dimensions using integer-based coordinates. In effect, this allows data cubes to exist in a SQL framework, but columnar DBMSs now dominate this market.
## 2.7 Vector Databases
> Similar to how the column-family model is a reduction of the document model, the vector data model simplifies the array data model to one-dimensional rasters. Given that vector DBMSs are attracting the most attention right now from developers and investors (similar to the NoSQL fad), it is necessary to discuss them separately. The reason for this interest is because developers use them to store single-dimension embeddings generated from AI tools. These tools use learned transformations to convert a record’s data (e.g., text, image) into a vector representing its latent semantics. For example, one could convert each Wikipedia article into an embedding using Google BERT and store them in a vector database along with additional article meta-data: 
```
(title, date, author, [embedding-vector])
```
> The size of these embedding vectors range from 100s of dimensions for simple transformers to 1000s for highend models; these sizes will obviously grow over time with the development of more sophisticated models.  

- 这些嵌入向量的大小范围从几百维（对于简单的变换）到几千维（对于高端模型），这些大小随着更复杂模型的发展将显著增加。

> The key difference between vector and array DBMSs is their query patterns. The former are designed for similarity searches that find records whose vectors have the shortest distance to a given input vector in a highdimensional space. The input vector is another embedding generated with the same transformer used to populate the database. Unlike array DBMSs, applications do not use vector DBMSs to search for matches at an offset in a vector nor extract slices across multiple vectors. Instead, the dominant use case is this similarity search.  
> To avoid brute force scans for finding the most similar records, vector DBMSs build indexes to accelerate approximate nearest neighbor (ANN) searches. Applications issue queries with predicates on both the embedding index and non-embedding attributes (i.e., metadata). The DBMS then chooses whether to use the nonembedding predicate on records before (pre-filter) or after (post-filter) the vector search.  
> There are dozens of new DBMSs in this emerging category, with Pinecone [58], Milvus [40], and Weaviate [84] as the leading systems. Text search engines, including Elasticsearch [23], Solr [70], and Vespa [79], expanded their APIs to support vector search. Other DBMSs rebranded重塑 themselves as vector databases to <u>jump on the bandwagon</u>跟风,随大流, such as Kdb+ [34].  
> One compelling feature of vector DBMSs is that they provide better integration with AI tools (e.g., Chat- GPT [16], LangChain [36]) than RDBMSs. These systems natively support transforming a record’s data into an embedding upon insertion using these tools and then uses the same transformation to convert a query’s input arguments into an embedding to perform the ANN search; other DBMSs require the application to perform these transformations outside of the database.  
> **Discussion**: Unlike array DBMSs that require a customized storage manager and execution engine to support efficient operations on multi-dimensional data, vector DBMSs are essentially document-oriented DBMSs with specialized ANN indexes. Such indexes are a feature, not the foundation of a new system architecture.  
> After LLMs became “mainstream” with ChatGPT in late 2022, it took less than one year for several RDBMSs to add their own vector search extensions. In 2023, many of the major RDBMSs added vector indexes, including Oracle [7], SingleStore [137], Rockset [8], and Clickhouse [157]. Contrast this with JSON support in RDBMSs. NoSQL systems like MongoDB and CouchDB became popular in the late 2000s and it took several years for RDBMSs to add support for it.  
> There are two likely explanations for the quick proliferation of vector indexes. The first is that similarity search via embeddings is such a compelling use case that every DBMS vendor rushed out their version and announced it immediately. The second is that the engineering effort to introduce a new index data structure is small enough that it did not take that much work for the DBMS vendors to add vector search. Most of them did not write their vector index from scratch and instead integrated an open-source library (e.g., pgVector [145], DiskANN [19], FAISS [24]).  
- 有两个可能的解释说明了向量索引快速传播的原因。首先，通过嵌入的相似性搜索是一个引人注目的用例，因此每个DBMS供应商都急切推出自己的版本并立即进行宣传。第二是引入新的索引数据结构的工程工作量足够小，以至于对DBMS供应商来说并没有太大工作。这些供应商中大多数并没有从头编写其向量索引，而是集成了开源库（例如pgVector、DiskANN、FAISS）
> We anticipate that vector DBMSs will undergo the same evolution as document DBMSs by adding features to become more relational-like (e.g., SQL, transactions, extensibility). Meanwhile, relational incumbents对手 will have added vector indexes to their already long list of features and moved on to the next emerging trend.  
## 2.8 Graph Databases
> There has been a lot of academic and industry interest in the last decade in graph databases [183]. Many applications use knowledge graphs to model semi-structured information. Social media applications inherently contain graph-oriented relationships (“likes”, “friend-of”). Relational design tools provide users with an entityrelationship (ER) model of their database. An ER diagram is a graph; thus, this paradigm has clear use cases. The two most prevalent approaches to represent graphs are (1) the resource description framework (RDF) and (2) property graphs [126]. With property graphs, the DBMS maintains a directed multi-graph structure that supports key/value labels for nodes and edges. RDF databases (aka triplestores) only model a directed graph with labeled edges. Since property graphs are more common and are a superset of RDF, we will only discuss them. We consider two use cases for graph DBMSs and discuss the problems that will limit their adoption.  
> The first category of systems are for operational / OLTP workloads: an application, for example, adds a friend link in the database by updating a single record, presumably in a transactional manner. Neo4j [44] is the most popular graph DBMS for OLTP applications. It supports edges using pointers (as in CODASYL) but it does not cluster nodes with their “parent” or “offspring”. Such an architecture is advantageous for traversing long edge chains since it will do pointer chasing, whereas a RDBMS has to do this via joins. But their potential market success comes down to whether there are enough “long chain” scenarios that merit forgoing a RDBMS.  
> The second use case is analytics, which seeks to derive information from the graph. An example of this scenario is finding which user has the most friends under 30 years old. Notable entries like Tigergraph [74] and JanusGraph [32] focus on query languages and storage on a graph DBMS. Other systems, such as Giraph [26] and Turi [78] (formerly Graphlab [27]) provide a computing fabric to support parallel execution of graph-oriented programs, typically written by a user.  
> Unlike queries in relational analytics that are characterized by chains of joins, queries for graph analytics contain operations like shortest path最短路径, cut set割集, or clique determination团体判定等. Algorithm choice and data representation will determine a DBMS’s performance. This argues for a computing fabric that allows developers to write their own algorithms using an abstraction that hides the underlying system topology. However, previous research shows that `distributed algorithms rarely outperform single-node implementations because of communication costs` [160]. A better strategy is to compress a graph into a space-efficient data structure that fits in memory on a single node and then run the query against this data structure. All but the largest graph databases are probably best handled this way.  
- 然而，之前的研究表明，分布式算法通常无法超越单节点实现，因为通信成本过高。
- 因此，一个更好的策略是将图压缩到一个空间高效的数据结构中，以便在单节点内存中运行查询。
- 除了最大的图形数据库 之外，其他所有数据库都可能以这种方式处理
> **Discussion**: Regardless of whether a graph DBMS targets OLTP or OLAP workloads, the key challenge these systems have to overcome is that it is possible to simulate a graph as a collection of tables: Node (node_id, node_data) Edge (node_id_1, node_id_2, edge_data) This means that RDBMSs are always an option to support graphs. But “vanilla” SQL is not expressive enough for graph queries and thus require multiple client-server roundtrips for traversal operations. Some RDBMSs, including MSSQL [3] and Oracle [50], provide built-in SQL extensions that make storing and querying graph data easier. Other DBMSs use a translation layer on top of relations to support graph-oriented APIs. Amazon Neptune [45] is a graph-oriented veneer on top of Aurora MySQL. Apache AGE provides an OpenCypher interface on top of PostgreSQL [10].  
> More recently, SQL:2023 introduced property graph queries (SQL/PGQ) for defining and traversing graphs in a RDBMS [196]. The syntax builds on existing languages (e.g., Neo4j’s Cypher [49], Oracle’s PGQL [51], and TigerGraph’s GSQL [75]), and shares aspects of the emerging GQL standard [126]. Thus, SQL/PGQ further narrows the functionality difference between RDBMSs and native graph DBMSs.  
-  SQL/PGQ进一步缩小了RDBMS和原生图DBMS之间的功能差异
> The question is whether graph DBMS vendors can make their specialized systems fast enough to overcome the above disadvantages. There have been several performance studies showing that graph simulation on RDBMSs outperform graph DBMSs [130, 143]. More recent work showed how SQL/PGQ in DuckDB outperforms a leading graph DBMS by up to 10⇥ [196]. This trend will continue with further improvements in worstcase optimal joins [132, 168] and factorized execution algorithms [100] for graph queries in RDBMSs.
- 关键问题是图DBMS供应商能否使其专用系统足够快，以克服以上劣势。多项性能研究表明，RDBMS上的图模拟的性能优于图DBMS。更近期的工作显示，DuckDB中的SQL/PGQ比领先的图DBMS快了多达10倍。随着最坏情况下的优化连接和深度执行算法的进一步改进，这一趋势将继续

## 2.9 Summary
> A reasonable conclusion from the above section is that non-SQL, non-relational systems are either a niche market or are fast becoming SQL/RM systems. Specifically:   
> **MapReduce Systems**: They died years ago and are, at best, a legacy technology at present. 
> **Key-value Stores**: Many have either matured into RM systems or are only used for specific problems. These can generally be equaled or beaten by modern high-performance RDBMSs. 
> **Document Databases**: Such NoSQL systems are on a collision course with RDBMSs. The differences between the two kinds of systems have diminished over time and should become nearly indistinguishable in the future. 
> **Column-Family Systems**: These remain a niche market. Without Google, this paper would not be talking about this category. • Text Search Engines: These systems are used for text fields in a polystore architecture. It would be valuable if RDBMSs had a better story for search so these would not have to be a separate product. 
> **Array Databases**: Scientific applications will continue to ignore RDBMSs in favor of bespoke array systems. They may become more important because RDBMSs cannot efficiently store and analyze arrays despite new SQL/MDA enhancements. 
> **Vector Databases**: They are single-purpose DBMSs with indexes to accelerate nearest-neighbor search. RM DBMSs should soon provide native support for these data structures and search methods using their extendable type system that will render such specialized databases unnecessary. 
> **Graph Databases**: OLTP graph applications will be largely served by RDBMSs. In addition, analytic graph applications have unique requirements that are best done in main memory with specialized data structures. RDBMSs will provide graph-centric APIs on top of SQL or via extensions. We do not expect specialized graph DBMSs to be a large market.  
> Beyond the above, there are also proposals to rebrand previous data models as something novel. For example, graph-relational [158] is the same as the semantic data model [202]. Likewise, document-relational is the document model with foreign keys [199]. Others provide a non-SQL veneer over a RDBMS (e.g., PRQL [64], Malloy [39]). Although these languages deal with some of SQL’s shortcomings, they are not compelling enough to overcome its entrenched固有的，根深蒂固的 userbase and ecosystem.