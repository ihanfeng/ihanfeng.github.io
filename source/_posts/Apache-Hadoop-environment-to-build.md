
---
title: Hadoop环境搭建
date: 2017-11-07 21:49:23
tags:
  - 大数据
  - Hadoop
  - CDH5
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
---
# 摘要

 “工欲善其事，必先利其器”，在学习Hadoop之前，我们需要有个好的开发环境。

# 版本选择

## Apache Hadoop

Hadoop是Apache顶级项目，我们可以直接从Apache官网下载。

官网：[http://hadoop.apache.org/releases.html](http://hadoop.apache.org/releases.html)

目前稳定版本为 [2.8.2](http://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.8.2/hadoop-2.8.2.tar.gz)

## CDH

Cloidera提供Hadoop支持、咨询和管理工具的公司，在Hadoop生态圈地位举足轻重。其著名产品Cloudera's Distribution for Hadoop，即CDH。

CDH包括Hadoop、HBase、Hive、Pig、Sqoop、Flume、Xookeeper、Oozie、Mahout等，几乎覆盖整个Hadoop生态圈，保证组件之间的兼容性。

CDH最新版本CDH5，基于Hadoop 2.3.
CDH经典版本CDH3，基于Hadoop0.20.2，生产稳定版本。

笔者在学习中决定采用最新的CDH5进行研究。

下载地址：[官网](https://www.cloudera.com/downloads/cdh/5-13-0.html)
[CDH5](http://archive.cloudera.com/cdh5/cdh/5/)

<!-- more  -->

# Hadoop架构

Hadoop主要由分布式系统HDFS和分布式计算框架MapReduce构成。

分布式文件系统主要用于海量数据的存储，MapReduce则是基于分布式文件系统对存储在分布式文件系统中的数据进行分布式计算。

## Hadoop HDFS架构

构成HDFS集群的主要是两类节点，并以主从（master/slave）模式，即一个NameNode和多个DataNode，还有一个节点叫SecondaryNameNode，作为NameNode镜像数据备份。

守护进程 | 集群数目 | 作用
----|------|----
NameNode | 1  | 存储文件系统的元数据，存储文件与数据块映射，并提供文件系统的全景图
SecondaryNameNod | 1  | 备份NameNode数据，并负责镜像与NameNode日志数据的合并
DataNode | N个（N>1）  | 存储块数据


{% asset_img hdfs-core.png %}

## MapReduce架构

构成MapReduce集群为两类节点，JobTracker和TaskTracker，采用主从（master/slave）架构。

JobTracker和TaskTracker也是两种守护进程，运行在各自的节点上。客户端负责用户作业提交。

守护进程 | 集群数目 | 作用
--------|-------|------
TaskTracker|1|负责接受客户端作业提交，调度任务到TaskTracker上运行，并提供监控TaskTracker及任务进度等管理功能
TaskTracke|N个（N>1）|实例化用户程序，在本地执行任务并周期性地向JobTracker汇报状态。

{% asset_img mapreduce-core.png %}

## Hadoop架构

Hadoop集群部署方式。

{% asset_img hadoop-cluster-1.png %}

{% asset_img hadoop-cluster-2.png %}

# 环境准备
## 硬件配置

操作系统| win10 64位 1709
---|---
CPU|Intel I5 7500
内存|金士顿 DDR4 2400 8G
硬盘|100G以上
## 软件配置
虚拟机| VMWare 14 Pro
---|---
操作系统| CentOS7
JDK|jdk-8u131-linux-x64.rpm
远程工具| xshell 5
Hadoop组件|hadoop-2.6.0-cdh5.13.0.tar.gz<br>hbase-1.2.0-cdh5.13.0.tar.gz<br>hive-1.1.0-cdh5.13.0.tar.gz<br>pig-0.12.0-cdh5.13.0.tar.gz<br>oozie-4.1.0-cdh5.13.0.tar.gz<br>sqoop-1.4.6-cdh5.13.0.tar.gz<br>mahout-0.9-cdh5.13.0.tar.gz<br>flume-ng-1.6.0-cdh5.13.0.tar.gz<br>zookeeper-3.4.5-cdh5.13.0.tar.gz


# 安装Hadoop

## Hadoop运行模式


1. 单机模式

  Hadoop默认模式，不进行任何配置，所有守护进程都是一个Java进程。

2. 伪分布式模式

  所有的守护进程都运行在一个节点上，模拟了一个具有Hadoop完整功能的微型集群。

3. 完全分布式模式

  Hadoop的守护进程运行在多个节点上，形成一个真正意义上的集群。

伪分布式模式需要1个虚拟机。  
完全分布式模式需要至少2台以上虚拟机。节点需要3个或者5个，保持奇数。

## Hadoop安装步骤

基于完全分布式模式2主3从方式

序号|IP|Host|部署模块|进程
---|---|---|---|---
1|192.168.128.101|hadoop-master-1|NameNode、ResourceManager、JournalNode、zookeeper|NameNode、ResourceManager、JournalNode、zookeeper
2|192.168.128.102|hadoop-master-2|NameNode、ResourceManager、JournalNode、zookeeper|NameNode、ResourceManager、JournalNode、zookeeper
3|192.168.128.103|hadoop-slave-1|DataNode、JournalNode、zookeeper|DataNode、JournalNode、zookeeper
4|192.168.128.104|hadoop-slave-2|DataNode、JournalNode|DataNode、JournalNode
5|192.168.128.105|hadoop-slave-3|DataNode、JournalNode|DataNode、JournalNode

### 安装克隆运行环境

通过VMWare虚拟机克隆基础镜像centos7-base创建5台新的虚拟机，centos7-hadoop-master-1、centos7-hadoop-master-2、centos7-hadoop-slave-1、centos7-hadoop-slave-2、centos7-hadoop-slave-3。

### 设置主机名和用户名

要查看主机名相关的设置：
```shell
hostnamectl
## 或者  
hostnamectl status
```
要同时修改所有三个主机名：静态、瞬态和灵活主机名：
```shell
 hostnamectl set-hostname hadoop-master-1
```
新增用户名hadoop
```shell
useradd hadoop
```
为用户hadoop设置密码
```shell
passwd hadoop
```
统一密码设置为 `vm2017`

设置hosts，方便Hadoop各个节点之间能够通过主机名进行互相访问。
```shell
sudo  vi /etc/hosts
```
文本后面添加
```shell
192.168.128.101 hadoop-master-1
192.168.128.102 hadoop-master-2
192.168.128.103 hadoop-slave-1
192.168.128.104 hadoop-slave-2
192.168.128.105 hadoop-slave-3
```

### 配置静态IP地址
```shell
sudo  vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
修改IPADDR属性为对应的IP地址，记住几个虚拟机之间IP不可重复。

重启网络
```shell
service network restart
```
### 配置SSH无密码连接

1. 关闭防火墙

  ```shell
  systemctl stop firewalld.service #停止firewall
  systemctl disable firewalld.service #禁止firewall开机启动
  firewall-cmd --state #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
  ```
2. 检查SSH是否安装

  ```shell
  rpm -qa |grep openssh
  ```

  返回如下表示已经安装

  ```shell
  openssh-7.4p1-13.el7_4.x86_64
  openssh-server-7.4p1-13.el7_4.x86_64
  openssh-clients-7.4p1-13.el7_4.x86_64
  ```

  ```shell
  rpm -qa |grep rsync
  ```
返回如下表示已经安装

  ```shell
  rsync-3.0.9-18.el7.x86_64
  ```

3. 生成SSH公钥

  对于完全分布式模式，有多个节点，但是只需要主节点无密码连接从节点，因此需要在主节点生成公钥
  ```shell
  ssh-keygen -t rsa
  ```
  遇到提示回车即可。

  将公钥发至从节点的authorized_keys的列表
  ```shell
  ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@hadoop-slave-1
  ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@hadoop-slave-2
  ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@hadoop-slave-3
  ```

  执行结果：
  ```shell
  [hadoop@hadoop-master-2 ~]$ ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@hadoop-slave-1
  /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/hadoop/.ssh/id_rsa.pub"
  The authenticity of host 'hadoop-slave-1 (192.168.128.103)' can't be established.
  ECDSA key fingerprint is SHA256:DChFHDTUaoTvDRo+TEzLxWWr9nkyID0813NaD/LAfew.
  ECDSA key fingerprint is MD5:91:bd:81:17:d3:25:7e:09:ac:8e:45:c8:f8:a3:c9:39.
  Are you sure you want to continue connecting (yes/no)? yes
  /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
  /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
  hadoop@hadoop-slave-1's password:

  Number of key(s) added: 1

  Now try logging into the machine, with:   "ssh 'hadoop@hadoop-slave-1'"
  and check to make sure that only the key(s) you wanted were added.

  [hadoop@hadoop-master-2 ~]$ ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@hadoop-slave-2
  /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/hadoop/.ssh/id_rsa.pub"
  The authenticity of host 'hadoop-slave-2 (192.168.128.104)' can't be established.
  ECDSA key fingerprint is SHA256:DChFHDTUaoTvDRo+TEzLxWWr9nkyID0813NaD/LAfew.
  ECDSA key fingerprint is MD5:91:bd:81:17:d3:25:7e:09:ac:8e:45:c8:f8:a3:c9:39.
  Are you sure you want to continue connecting (yes/no)? yes
  /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
  /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
  hadoop@hadoop-slave-2's password:

  Number of key(s) added: 1

  Now try logging into the machine, with:   "ssh 'hadoop@hadoop-slave-2'"
  and check to make sure that only the key(s) you wanted were added.

  [hadoop@hadoop-master-2 ~]$ ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@hadoop-slave-3
  /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/hadoop/.ssh/id_rsa.pub"
  The authenticity of host 'hadoop-slave-3 (192.168.128.105)' can't be established.
  ECDSA key fingerprint is SHA256:DChFHDTUaoTvDRo+TEzLxWWr9nkyID0813NaD/LAfew.
  ECDSA key fingerprint is MD5:91:bd:81:17:d3:25:7e:09:ac:8e:45:c8:f8:a3:c9:39.
  Are you sure you want to continue connecting (yes/no)? yes
  /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
  /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
  hadoop@hadoop-slave-3's password:

  Number of key(s) added: 1

  Now try logging into the machine, with:   "ssh 'hadoop@hadoop-slave-3'"
  and check to make sure that only the key(s) you wanted were added.
  ```
4. 验证安装（以hadoop用户执行）

对于完全分布式模式，在主节点执行
```shell
ssh hadoop-slave-1
```
如果没有出现输入密码的提示则表示安装成功。
```shell
[hadoop@hadoop-master-1 ~]$ ssh hadoop-slave-1
Last login: Thu Nov  9 05:03:42 2017 from 192.168.128.1
[hadoop@hadoop-slave-1 ~]$ exit
logout
Connection to hadoop-slave-1 closed.
[hadoop@hadoop-master-1 ~]$ ssh hadoop-slave-2
Last login: Thu Nov  9 05:03:54 2017 from 192.168.128.1
[hadoop@hadoop-slave-2 ~]$ exit
logout
Connection to hadoop-slave-2 closed.
[hadoop@hadoop-master-1 ~]$ ssh hadoop-slave-3
Last login: Thu Nov  9 05:04:28 2017 from 192.168.128.1
[hadoop@hadoop-slave-3 ~]$ exit
logout
Connection to hadoop-slave-3 closed.
[hadoop@hadoop-master-1 ~]$
```
### 配置Hadoop

### 格式HDFS

### 启动Hadoop并验证安装

# 安装Hive
