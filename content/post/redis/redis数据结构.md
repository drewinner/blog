---
title: "Redis数据结构"
date: 2020-10-09T22:23:57+08:00
draft: true
tags: ["redis常见问题"]
categories: ["redis"]
---
### 1. redis为什么这么快
1. 内存
2. 数据结构

### 2. 五种基本数据类型：
1. string,list,hash,set,sorted set
2. redis键值对中值的数据类型、也就是数据的保存形式

### 3. 底层数据结构
1. 简单动态字符串
2. 双向链表
3. 压缩列表
4. 哈希表
5. 跳表
6. 整数数组

### 4. 五种基本数据类型和及底层数据结构对应关系
![](/images/goredis/redis-value-data-001.jpg)

```
1. 字符串底层数据结构
	int 8个字节的长整型
	embstr 小于等于39个字节的字符串
	raw 大于39个字节的字符串
2. list底层数据结构
	ziplist:当列表元素个数小于list-max-ziplist-entries配置（默认512个)
	同时列表中每个元素的值都小于list-max-ziplist-value(默认64字节).
	linkedlist
3. 哈希底层编码
	ziplist:当哈希元素个数小于hash-max-ziplist-entries(默认512个)、
	同时所有值都小于hash-max-ziplist-value配置(默认64字节)
	哈希表：不满足上面条件时为hash表
4. 有序集合
	ziplist:当有序集合元素个数小于zset-max-ziplist-entries(默认128个)、
	同时所有值都小于zset-max-ziplist-value配置(默认64字节)
	skiplist：不满足上面条件时为跳表
5. 无序集合
	整数集合:当集合中元素个数小于set-max-intset-entries(默认512个)时
	hashtable:不满足上面条件时为跳表
```
图中可以看出、string类型只有一种数据结构即简单动态字符串。list、hash，sorted set,set有两种底层数据结构。特点是一个键对应了一个集合的数据

### 5. 问题
1. 这些数据结构都是值的底层实现、键和值本身之间用什么结构组织
	1. 键和值是通过哈希表组织数据的
	2. 一个哈希表其实就是个数组、数组的每个元素称为一个哈希桶、所以我们常说一个哈希表是由多个哈希桶组成
	3. 哈希桶中保存的并不是值本身、而是指向具体值的指针、无论值是字符串还是集合类型
	![](/images/goredis/redis-value-data-002.jpg)
	4. redis变慢的原因之一: 哈希冲突和rehash可能带来的阻塞操作
	5. redis解决哈希冲突：
		1. 链表法  其它语言可能采取开放寻址法具体可以自己查询、带来的问题：链表越来越长、导致查找复杂度增加、所以rehash。
		2. redis rehash方式：默认使用两个hash表：hash表1和hash表2、开始默认使用hash表1、此时的hash表2没有分配空间、随着数据增加开始执行rehash
			1. 给hash分配更大的空间、例如是hash表1的两倍
			2. 把hash表1中的数据重新映射并保存到hash表2中
			3. 释放hash表1的空间
			4. rehash涉及大量数据拷贝，采取了渐进式rehash：redis正常处理客户端请求、每处理一个请求、从hash表1中的第一个位置开始、顺带索引上的entries拷贝到hash表2中、同时redis还有计划任务辅助rehash操作。
2. 为什么集合类型有这么多的底层结构、他们都是怎么组织数据的、都很快吗？
	1. 集合的操作效率：一个集合类型的值、第一步是通过全局哈希表找到对应的hash桶位置、第二步是在集合中进行增删改查操作。
	2. 集合的操作效率和哪些因素有关：底层数据结构（比如hash比链表实现的集合访问效率更高);和这些操作本身的执行特点有关
	3. 底层数据结构分类： 整数数组、双向链表、哈希表、压缩列表和跳表。
	4. 按查找时间复杂度分类
	
	![](/images/goredis/redis-value-data-003.jpg)

	5. 不同操作的复杂度
		1. 字符串操作
		![](/images/goredis/redis-value-data-004.jpg)
		2. list操作
		![](/images/goredis/redis-value-data-005.jpg)
		提示：
			1. lpush+lpop 栈
			2. lpush+rpop 队列
			3. lpush+ltrim 有限集合
			4. lpush+brpop 消息队列
3. 什么是简单动态字符串、和常用的字符串是一回事吗？