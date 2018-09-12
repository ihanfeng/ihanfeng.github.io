---
title: Hadoop生态系统
tags:
  - 大数据
  - Hadoop
  - 生态系统
  - HDFS
  - Yarn
  - Flume
  - MapReduce
  - Hive
  - Pig
  - Mahout
  - HBase
  - Zookeeper
categories:
  - 大数据
  - Hadoop
abbrlink: 41053
date: 2017-11-07 19:45:00
---

# 认识Hadoop

``Hadoop``是Apache开源组织的一个分布式计算开源框架[http://hadoop.apache.org/](http://hadoop.apache.org/),可编写和运行分布式应用处理大规模数据。 Hadoop框架的核心是HDFS和MapReduce。其中 HDFS 是分布式文件系统，MapReduce 是分布式数据处理模型和执行环境。

特点：

1. 运行方便：Hadoop是运行在由一般商用机器构成的大型集群上。Hadoop在云计算服务层次中属于PaaS(Platform-as-a- Service)：平台即服务。
2. 健壮性：Hadoop致力于在一般的商用硬件上运行，能够从容的处理类似硬件失效这类的故障。
3. 可扩展性：Hadoop通过增加集群节点，可以线性地扩展以处理更大的数据集。
4. 简单：Hadoop允许用户快速编写高效的并行代码。

<!-- more -->

# 生态圈

## Hadoop的生态系统

{% asset_img hadoop-ecosystem-map.jpg %}

2. Nutch，互联网数据及Nutch搜索引擎应用
3. HDFS,Hadoop的分布式文件系统
5. MapReduce,分布式计算框架
6. Flume、Scribe，Chukwa数据收集，收集非结构化数据的工具。
7. Hiho、Sqoop,讲关系数据库中的数据导入HDFS的工具
8. Hive数据仓库，pig分析数据的工具
10. Oozie作业流调度引擎
11. Hue，Hadoop自己的监控管理工具
12. Avro 数据序列化工具
13. mahout数据挖掘工具
14. Hbase分布式的面向列的开源数据库

## 生态系统特点

- 源码开放
- 社区活跃，参与者众多
- 涉及分布式存储和计算的方方面面
- 得到业界的验证

## 生态组件

Hadoop1.0时代的生态系统

{% asset_img hadoop-ecosystem-map-1.x.jpg %}

Hadoop2.0时代的生态系统

{% asset_img hadoop-ecosystem-map-2.x.jpg %}

### Hadoop核心

{% asset_img hadoop-core.jpg %}

由上图可以看出Hadoop1.0与Hadoop2.0的区别。Hadoop1.0的核心由HDFS（Hadoop Distributed File System）和MapReduce(分布式计算框架)构成。而在Hadoop2.0中增加了Yarn(Yet Another Resource Negotiator),来负责集群资源的统一管理和调度。

### HDFS(分布式文件系统)

HDFS源自于Google发表于2003年10月的GFS论文，也即是说HDFS是GFS的克隆
版。

HDFS具有如下特点：
- 良好的扩展性
- 高容错性
- 适合PB级以上海量数据的存储

HDFS的基本原理
- 将文件切分成等大的数据块，存储到多台机器上
- 将数据切分、容错、负载均衡等功能透明化
- 可将HDFS看成容量巨大、具有高容错性的磁盘

HDFS的应用场景
- 海量数据的可靠性存储
- 数据归档

### Yarn(资源管理系统)

Yarn是Hadoop2.0新增的系统，负责集群的资源管理和调度，使得多种计算框架可以运行在一个集群中。

Yarn具有如下特点：
- 良好的扩展性、高可用性
- 对多种数据类型的应用程序进行统一管理和资源调度
- 自带了多种用户调度器，适合共享集群环境

{% asset_img yarn.jpg %}

### MapReduce(分布式计算框架)

MapReduce源自于Google发表于2004年12月的MapReduce论文，也就是说，Hadoop MapReduce是Google MapReduce的克隆版。

MapReduce是一种编程模型，你要函数式编程思想，将对数据集处理的过程分为Map和Reduce两个阶段。

MapReduce具有如下特点：
- 良好的扩展性
- 高容错性
- 适合PB级以上海量数据的离线处理

{% asset_img map-reduce.jpg %}

### Hive(基于MR的数据仓库)

Hive由facebook开源，最初用于解决海量结构化的日志数据统计问题；是一种ETL(Extraction-Transformation-Loading)工具。它也是构建在Hadoop之上的数据仓库；数据计算使用MR,数据存储使用HDFS。

Hive定义了一种类似SQL查询语言的HiveQL查询语言，除了不支持更新、索引和事务，几乎SQL的其他特征都能支持。它通常用于离线数据处理（采用MapReduce);我们可以认为Hive的HiveQL语言是MapReduce语言的翻译器，把MapReduce程序简化为HiveQL语言。但有些复杂的MapReduce程序是无法用HiveQL来描述的。

