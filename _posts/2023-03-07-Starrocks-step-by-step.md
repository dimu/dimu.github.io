---
layout:     post
title:      starrocks
subtitle:   basic concept
date:       2023-03-07
author:     加菲猫
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - BigData
    - OLAP
    - Real DataWarehouse
---

## starrocks介绍



## 1.starrocks install 

### 1.1 k8s部署方式
1. 参考[k8s部署方式](https://docs.starrocks.io/zh-cn/latest/administration/sr_operator)

### 1.2 手动部署模式

#### 1.2.1 单FE单BE模式

参考官网

#### 1.2.2 集群扩缩容

- 1. 下载安装包，目前有些版本为测试版本，可以通过文件名区分   
	```
	wget https://releases.mirrorship.cn/starrocks/StarRocks-2.3.10.tar.gz
	tar -xzvf StarRocks-2.3.10.tar.gz
	```


### 1.3 stargo部署模式

stargo是个人开发者写的go语言部署端，尝试了下报cdn出错，并且个人开发者维护不积极，问题坑比较多，慎用



## starrock 连接方式

```
	mysql -hhost_ip -P30296(default：9030) -uroot
```

## ddl

### create database

starrocks兼容mysql指令，操作

```
	create database dwx_test;
	show databases;
	
```

### create table

注意事项

- 在 StarRocks 中，字段名不区分大小写，表名区分大小写。
- 建表时，DISTRIBUTED BY 为必填字段。


```
use dwx_test;
CREATE TABLE IF NOT EXISTS `detailDemo` (
    `recruit_date`  DATE           NOT NULL COMMENT "YYYY-MM-DD",
    `region_num`    TINYINT        COMMENT "range [-128, 127]",
    `num_plate`     SMALLINT       COMMENT "range [-32768, 32767] ",
    `tel`           INT            COMMENT "range [-2147483648, 2147483647]",
    `id`            BIGINT         COMMENT "range [-2^63 + 1 ~ 2^63 - 1]",
    `password`      LARGEINT       COMMENT "range [-2^127 + 1 ~ 2^127 - 1]",
    `name`          CHAR(20)       NOT NULL COMMENT "range char(m),m in (1-255)",
    `profile`       VARCHAR(500)   NOT NULL COMMENT "upper limit value 1048576 bytes",
    `hobby`         STRING         NOT NULL COMMENT "upper limit value 65533 bytes",
    `leave_time`    DATETIME       COMMENT "YYYY-MM-DD HH:MM:SS",
    `channel`       FLOAT          COMMENT "4 bytes",
    `income`        DOUBLE         COMMENT "8 bytes",
    `account`       DECIMAL(12,4)  COMMENT "",
    `ispass`        BOOLEAN        COMMENT "true/false"
) ENGINE=OLAP
DUPLICATE KEY(`recruit_date`, `region_num`)
PARTITION BY RANGE(`recruit_date`)
(
    PARTITION p20220311 VALUES [('2022-03-11'), ('2022-03-12')),
    PARTITION p20220312 VALUES [('2022-03-12'), ('2022-03-13')),
    PARTITION p20220313 VALUES [('2022-03-13'), ('2022-03-14')),
    PARTITION p20220314 VALUES [('2022-03-14'), ('2022-03-15')),
    PARTITION p20220315 VALUES [('2022-03-15'), ('2022-03-16'))
)
DISTRIBUTED BY HASH(`recruit_date`, `region_num`) BUCKETS 8
PROPERTIES (
    "replication_num" = "1" 
);

```

- starrock支持的数据类型

参考：https://docs.starrocks.io/zh-cn/latest/sql-reference/sql-statements/data-types/BIGINT

大致包含：

```
数值类型：

tinyint   1个字节
smallint 2个字节
int      4个字节
bigint   8个字节
largeint  16个字节 //比较特殊类型
decimal  
float   4个字节
double  8个字节
boolean

字符串类型：
char  最大255个字符
varchar 单位字节，2.1版本最长支持65533个字节，2.1版本以后支持1048576个字节
string  最大65533个字节

日期类型：
date 日期类型
datetime 日期时间类型

其他类型：
array  
bitmap  参考https://docs.starrocks.io/zh-cn/latest/sql-reference/sql-statements/data-types/BITMAP
json  参考：https://docs.starrocks.io/zh-cn/latest/sql-reference/sql-statements/data-types/JSON
hll HyperLogLog 用于近似去重，参考：https://docs.starrocks.io/zh-cn/latest/sql-reference/sql-statements/data-types/HLL

```


- 表信息
```
	show tables;  //查看数据库下面所有表
	desc xxxTable;  //查看表的列信息
	show create table xxxTable; //查看建表语句
```


### modify table

常见的修改表方式

```
  ALTER TABLE detailDemo ADD COLUMN uv BIGINT DEFAULT '0' after ispass;  //新增字段
  ALTER TABLE detailDemo DROP COLUMN uv;  //删除字段
```

## 3.data load

### 3.1 Broker load
**使用场景：**

### 3.2 Spark load


### 3.3 Stream load

**使用场景：**同步数据导入方式，用户通过http协议请求本地文件或者数据流导入到StarRocks中，并等待系统返回导入的结果状态，从而判断导入是否成功。该导入支持的数据源有**Apache Flink**或者**CSV文件**等。

```
语法:curl --location-trusted -u user:passwd [-H ""...] -T data.file -XPUT http://fe_host:http_port/api/{db}/{table}/_stream_load


例子：curl --location-trusted -u root: -T detailDemo_data -H "label: streamDemo" \
-H "column_separator:," \
http://127.0.0.1:8030/api/dwx_test/detailDemo/_stream_load
```

导入成功后返回的json中会包含Status字段，如果为success即成功。

### 3.4 Routine load


### 3.5 Insert into