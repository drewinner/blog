---
title: "go-redis源码分析(一) 项目概览"
date: 2020-08-18T11:14:44+08:00
draft: true
tags: ["go-redis"]
categories: ["go-redis"]
---
# 1. 测试用例地址
```
github.com/drewinner/go-redis-demo
```

# 2. 项目结构
![](/images/goredis/go-redis-001.jpg)

结构说明
1. redisclusterConn.go 创建连接
2. opredis.go 操作redis方法封装
3. rediscluster_test.go 测试用例
