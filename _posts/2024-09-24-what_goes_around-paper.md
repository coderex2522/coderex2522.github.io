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
> For our discussion here, we group the research and development thrusts in data models and query languages for database into eight categories.

# MapReduce Systems
> Google constructed their MapReduce (MR) framework in 2003 as a “`point solution`” for processing its periodic crawl of the internet [122]. At the time, Google had <u>little expertis</u>经验有限 in DBMS technology, and they built MR to meet their crawl needs. In database terms, `Map is a user-defined function` (UDF) that performs computation and/or filtering while `Reduce is a GROUP BY operation`. <u>To a first approximation</u>初步来看, MR runs a single query: 
> `SELECT map() FROM crawl_table GROUP BY reduce()` 
> Google’s MR approach did not <u>prescribe</u>规定 a specific data model or query language. Rather, it was up to the Map and Reduce functions written in a procedural MR program to <u>parse and decipher</u>分析和解读 the contents of data files.

- 谷歌的MR方法并未规定特定的数据模型或查询语言。相反，解析和解读数据文件内容的任务落在了作为程序运行的Map和Reduce函数上。

> There was a lot of interest in MR-based systems at other companies in the late 2000s. Yahoo! developed an open-source version of MR in 2005, called Hadoop. It ran on top of a distributed file system HDFS that was a clone of the Google File System [134]. Several startups were formed to support Hadoop in the commercial marketplace. We will use MR to refer to the Google implementation and Hadoop to refer to the open-source version. They are functionally similar. 
> There was a <u>controversy</u>争论，争议 about the value of Hadoop compared to RDBMSs designed for OLAP workloads. This <u>culminated</u>达到顶点/巅峰 in a 2009 study that showed that data warehouse DBMSs outperformed Hadoop [172]. This generated <u>dueling</u>决斗，争论 articles from Google and the DBMS community [123, 190]. Google argued that with careful engineering, a MR system will beat DBMSs, and a user does not have to load data with a schema before running queries on it. Thus, MR is better for “one shot” tasks, such as text processing and ETL operations. The DBMS community argued that MR incurs performance problems due to its design that existing parallel DBMSs already solved. Furthermore, the use of higher-level languages (SQL) operating over partitioned tables has proven to be a good programming model [127]. A lot of the discussion in the two papers was on implementation issues (e.g., indexing, parsing, push vs. pull query processing, failure recovery). From reading both papers a reasonable conclusion would be that there is a place for both kinds of systems. However, two changes in the technology world rendered the debate moot. The first event was that the Hadoop technology and services market cratered in the 2010s. Many enterprises spent a lot of money on Hadoop clusters, only to find there was little interest in this functionality. Developers found it difficult to shoehorn their application into the restricted MR/Hadoop paradigm. There were considerable efforts to provide a SQL and RM interface on top of Hadoop, most notable was Meta’s Hive [30, 197]. The next event occurred eight months after the CACM article when Google announced that they were moving their crawl processing from MR to BigTable [164]. The reason was that Google needed to interactively update its crawl database in real time but MR was a batch system. Google finally announced in 2014 that MR had no place in their technology stack and killed it off [194]. The first event left the three leading Hadoop vendors (Cloudera, Hortonworks, MapR) without a viable product to sell. Cloudera rebranded Hadoop to mean the whole stack (application, Hadoop, HDFS). In a further sleight-of-hand, Cloudera built a RDBMS, Impala [150], on top of HDFS but not using Hadoop. They realized that Hadoop had no place as an internal interface in a SQL DBMS, and they configured it out of their stack with software built directly on HDFS. In a similar vein, MapR built Drill [22] directly on HDFS, and Meta created Presto [185] to replace Hive. Discussion: MR’s deficiencies were so significant that it could not be saved despite the adoption and enthusiasm from the developer community. Hadoop died about a decade ago, leaving a legacy of HDFS clusters in enterprises and a collection of companies dedicated to making money from them. At present, HDFS has lost its luster, as enterprises realize that there are better distributed storage alternatives [124]. Meanwhile, distributed RDBMSs are thriving, especially in the cloud. Some aspects of MR system implementations related to scalability, elasticity, and fault tolerance are carried over into distributed RDBMSs. MR also brought about the revival of shared-disk architectures with disaggregated storage, subsequently giving rise to open-source file formats and data lakes (see Sec. 3.3). Hadoop’s limitations opened the door for other data processing platforms, namely Spark [201] and Flink [109]. Both systems started as better implementations of MR with procedural APIs but have since added support for SQL [105].