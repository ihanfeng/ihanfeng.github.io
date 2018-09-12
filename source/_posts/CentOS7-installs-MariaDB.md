---
title: CentOS7安装MariaDB
tags:
  - 数据库
  - MariaDB
categories:
  - 数据库
  - MariaDB
abbrlink: 11326
date: 2017-11-11 11:37:20
copyright: ture
---

# 摘要
CentOS 6 或早期的版本中提供的是 MySQL 的服务器/客户端安装包，但 CentOS 7 已使用了 MariaDB 替代了默认的 MySQL。MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可 MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品。

<!-- more -->

# 全部删除MySQL/MariaDB

MySQL 已经不再包含在 CentOS 7 的源中，而改用了 MariaDB;

##  删除现有软件包

使用`rpm -qa | grep mariadb`搜索 MariaDB 现有的包：

如果存在，使用`rpm -e --nodeps mariadb-*`全部删除：

```shell
[admin@dev ~]$ rpm -qa | grep mariadb
mariadb-libs-5.5.56-2.el7.x86_64
[admin@dev ~]$ rpm -e mysql-*
error: package mysql-* is not installed
[admin@dev ~]$
```
## 清理MySQL

使用`rpm -qa | grep mariadb`搜索 MariaDB 现有的包：

如果存在，使用`yum remove mysql mysql-server mysql-libs compat-mysql51`全部删除；

```shell
[admin@dev ~]$ sudo yum remove mysql mysql-server mysql-libs compat-mysql51
[sudo] password for admin:
Loaded plugins: fastestmirror
No Match for argument: mysql
No Match for argument: mysql-server
No Match for argument: compat-mysql51
Resolving Dependencies
--> Running transaction check
---> Package mariadb-libs.x86_64 1:5.5.56-2.el7 will be erased
--> Processing Dependency: libmysqlclient.so.18()(64bit) for package: 2:postfix-2.10.1-6.el7.x86_64
--> Processing Dependency: libmysqlclient.so.18(libmysqlclient_18)(64bit) for package: 2:postfix-2.10.1-6.el7.x86_64
--> Running transaction check
---> Package postfix.x86_64 2:2.10.1-6.el7 will be erased
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================
 Package               Arch            Version                 Repository          Size
========================================================================================
Removing:
 mariadb-libs          x86_64          1:5.5.56-2.el7          @anaconda          4.4 M
Removing for dependencies:
 postfix               x86_64          2:2.10.1-6.el7          @anaconda           12 M

Transaction Summary
========================================================================================
Remove  1 Package (+1 Dependent package)

Installed size: 17 M
Is this ok [y/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Erasing    : 2:postfix-2.10.1-6.el7.x86_64                                        1/2
  Erasing    : 1:mariadb-libs-5.5.56-2.el7.x86_64                                   2/2
  Verifying  : 1:mariadb-libs-5.5.56-2.el7.x86_64                                   1/2
  Verifying  : 2:postfix-2.10.1-6.el7.x86_64                                        2/2

Removed:
  mariadb-libs.x86_64 1:5.5.56-2.el7                                                    

Dependency Removed:
  postfix.x86_64 2:2.10.1-6.el7                                                         

Complete!
[admin@dev ~]$ rpm -qa|grep mariadb
[admin@dev ~]$

```
## 配置MariaDB

开始新的安装, 创建MariaDB.repo文件

```shell
sudo vi /etc/yum.repos.d/MariaDB.repo
```

新增以下内容：

```
# MariaDB 10.2 CentOS repository list - created 2017-11-11 03:24 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

系统及版本选择：[https://downloads.mariadb.org/mariadb/repositories/#mirror=tuna](https://downloads.mariadb.org/mariadb/repositories/#mirror=tuna)

##  安装MariaDB

```
sudo yum -y install MariaDB-server MariaDB-client
```
安装成功之后启动MariaDB服务。

```
systemctl start mariadb #启动服务
systemctl enable mariadb #设置开机启动
systemctl restart mariadb #重新启动
systemctl stop mariadb.service #停止MariaDB
```
登录到数据库

用mysql -uroot命令登录到MariaDB，此时root账户的密码为空。

## 初始化配置

进行MariaDB的相关简单配置,使用`mysql_secure_installation`命令进行配置。
```
mysql_secure_installation
```
初始化MariaDB完成，接下来测试登录
```
mysql -uroot -ppassword
```
## 配置MariaDB的字符集

查看`/etc/my.cnf`文件内容，其中包含一句`!includedir /etc/my.cnf.d `说明在该配置文件中引入`/etc/my.cnf.d` 目录下的配置文件。

使用`vi server.cnf`命令编辑server.cnf文件，在[mysqld]标签下添加
```
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
```
文件`/etc/my.cnf.d/mysql-clients.cnf`

在[mysql]中添加

```
default-character-set=utf8
```
全部配置完成，重启mariadb

```
systemctl restart mariadb
```
之后进入MariaDB查看字符集

```
mysql> show variables like "%character%";show variables like "%collation%";
```
显示结果:

```
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)

+----------------------+-----------------+
| Variable_name        | Value           |
+----------------------+-----------------+
| collation_connection | utf8_unicode_ci |
| collation_database   | utf8_unicode_ci |
| collation_server     | utf8_unicode_ci |
+----------------------+-----------------+
3 rows in set (0.00 sec)

```
字符集配置完成。

## 添加用户，设置权限

创建用户命令

```
MariaDB [(none)]> create user dev@localhost identified by 'dev123456';
Query OK, 0 rows affected (0.00 sec)

```

授予外网登陆权限

```
MariaDB [(none)]> grant all privileges on *.* to dev@'%' identified by 'dev123456';
Query OK, 0 rows affected (0.00 sec)
```
授予权限并且可以授权

```
grant all privileges on *.* to dev@'hostname' identified by 'dev123456' with grant option;
```
查看结果：

```
MariaDB [mysql]> select host,user,password from user;
+-----------+------+-------------------------------------------+
| host      | user | password                                  |
+-----------+------+-------------------------------------------+
| localhost | root | *84AAC12F54AB666ECFC2A83C676908C8BBC381B1 |
| 127.0.0.1 | root | *84AAC12F54AB666ECFC2A83C676908C8BBC381B1 |
| ::1       | root | *84AAC12F54AB666ECFC2A83C676908C8BBC381B1 |
| %         | dev  | *4830F76F782FF62FDDAD8358BF4F4795C98FB7DC |
| localhost | dev  | *4830F76F782FF62FDDAD8358BF4F4795C98FB7DC |
| hostname  | dev  | *4830F76F782FF62FDDAD8358BF4F4795C98FB7DC |
+-----------+------+-------------------------------------------+

```

防火墙记得开放3306端口
```shell
sudo firewall-cmd --zone=public --permanent --add-port=3306/tcp
service firewalld restart
```
