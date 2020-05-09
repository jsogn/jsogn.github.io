---
layout: post
title: 阿里云RDS与ECS服务器数据库做主从
tags:
  - mysql
categories:
  - mysql
abbrlink: 13832
---

### 前言

实现RDS for mysql与线下ECS上自建数据库数据实时同步，阿里云官方推荐使用DTS方式进行。原因有两个：

- mysql-bin正常情况下,RDS在本地只保存18个小时

- 当RDS实例切换时，会影响自建ECS数据同步(这个经过测试可以排除)

考虑到使用DTS工具会产生不少的费用(长期使用)，另一方面，在数据库中一个地区对应一个库，后续业务无法事先规划好库名，此时如果使用dts可能需要购买多个通道，进行配置，比较费时费力且费钱。基于这两个原因的考虑，决定使用搭建主从复制方式来实现数据同步

<!--more-->

### 基础概念

传统的MYSQL主从就是主库每做一个操作会在binlog上做一个position，每做一个event就在binlog做一个起始编号、一个终止编号。然后主库把binlog传递给从库，然后从库根据这个binlog的pos值就按照顺序做一样的操作，达到两个数据库保持一致的目的。

GTID不用这个position的方式，而是用了全局事物标识，这个标识的格式是`source_id:transaction_id`，如`3E11FA47-71CA-11E1-9E33-C80AA9429562:23`

- source_id即是server_uuid，在第一次启动时生成(函数 generate_server_uuid)，并持久化到DATADIR/auto.cnf文件里

- transaction_id是顺序化的序列号(sequence number)，在每台 MySQL 服务器上都是从 1 开始自增长的序列，是事务的唯一标识

它的主从过程是这样的：主库更新数据时，会在事务前产生GTID，连通sql记录到binlog日志中。从库的i/o线程将变更的binlog写入到relay log中，读取值是根据gitd_next变量，告诉从库下一个执行哪个GTID。从库的sql线程从relay log中获取GTID，然后对比从库的的binlog是否有记录。如果有记录，说明该GTID的事务已经执行，从库会忽略。如果没有记录，从库就会从relay log中执行该GTID的事务，并记录到从库binlog。在解析过程中会判断是否有主键，如果没有就用二级索引，如果没有二级索引就用全部扫描。

也就是说，无论是级联情况，还是一主多从情况，都可以通过GTID自动找点儿，而无需像之前那样通过binlog和binlog_position找点儿了

### RDS数据库配置

- 配置从实例读取数据使用的只读账号和授权数据库
- 将ECS的从实例的 IP 地址加入主实例的 IP 白名单中
- 登录主实例

#### 查询 server-id

```mysql
mysql> show variables like 'server_id';
```

#### 查询 GTID

```mysql
mysql> show global variables like 'gtid_purged';
```

### ECS数据库配置

#### mysql文件配置

```conf
server-id = 1001 #不可与RDS主库id相同
port = 3306
replicate-do-db = masterdb #需要同步的数据库

binlog_format = row #日志文件格式
log-bin = mysql-bin
log-bin-index = mysql-bin.index
relay-log = relay-log
relay_log_index = relay-log.index
slave-skip-errors = all
```

#### GTID配置

```conf
gtid_mode = on
enforce_gtid_consistency = on
log-slave-updates = 1

sql_mode = NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

#### 从库配置

```mysql
mysql> stop slave;
mysql> set global gtid_purged = '533ac4e6-9565-11e8-abb5-7cd30abca02e:1-3099396';
```

>注意：设置gtid_purged值时，gtid_executed值必须为空否则报错，该值清空的方法就是reset  master命令

```mysql
mysql>reset master;
```

#### 执行同步

```mysql
CHANGE MASTER TO
MASTER_HOST='xxxxxxx.mysql.rds.aliyuncs.com',
MASTER_PORT=3306,
MASTER_USER='username',
MASTER_PASSWORD='password',
master_auto_position=1;

mysql>start slave;
```

#### 查看从库状态

```mysql
mysql>show slave status\G;
```

两个yes表示成功

```mysql
  Slave_IO_Running: Yes
  Slave_SQL_Running: Yes
```
