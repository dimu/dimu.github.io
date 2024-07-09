---
layout:     post
title:      routine load data from kafka
subtitle:   starrocks
date:       2023-03-15
author:     加菲猫
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - BigData
    - OLAP
    - Real DataWarehouse
    - Starrocks
    - data load
---

## 应用场景

用于数据采集：直接创建routine load导入任务，该导入任务会常驻内存，持续不断消费Kafka消息中间件数据，
并且支持exactly-once语义，保证数据不丢不重。

routine load直接导入与ClickHouse支持Kafka集成一样，都能够直接提交任务给数据库引擎进行数据采集，
便于通过SQL语句实现数据采集。

## 支持数据格式

目前只支持CSV与JSON两种数据格式

### CSV

支持自定义列分隔符，分隔符UTF-8编码，最长字节不超过50个；**空字符**使用\\N表示

### JSON

是否支持JSON嵌套，支持JSON Path语义否

## 创建JSON导入

### 创建表

创建一个聚合表

```
CREATE TABLE `customer_buy_aggregate` ( 
    `commodity_id` varchar(26) NULL COMMENT "品类ID", 
    `customer_name` varchar(26) NULL COMMENT "顾客姓名", 
    `country` varchar(26) NULL COMMENT "顾客国籍", 
    `pay_time` bigint(20) NULL COMMENT "支付时间", 
    `pay_dt` date NULL COMMENT "支付日期", 
    `price`double SUM NULL COMMENT "支付金额"
)
ENGINE=OLAP
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`,`pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`) BUCKETS 5
; 
```

执行上面创建语句由于BE只有一个节点，会生成错误
```
ERROR 1064 (HY000): Failed to find enough host in all backends. need: 3, Current alive backend is [11001]
```

因为默认的副本并发为3个节点，所以需要手动指定**"replication_num" = "1"**，

创建一个明细表
```
CREATE TABLE `customer_buy_info` ( 
    `commodity_id` varchar(26) NULL COMMENT "品类ID", 
    `customer_name` varchar(26) NULL COMMENT "顾客姓名", 
    `country` varchar(26) NULL COMMENT "顾客国籍", 
    `pay_time` bigint(20) NULL COMMENT "支付时间", 
    `price`double NULL COMMENT "支付金额"
)
ENGINE=OLAP
DUPLICATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`) 
DISTRIBUTED BY HASH(`commodity_id`) BUCKETS 5
PROPERTIES (
"replication_num" = "1"
); 
```

### 创建导入任务

配置写入明细表的routine load

```
CREATE ROUTINE LOAD dwx_test.load_detail ON customer_buy_info
COLUMNS(commodity_id, customer_name, country, pay_time, price)
PROPERTIES
(
    "desired_concurrent_number"="5",
    "format" ="json",
    "jsonpaths" ="[\"$.commodity_id\",\"$.customer_name\",\"$.country\",\"$.pay_time\",\"$.price\"]"
 )
FROM KAFKA
(
"kafka_broker_list"="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>,
    "kafka_topic" = "testA",
    "property.kafka_default_offsets" = "OFFSET_END"
);
```
其中format格式json，通过jsonpath指定映射关系

配置写入聚合表的routine load, 通过from_unixtime从字段pay_time中抽取数据
```
CREATE ROUTINE LOAD dwx_test.load_aggregate ON customer_buy_aggregate
COLUMNS(commodity_id, customer_name, country, pay_time, price, pay_dt=from_unixtime(pay_time, '%Y%m%d'))
PROPERTIES
(
    "desired_concurrent_number"="5",
    "format" ="json",
    "jsonpaths" ="[\"$.commodity_id\",\"$.customer_name\",\"$.country\",\"$.pay_time\",\"$.price\"]"
 )
FROM KAFKA
(
"kafka_broker_list"="172.19.3.200:9092,172.19.3.201:9092,172.19.3.202:9092",
    "kafka_topic" = "testA",
    "property.kafka_default_offsets" = "OFFSET_END"
);
```

### 往Kafka写入JSON数据
```
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```


## 配置参数

### 任务调优

## 查看导入任务 

![load routine结果](https://dimu.github.io/img/summary/database/starrocks/show_load_routine_kafka.png "查看load routine结果")

## 停止导入任务

## 采坑集锦

### 当load的topic数据格式不对，无法消费时，任务一直重复消费，没有