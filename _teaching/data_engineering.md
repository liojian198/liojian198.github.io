---
title: "开源数据工程"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/data_engineering
date: 2025-08-25
---

# 简介 

  记得以前读书的时候，我们老师就告诉我们，数学的分支那么多，不用一个一个去吃透，你就“看个大概”(这个看个大概现在理解了，是把各个领域的内容组织到自己的知识体系里面去，这样才有机会用到，要不然根本不知道会用到相关的知识。)，到时候用到了再去查。 当时对这句话的理解是，等用到了再学， 慢慢的发现了，永远用不到，哈哈哈。还是要通过手段在顶层将知识加入到自己的知识体系里面去，才能灵活的应用起来。对自己的要求也很简单，要有广度的知识，也要有深度学习知识的能力，领域交叉的能力。

  参考这位老哥的文章(https://www.pracdata.io/p/open-source-data-engineering-landscape-2025,https://substack.com/@alirezasadeghi1，很有深度。后续再解读。

  https://www.pracdata.io/

  参考https://github.com/pracdata/awesome-open-source-data-engineering?tab=readme-ov-file.也是那位老哥搞得。年纪轻轻，

  # 开局上图

  分析平台和数据工程生态系统中使用的开源工具的精选列表

  <img src='/images/data_engineering/image.png'>

# 详细介绍

## STORAGE SYSTEMS  存储系统

  从更高的层次去看存储系统，如下图

  <img src='/images/data_engineering/image_storage_system.png'>


### OLAP Extensions & HTAS  OLAP 扩展和 HTAS

 These extensions allow to seamlessly extend OLTP databases, transforming these systems into HTAP (Hybrid Transactional/Analytical Processing) or new HTAS (Hybrid Transactional Analytical Storage) database engine that integrate headless data storage—like data lakes and lakehouses—with transactional database systems.

 参考文章: https://jack-vanlightly.com/blog/2024/5/2/hybrid-transactional-analytical-storage

  <img src='/images/data_engineering/image_htas.png'>


### Zero-Disk Architecture  零磁盘架构

  零磁盘架构可能是存储系统中最具变革性的趋势，从根本上改变了数据库系统管理存储和计算层的方式。

  这种架构方法完全消除了对本地附加磁盘的需求，而是使用 S3 对象存储等远程深度存储解决方案作为主要持久层。

  除了 OLAP 存储系统（例如云数据仓库和开放表格式）之外，我们还见证了这种模式在 NoSQL、实时、流式和事务系统中的显着出现.

  基于磁盘的系统与零磁盘系统的主要 权衡是成本与性能 ， 以及读取和写入物理存储数据的 I/O 延迟 。虽然基于磁盘的系统可以管理快速的亚毫秒级 I/O，但零磁盘系统通过廉价的可扩展对象存储实现了规模经济性，但代价是在向对象存储服务读取和写入数据时面临长达一秒的延迟。

  参考文章: https://www.pracdata.io/p/zero-disk-architecture-the-future

  <img src='/images/data_engineering/image_zero_disk.png'>

### Relational DBMS  关系型 DBMS

  PostgreSQL - 高级对象关系数据库管理系统

  MySQL - 最受欢迎的开源数据库之一

  MariaDB - 一个流行的 MySQL 服务器分支

  Supabase - 开源 Firebase 替代品

  SQlite - 最流行的嵌入式数据库引擎  

### Distributed SQL DBMS  分布式 SQL DBMS

  Citus - 作为扩展的流行分布式 PostgreSQL

  CockroachDB - 云原生分布式 SQL 数据库

  YugabyteDB - 云原生分布式 SQL 数据库

  TiDB - 云原生、分布式、兼容 MySQL 的数据库

  OceanBase - 可扩展的分布式关系数据库

  ShardingSphere - 分布式 SQL 事务和查询引擎

  Neon - AWS Aurora Postgres 的无服务器开源替代方案

  CrateDB - 一个分布式且可扩展的 PostgreSQL 兼容 SQL 数据库

### Cache Store  缓存存储

  Redis - 一种流行的基于键值的缓存存储

  Memcached - 高性能多线程键值缓存存储

  Dragonfly - 与 Redis 和 Memcached API 兼容的现代缓存存储

### In-memory SQL Database  内存中 SQL 数据库

  Apache Ignite - 一种分布式、符合 ACID 标准的内存中 DBMS

  ReadySet - MySQL 和 Postgres 有线兼容的缓存层

  VoltDB - 分布式、水平可扩展、符合 ACID 标准的数据库

### Document Store  文档存储

MongoDB - 一个跨平台、面向文档的 NoSQL 数据库

RavenDB - 一个 ACID NoSQL 文档数据库

RethinkDB 重新思考数据库 | ⚠️ 非活动 |- 面向实时应用程序的面向文档的分布式数据库

CouchDB - 一个可扩展的面向文档的 NoSQL 数据库

Couchbase - 现代云原生 NoSQL 分布式数据库

FerretDB - 真正的开源 MongoDB 替代品！

LowDB 低数据库 | ⚠️ 非活动 |- 简单快速的 JSON 数据库

### NoSQL Multi-model  NoSQL 多模型

OrientDB - 支持图形、文档、响应式、全文和地理空间模型的多模型数据库管理系统

ArrangoDB - 一个多模型数据库，具有灵活的文档、图形和键值数据模型

SurrealDB - 一个可扩展的、分布式的、协作的文档图数据库

EdgeDB - 具有声明式模式的图关系数据库

### Graph Database  图数据库

Neo4j - 高性能领先的图数据库

JunasGraph - 高度可扩展的分布式图数据库

HugeGraph - 一个快速且高度可扩展的图形数据库

NebulaGraph - 分布式、水平可扩展性、快速开源图数据库
Cayley 凯利 | ⚠️ 非活动 |- 灵感来自谷歌知识图谱背后的图形数据库

Dgraph - 具有图形后端的水平可扩展分布式 GraphQL 数据库

Apache Age - 作为 PostgreSQL 扩展的图形数据库

FalkorDB - 一个在后台使用 GraphBLAS 的图形数据库，专为 LLM 量身定制

### Distributed Key-value Store 分布式键值存储


Riak  ⚠️ 非活动 - Basho Technologies 的去中心化键值数据存储

FoundationDB - 来自 Apple 的分布式事务键值存储

etcd - 用 Go 编写的分布式可靠键值存储

TiKV - 一个分布式事务键值数据库，最初是为了补充 TiDB 而创建的

Immudb - 具有内置加密证明和验证的数据库

Valkey - 从 Redis 分叉的分布式键值数据存储

Apache Kvrocks - 使用 RocksDB 作为存储引擎的分布式键值数据库

### Wide-column Key-value Store 宽列键值存储

Apache Cassandra - 一个高度可扩展的基于 LSM 树的分区行存储

Apache Hbase - 一个以 Google 的 Bigtable 为模型的面向宽列的分布式存储

Scylla - 基于 LSM 树的宽列 API，与 Apache Cassandra 和 Amazon DynamoDB 兼容

Apache Accumulo - 基于 Hadoop 的分布式键值存储，具有可扩展的数据存储和检索功能

### Embedded Key-value Store  嵌入式键值存储

LevelDB | ⚠️ 非活动 |- Google 编写的快速键值存储库

RocksDB - 由 Meta （Facebook） 开发的可嵌入的持久键值存储

MyRocks - MySQL 的 RocksDB 存储引擎

BadgerDB - 一个用纯 Go 编写的可嵌入、快速键值数据库

### Search Engine  搜索引擎

Apache Solr - 基于 Apache Lucene 构建的快速分布式搜索数据库

Elastic Search - 针对速度进行优化的分布式 RESTful 搜索引擎
Sphinx 狮身人面像 | ⚠️ 非活动 |- 具有高速索引的全文搜索引擎

Meilisearch - 具有强大集成支持的快速搜索 API

OpenSearch - Elasticsearch 和 Kibana 的社区驱动的开源分支

Quickwit - 用于可观测性数据的快速云原生搜索引擎

ParadeDB - 基于 Postgres 构建的搜索引擎

### Streaming Database  流式数据库

RisingWave - 用于流处理、分析和管理的可扩展 Postgres

Materialize - 专为运营工作负载构建的实时数据仓库

EventStoreDB - 专为事件溯源和事件驱动架构而设计的事件原生数据库

KsqlDB - 用于在 Apache Kafka 之上构建流处理应用程序的数据库

Timeplus Proton - 流式 SQL 引擎，快速轻量级，由 ClickHouse 提供支持

Fluss - 一种流式存储，充当湖屋体系结构的实时数据层

### Time-Series Database  时间序列数据库

Influxdb - 用于指标、事件和实时分析的可扩展数据存储

TimeScaleDB - 打包为 PostgreSQL 扩展的快速引入时间序列 SQL 数据库

Apache IoTDB - 与 Hadoop 和 Spark 生态无缝集成的物联网数据库

Netflix Atlas - 由 Netflix 开发并开源的 n 内存维度时间序列数据库

QuestDB - 用于快速摄取和 SQL 查询的时间序列数据库

TDEngine - 针对物联网 （IoT） 优化的高性能云原生时序数据库
KairosDB 凯罗斯数据库 | ⚠️ 非活动 |- 用 Java 编写的可扩展时间序列数据库

GreptimeDB - 用于指标、日志和事件的云原生统一时间序列数据库

HoraeDB - 分布式云原生时序数据库

### Columnar OLAP Database  列式 OLAP 数据库

Apache Kudu - Apache Hadoop 生态系统的面向列的数据存储

Greeenplum 格林普 |⛔️ 已存档 |- 用于分析的面向列的大规模并行 PostgreSQL

MonetDB - 最初由 CWI 数据库研究小组开发的高性能列式数据库

Databend - 一个用 Rust 构建的最后的、工作负载感知的云原生数据仓库

ByConity - 从 ClickHouse 分叉而来的云原生数据仓库

Hydra 九头蛇 | ⚠️ 非活动 |- 面向列的 Postgres 扩展

### Real-time OLAP Engine  实时 OLAP 引擎

ClickHouse - 最初由 Yandex 开发的面向列的实时数据库

Apache Pinot - 由 LinkedIn 开源的实时分布式 OLAP 数据存储

Apache Druid - 由 Metamarkets 开发并开源的高性能实时 OLAP 引擎

Apache Kylin - 一个分布式 OLAP 引擎，旨在提供对 Hadoop 的多维分析

Apache Doris - 基于 MPP 架构的高性能实时分析数据库

StarRocks - 支持多维分析的亚秒级 OLAP 数据库（Linux 基金会项目）

### In-process OLAP Engine  进程内 OLAP 引擎

DuckDB - 进程内 SQL OLAP 数据库管理系统

GlareDB - 用于跨分布式数据运行分析的 SQL 数据库

Apache DataFusion - 具有 SQL 和 Dataframe API 的可扩展查询引擎

chdb - 由 ClickHouse 提供支持的进程内 OLAP SQL 引擎

SlateDB - 基于对象存储构建的云原生嵌入式存储引擎

### OLAP Extensions  OLAP 扩展

pg_duckdb - 嵌入 DuckDB 分析引擎的 Postgres 扩展

pg_analytics - DuckDB 支持的 Postgres 分析扩展

pg_mooncake - 基于 DuckDB 的 Postres 的列式存储扩展

pg_parquet - 用于读取和写入数据湖 Parquet 文件的 Postgres 扩展

## DATA LAKE PLATFORM  数据湖平台

### Distributed File System  分布式文件系统

Apache Hadoop HDFS - 高度可扩展的基于块的分布式文件系统

GlusterFS FS | ⚠️ 非活动 |- 可扩展到数 PB 的可扩展分布式存储

JuiceFS - 基于 Redis 和 S3 构建的分布式 POSIX 文件系统

Lustre - 一个分布式并行文件系统，专门用于提供符合 POSIX 的全局命名空间

### Distributed Object Store  分布式对象存储

Apache Ozone - 适用于 Apache Hadoop 的可扩展、冗余和分布式对象存储

Ceph - 分布式对象、块和文件存储平台

Minio - 与 Amazon S3 API 兼容的高性能对象存储

Garage - 一种与 S3 兼容的分布式对象存储，专为中小型自托管而设计

### Serialisation Framework  序列化框架

Apache Parquet - 一种高效的列式二进制存储格式，支持嵌套数据

Apache Avro - 一个高效、快速的基于行的二进制序列化框架

Apache ORC - 一种专为 Hadoop 设计的自描述类型感知列式文件格式

Lance - 一种用 Rust 实现的 ML 和 LLM 的现代列式数据格式

Vortex - 一种高度可扩展且快速的列式文件格式

Arrow Feather - 一种用于存储箭头表或数据帧的便携式文件格式

### Open Table Format  开放表格式

Apache Hudi - 一种开放表格式，旨在支持在云和 Hadoop 上增量数据摄取

Apache Iceberg - 一种高性能表格式，适用于 Netflix 开发的大型分析表

Delta Lake - 由 Databricks 开发的用于构建湖屋体系结构的存储框架

Apache Paimon - 一个支持流式高速数据摄取的 Apache inclubing 项目

OpenHouse - 具有开放数据湖屋格式的数据服务的声明式目录

### Native Open Table Format Library 原生开放表格式库

Delta-rs - Delta Lake 的原生 Rust 库，与 Python 绑定

PyIceberg - 一个用于与 Iceberg 表格式交互的原生 Python 库

Hudi-rs- Apache Hudi 的原生 Rust 库，与 Python 绑定

### Universal Lakehouse  通用数据湖

Apache XTable - 一个统一的框架，支持跨多种开源表格式的互作性

Apache Amoro - 基于开放数据湖格式构建的湖屋管理系统

## DATA INTEGRATION  数据集成

### Data Integration Platform 数据集成平台

Airbyte - 具有广泛连接器的 ETL / ELT 数据管道的数据集成平台

Apache Nifi - 一个可靠、可扩展的低代码数据集成平台，具有良好的企业支持

Apache Camel - 支持多种企业集成模式的嵌入式集成框架

Apache Gobblin - 由 LinkedIn 构建的分布式数据集成框架，支持流式数据和批处理数据

Apache Inlong - 支持海量数据的集成框架，最初在腾讯构建

Meltano - 声明式代码优先数据集成引擎

Apache SeaTunnel - 支持多种摄取模式的高性能分布式数据集成工具

Estuary Flow - 用于快速数据集成的实时 ETL 和数据管道平台

dlt - 一个轻量级的数据集成库，用于 Python 优先数据平台

### CDC Tool  CDC 工具

Debezium - 支持各种数据库的变更数据捕获框架

Kafka Connect - 支持 CDC 的 Apache Kafka 之上的流数据集成框架和运行时

Redpanda Conenct - Redpanda 之上的数据流和集成框架

Flink CDC - 支持不同数据库的 Apache Flink 引擎的 CDC 连接器

Brooklin  | ⚠️ 非活动 |- 用于在各种异构源和目标系统之间流式传输数据的分布式平台

RudderStack - 用于构建数据管道的无头客户数据平台，是 Segment 的开放替代方案

Artie Transfer - OLTP 和 OLAP 数据库之间的实时 CDC 复制解决方案

Dozer  - 一种基于 CDC 的实时数据集成工具，可在各种源和汇之间进行

PeerDB - 一种 CDC 工具，用于将数据从 Postgres 复制到数据仓库、队列和其他存储

### Data Migration  数据迁移

DBmate - 一个轻量级的、与框架无关的数据库迁移工具。

Ingestr - 一种 CLI 工具，用于使用单个命令在任何数据库之间复制数据

Sling — 用于将数据从源传输到目标存储/数据库的 CLI 工具

### Log & Event Collection  日志和事件收集

CloudQuery - 一种 ETL 工具，用于将数据从云 API 同步到各种受支持的目的地

Snowplow | ⚠️ 非活动 |- 用于收集行为数据并加载到各种云存储系统中的云原生引擎

EventMesh - 一个无服务器事件中间战，用于收集事件数据并将其加载到各种目标中
Apache Flume ⚠️ 非活动 |- 可扩展的分布式日志聚合服务

Steampipe - 一种零 ETL 解决方案，用于直接从 API 和服务获取数据

Jitsu - 一个完全可编写脚本的数据摄取引擎，用于收集事件数据

### Event Hub  事件中心

Apache Kafka - 高度可扩展的分布式事件存储和流平台

NSQ - 一个实时分布式消息传递平台，旨在大规模运行

Apache Pulsar - 可扩展的分布式发布-订阅消息传递系统

Apache RocketMQ - 云原生消息传递和流媒体平台

Redpanda - 高性能的 Kafka API 兼容流数据平台

Memphis ⚠️ 非活动 |- 用于构建事件驱动应用程序的可扩展数据流平台

AutoMQ - 使用 S3 作为主存储层的 Kafka 的云优先替代方案

### Reverse ETL  反向 ETL

Multiwoven - Hightouch 和 RudderStack 的反向 ETL 开源替代方案

## DATA PROCESSING AND COMPUTATION 数据处理和计算

### Unified Processing  统一处理

Apache Beam - 支持在流行的分布式处理后端执行的统一编程模型

Apache Spark - 用于大规模数据处理的统一分析引擎

Dinky - 基于 Apache Flink 的统一流式处理计算平台

Feldora - 统一的增量计算引擎

### Batch processing  批处理

Hadoop MapReduce - Apache Hadoop 项目的高度可扩展的分布式批处理框架

Apache Tez - 为 Apache Hive 和 Hadoop 构建的分布式数据处理管道

### Stream Processing  流处理

Apache Flink - 可扩展的高吞吐量流处理框架

Apache Samza - 一个使用 Kafka 和 Hadoop 的分布式流处理框架，最初由 LinkedIn 开发

Apache Storm - 基于 Actor Model 框架的分布式实时计算系统

Akka - 基于 Actor 模型的高并发、分布式、消息驱动的处理系统

Bytewax - 一个带有 Rust 分布式处理引擎的 Python 流处理框架

Timeplus Proton - 流式 SQL 引擎，快速轻量级，由 ClickHouse 提供支持

FastStream - 一个用于与事件流（如 Apache Kafka）交互的 Python 框架

Bento - 来自 WarpStream Labs 的流处理引擎（从 Benthos 分叉而来）

Fluvio - 一个用 Rust 和 Web 汇编编写的精益分布式流处理系统

Arroyo - 用 Rust 编写的分布式流处理引擎

### Python Processing Framework Python 处理框架

Polars - 一个带有矢量化查询引擎的多线程数据帧，用 Rust 编写

PySpark - Python 中 Apache Spark 的接口

Vaex - 用于大型表格数据集的高性能 Python 库。

Apache Arrow - 一种高效的内存数据格式

Ibis - 一个支持许多引擎后端的便携式 Python 数据帧库

SQLFrame - 用于数据转换的 Spark DataFrame API 兼容库

Daft - 一个分布式查询引擎，用于使用 Python 或 SQL 进行大规模数据处理

cuDF - GPU 加速的 pandas API dataFrame 库

### Python Workflow Scaling  Python 工作流扩展

Dask - 具有任务调度功能的灵活并行计算库

RAY - 具有分布式运行时的统一框架，用于扩展 Python 应用程序

Modin - 用于将 Pandas 工作流程扩展到多区执行的库

Pandaral·lel | ⚠️ 非活动 |- 一个库，用于在所有可用 CPU 上并行化 Pandas 作

### SQL Toolkit  SQL 工具包

SQLAlchemy - Python SQL 工具包和对象关系映射器

SQLGlot - Python SQL 解析器和转译器

## WORKFLOW MANAGEMENT & DATAOPS 工作流管理和数据运营

### Workflow Orchestration  工作流编排

Apache Airflow - 一种平台，用于创建和调度工作流作为任务的有向无环图 （DAG）

Prefect - 基于 Python 的工作流编排工具

Argo - 一个容器原生工作流引擎，用于在 Kubernetes 上编排并行作业
Azkaban ⚠️ 非活动 |- 在 LinkedIn 上创建的批处理工作流作业调度程序，用于运行 Hadoop 作业

Cadence - 支持不同语言客户端库的分布式、可扩展的可用编排

Dagster - 用 Python 编写的云原生数据管道编排器

Apache DolpinScheduler - 低代码高性能工作流编排平台

Luigi - 一个 python 库，用于构建复杂的批处理作业管道

Flyte - 适用于数据和 ML 工作负载的可扩展且灵活的工作流编排平台

Kestra - 一个与声明性语言无关的 worfklow 编排和调度平台

Mage.ai - 用于集成、编制和管理数据管道的平台

Temporal - 一个弹性工作流管理系统，起源于 Uber 的 Cadence 的一个分支

Windmill - 快速工作流程引擎，以及 Airplane 和 Retool 的开源替代品

Maestro - 由 Netflix 开发的通用工作流程编排器

### Job Scheduling  作业调度

Celery - Python 的分布式任务队列系统

DKron - 分布式、容错作业调度系统

ApScheduler - Python 的高级任务调度器和任务队列系统

### Data Quality  数据质量

Data-diff 已存档 |- 用于比较数据库内或数据库之间的表的工具

远大前程 - 用 Python 编写的数据验证和分析工具

Deeque - 基于 Apache Spark 的库，用于测量大型数据集中的数据质量

Pandera - 轻量级、灵活、富有表现力的统计数据测试库

Soda - 用于数据质量测试的 CLI 工具和 Python 库

Pydantic - 使用 Python 类型提示的数据验证库

### Data Versioning  数据版本控制

LakeFS - 存储在数据湖中的数据的数据版本控制

Project Nessie - 具有类似 Git 语义的数据湖的事务性目录

DVC - 用于数据和 ML 实验的数据版本控制工具

Dolt - Git for data 工具

Git-lfs - 用于对大文件进行版本控制的 Git 扩展

Datachain - 一个基于 Python 的框架，用于对非结构化数据进行版本控制

### Data Modeling  数据建模

dbt - 数据管道的数据建模和转换工具

SQLMesh - 向后兼容 dbt 的数据转换和建模框架

### Pipeline Observability  管道可观测性

Elementry - 用于监控数据管道的 dbt 原生数据可观测性解决方案

## DATA INFRASTRUCTURE  数据基础设施

### Resource Scheduling  资源调度

Apache Yarn - Apache Hadoop 集群的默认资源调度程序

Apache Mesos - 由加州大学伯克利分校博士生开发的资源调度和集群资源抽象框架

Kubernetes - 生产级容器调度和管理工具

Apache YuniKorn - 用于容器编排器系统的轻量级通用资源调度器

Docker - 流行的作系统级虚拟化和容器化软件

### Cluster Administration  集群管理

Apache Ambari - 用于配置、管理和监控 Apache Hadoop 集群的工具

Apache Helix - 在 LinkedIn 开发的通用集群管理框架

### Security  安全

Apache Knox - 用于管理对 Hadoop 集群的访问的网关和 SSO 服务

Apache Ranger - Hadoop 和其他流行服务的安全和治理平台

Kerberos - 一种流行的企业网络身份验证协议

### Metrics Store  指标存储

Influxdb - 用于指标和事件的可扩展数据存储

Mimir - 由 Grafana Labs 开发的 Prometheus 的可扩展长期指标存储

OpenTSDB - 基于 Apache Hbase 编写的分布式、可扩展的时间序列数据库

M3 - 分布式 TSDB 和指标存储和聚合器

### Observability Framework  可观测性框架

Prometheus - 一种流行的指标收集和管理工具

ELK - 由 Elasticsearch、Kibana、Beats 和 Logstash 组成的 poular 可观测性堆栈

Graphite - 成熟的基础设施监控和可观测性系统

OpenTelemetry - 用于管理和监控指标的 API、SDK 和工具的集合

VictoriaMetrics - 具有时间序列数据库的可扩展监控解决方案

Zabbix - 实时基础设施和应用程序监控服务

### Monitoring Dashboard  监控仪表板

Grafana - 一个流行的开放且可组合的可观测性和数据可视化平台

Kibana - Elasticsearch 的可视化和搜索仪表板

Redpanda 控制台 - 用于监控和管理 Apache Kafka 和 Redpanda 工作负载的 UI

### Log & Metrics Pipeline  日志和指标管道

Fluentd - A metric collection, buffering and router service
Fluentd - 指标收集、缓冲和路由器服务
Fluent Bit - A fast log processor and forwarder, and part of the Fluentd ecosystem
Fluent Bit - 快速日志处理器和转发器，是 Fluentd 生态系统的一部分
Logstash - A server-side log and metric transport and processor, as part of the ELK stack
Logstash - 服务器端日志和指标传输和处理器，作为 ELK 堆栈的一部分
Telegraf - A plugin-driven server agent for collecting & reporting metrics developed by Influxdata
Telegraf - 由 Influxdata 开发的用于收集和报告指标的插件驱动的服务器代理
Vector - A high-performance, end-to-end (agent & aggregator) observability data pipeline
Vector - 高性能、端到端（代理和聚合器）可观测性数据管道
StatsD | ⚠️ Inactive | - A network daemon for collection, aggregation and routing of metrics
统计 D | ⚠️ 非活动 |- 用于收集、聚合和路由指标的网络守护程序

### Cost Management  成本管理

OpenCost - Kubernetes 工作负载和云成本的成本监控

## METADATA MANAGEMENT  元数据管理

### Metadata Platform  元数据平台

Amundsen - 由 Lyft 工程师开发的数据发现和元数据引擎

Apache Atlas - Apache Hadoop 生态系统的数据可观测性平台

DataHub - Netflix 开发的现代数据堆栈元数据平台

Marquez - 用于收集、聚合和可视化元数据的元数据服务

ckan - 用于编目、管理和访问数据的数据管理系统

Open Metadata 开放元数据 - 使用中央元数据存储库进行发现和治理的统一平台

ODD 平台 - 数据发现和可观测性平台

### Open Standards  开放标准

Open Lineage  世系元数据收集的开放标准

Open Metadata  一个统一的元数据平台，提供用于管理元数据的开放平台

Egeria - 开放元数据和治理标准以促进元数据交换

### Schema & Catalog Service  架构和目录服务

Hive 元存储 - 作为 Apache Hive 项目的一部分的常用架构管理和元存储服务

Confluent Schema Registry - 由 Confluent 开发的 Kafka 架构注册表

Apache Polaris - Apache Iceberg 的可互作开源目录

Unity 目录 - 数据湖屋格式和其他数据/AI 资产的通用目录

Lakekeeper - Rust 原生 Apache Iceberg REST 目录

Apache Gravitino - 地理分布式和联合开放数据目录

## ANALYTICS & VISUALISATION 分析与可视化

### BI & Dashboard  BI 和仪表板

Apache Superset - 一个 poular 开源数据可视化和数据探索平台

Metabase - 一个简单的数据可视化和探索仪表板

Redash - 一种用于浏览、查询、可视化和与许多数据源连接器共享数据的工具

Lightdash - 将 dbt 项目转变为全栈 BI 平台的自助式 BI

### BI as Code (Web App) BI 即代码（Web 应用程序)

Streamlit - 一个将数据打包和共享为 Web 应用程序的 python 工具

证据 - 一种在纯 SQL 和 Markdown 中构建交互式数据可视化的工具

dash - 用于构建 ML 和数据科学 Web 应用程序的 Python 框架

Vizro - 用于创建模块化数据可视化应用程序的工具包

Mercury - 将 Jupyter Notebook 转换为 Web 应用程序的工具

Quary - 基于代码的 BI 解决方案

### Query & Collaboration  查询与协作

Hue - 由 Cloudera 开发的具有 Hadoop 生态系统支持的查询和数据探索工具

Apache Zeppelin - 用于 Hadoop 交互式数据分析和协作的基于 Web 的笔记本

Querybook - 由 Pinterest 开发的简单查询和笔记本 UI

Jupyter - 一种流行的基于 Web 的交互式笔记本应用程序

IPython - 用于数据分析的增强型交互式 Python shell

Datasette - 用于浏览和发布数据的工具

### MPP Query Engine  MPP 查询引擎

Apache Hive - Hadoop 之上的数据仓库和 MPP 引擎

Apache Implala - 主要用于 Hadoop 集群的 MPP 引擎，由 Cloudera 开发

Presto - 用于大数据的分布式 SQL 查询引擎

Trino - 以前的 PrestoSQL 分布式 SQL 查询引擎

Apache Drill - 针对 NoSQL 和 Hadoop 数据存储系统的分布式 MPP 查询引擎

DataFusion Ballista - 基于 Apache DataFusion 的分布式查询执行引擎

### Semantic & Middleware Layer 语义和中间件层

Alluxio - 数据编排和虚拟分布式存储系统

Cube - 用于构建支持流行 databse 引擎的数据应用程序的语义层

Apache Linkis - 一种计算中间件，用于促进应用程序和数据引擎之间的连接和编排

Apache Gluten - 用于将基于 JVM 的 SQL 引擎执行卸载到本机引擎的中间层

Apache OpenDAL - 一个开放的数据访问 Llyer，支持与各种存储服务的无缝交互

### Data Sharing  数据共享

delta-sharing - 一种用于安全实时交换大型数据集的开放协议

## ML/AI PLATFORM  ML/AI 平台

### Vector Storage  矢量存储

milvus - 云原生矢量数据库，用于 AI 应用的存储

qdrant - 用于 AI 的高性能、可扩展的 Vector 数据库

chroma - 用于构建 LLM 应用程序的 AI 原生嵌入数据库

marqo - 文本和图像的端到端矢量搜索引擎

LanceDB - 一个用 Rust 编写的 AI 应用程序的无服务器向量数据库

weaviate - 可扩展的云原生支持对象和矢量存储

deeplake - 用于深度学习应用的存储格式优化 AI 数据库

Vespa - 用于组织向量、张量、文本和结构化数据的存储

vald - 可扩展的分布式近似最近邻 （ANN） 密集向量搜索引擎

pgvector - 作为 Postgres 扩展的向量相似性搜索

### MLOps

mlflow - 一个简化机器学习开发和生命周期管理的平台

Metaflow - 一种用于构建和管理 ML/AI 和数据科学项目的工具，由 Netflix 开发

SkyPilot - 用于在任何云上运行 LLM、AI 和批处理作业的框架

Jina - 使用云原生堆栈构建多模态 AI 应用程序的工具

NNI |⛔️ 已存档 |- 用于自动化机器学习生命周期的 autoML 工具包，来自 Microsoft

BentoML - 用于构建可靠且可扩展的 AI 应用程序的框架

Determined AI - 一个简化分布式训练、调优和实验跟踪的 ML 平台

RAY - 用于扩展 AI 和 Python 应用程序的统一框架

kubeflow - 用于 ML 作的云原生平台 - 管道、训练和部署

Kedro - 用于构建生产就绪数据科学和 ML 工作流程的工具箱和框架

Pachyderm - 可计算的 ML 和数据科学数据处理工作流管理平台

### LLMOps

Dify - 具有 AI 工作流程、RAG 管道和模型管理的 LLM 开发平台

Haystack - 用于构建可定制、生产就绪的 LLM 应用程序的 AI 编排框架

Superduper - 一个基于 Python 的框架，用于构建 AI 数据工作流程和应用程序

Cognee - 用于实现 LLM 工作流程的 LLM 内存引擎

vLLM - 用于 LLM 的高吞吐量和内存效率推理和服务引擎
