---
layout:     post
title:      cdh can't restart yarn service after a long time running
subtitle:   yarn problem
date:       2025-03-24
author:     加菲猫
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - BigData
    - Yarn
---

## 问题背景

通过cloudera manager重启yarn角色，发现集群中部分node manger可以启动，有些节点无法启动。后来发现无法启动的节点比可以启动一批节点运行时间早一年左右。

## 分析逻辑

1. 分析是否端口被占用
2. 是否yarn log的存储路径磁盘空间不够
3. 检查/var/log/cloudera-yarn目录下日志文件是否输出异常
4. 检查cloudera-scm-agent的日志是否正常
5. 检查/var/run/process/yarn/***-log 运行配置下日志文件是否正常

## 分析结果

1. 对比发现，yarn的进程在，但是只监听到7337端口，没有监听到8041,8042等端口
2. yarn日志文件存储路径空间都足够
3. 日志无明显异常，报无法连接本地的8042,8041端口，虽然nodemanager进程一直在，但是进程一直服务监听8041rpc端口，以及8042web应用端口

## 处理方式

1. 在cdh yarn 配置页面将NodeManager的堆栈大小由1GB调整为10GB，再单独重启一台服务器节点，发现yarn的日志有变化，不停输出删除yarn 缓存log日志
2. 持续一段时间后，8041、8042端口启动正常

## 分析原因

1. nodemanger进程虽然在，但是在启动几个监听的服务进程时，需要同步加载**部分日志**，在运行久后可能文件较大，会超过堆栈大小，但是日志中只报了gc相关问题

## 经验总结

1. 后续需要关注堆栈设置大小
2. 要关注日志文件保留的时间以及大小，避免出现重启服务配置不匹配问题