Hive提供shell、JDBC/ODBC、Thrift、Web等接口。

<strong>Hive应用场景</strong>

- 日志分析：统计一个网站一个时间段内的pv、uv ；比如百度。淘宝等互联网公司使用hive进行日志分析
- 多维度数据分析
- 海量结构化数据离线分析
- 低成本进行数据分析（不直接编写MR）

{% asset_img hive.jpg %}

### Pig(数据仓库)

Pig由yahoo!开源，设计动机是提供一种基于MapReduce的ad-hoc数据分析工具。它通常用于进行离线分析。

Pig是构建在Hadoop之上的数据仓库，定义了一种类似于SQL的数据流语言–Pig Latin,Pig Latin可以完成排序、过滤、求和、关联等操作，可以支持自定义函数。Pig自动把Pig Latin映射为MapReduce作业，上传到集群运行，减少用户编写Java程序的苦恼。

Pig有三种运行方式：Grunt shell、脚本方式、嵌入式。

<strong>Pig与Hive的比较</strong>

{% asset_img ping-hive-compare.jpg %}

### Mahout(数据挖掘库)

Mahout是基于Hadoop的机器学习和数据挖掘的分布式计算框架。它实现了三大算法：推荐、聚类、分类。

{% asset_img mahout.jpg %}

### HBase(分布式数据库)

HBase源自Google发表于2006年11月的Bigtable论文。也就是说，HBase是Google Bigtable的克隆版。

HBase可以使用shell、web、api等多种方式访问。它是NoSQL的典型代表产品。

HBase的特点
- 高可靠性
- 高性能
- 面向列
- 良好的扩展性

HBase的数据模型

{% asset_img hbase-data-model.jpg %}

- Table（表）：类似于传统数据库中的表
- Column Family(列簇)：Table在水平方向有一个或者多个Column Family组成；一个Column Family 中可以由任意多个Column组成。
- Row Key(行健)：Table的主键；Table中的记录按照Row Key排序。
- Timestamp（时间戳）：每一行数据均对应一个时间戳；也可以当做版本号。

### Zookeeper(分布式协作服务)

Zookeeper源自Google发表于2006年11月的Chubby论文，也就是说Zookeeper是Chubby的克隆版。

Zookeeper解决分布式环境下数据管理问题：
- 统一命名
- 状态同步
- 集群管理
- 配置同步

Zookeeper的应用
- HDFS
- Yarn
- Storm
- HBase
- Flume
- Dubbo
- Metaq

### Sqoop（数据同步工具）

Sqoop是连接Hadoop与传统数据库之间的桥梁，它支持多种数据库，包括MySQL、DB2等；插拔式，用户可以根据需要支持新的数据库。

Sqoop实质上是一个MapReduce程序，充分利用MR并行的特点,充分利用MR的容错性。

{% asset_img sqoop.jpg %}

### Flume(日志收集工具)

Flume是Cloudera提供的一个高可用、高可靠、分布式的海量日志采集、聚合和传输系统，Flume支持在日志系统中定制各类数据发送方，用于手机数据。

Flume的特点
- 分布式
- 高可靠性
- 高容错性
- 易于定制与扩展

### Oozie(作业流调度系统)

目前计算框架和作业类型种类繁多：如MapReduce、Stream、HQL、Pig等。这些作业之间存在依赖关系，周期性作业，定时执行的作业，作业执行状态监控与报警等。如何对这些框架和作业进行统一管理和调度？

解决方案有多种：
- Linux Crontab
- 自己设计调度系统（淘宝等公司）
- 直接使用开源系统（Oozie）

{% asset_img oozie.jpg %}

### Kafka(消息队列)

高吞吐量的分布式分部订阅消息系统。
