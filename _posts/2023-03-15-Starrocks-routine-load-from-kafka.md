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

## 创建导入

## 配置参数

### 任务调优

## 查看导入任务 

## 停止导入任务

