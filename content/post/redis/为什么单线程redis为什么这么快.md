---
title: "为什么单线程redis为什么这么快"
date: 2020-10-18T09:50:27+08:00
draft: true
tags: ["redis常见问题"]
categories: ["redis"]
---
### 1. 通常所说的redis单线程指
redis的网络IO和键值对读写是由一个线程来完成的。其它功能、比如持久化、异步删除、集群数据同步等则是由额外线程来完成的
### 2. 多线程开销
1. 共享资源并发访问控制
2. 线程切换

### 3. 单线程redis为什么这么快
1. 内存及高效的数据结构
2. 多路复用机制、使其在网络IO操作中能并发处理大量的客户端请求

### 4. 基本io模型与阻塞点：
1. 一个get请求处理、一般需要几个过程、监听客户端请求(bind/listen),和客户端建立链接(accept),从socket中读取数据(recv),解析客户端发送请求(parse),根据请求类型读取键值数据(get),给客户端返回结果(send)

![](/images/goredis/redis-value-data-009.jpg)

上图黑色部分为网络IO处理、灰色部分为键值数据操作

### 5. 非阻塞模式
1. socket网络模型的非阻塞模式设置、主要体现在三个函数调用上。
2. 在socket模型中、不同操作会返回不同的套接字类型。socket方法会返回主动套接字、然后调用listen方法、将主动套接字转化为监听套接字、此时可监听客户端链接请求。最后调用accept()方法接收到达的客户端连接、并返回已连接套接字。
![](/images/goredis/redis-value-data-010.jpg)
针对监听套接字我们可以设置非阻塞模式：当redis调用accept()但一直未有连接请求到达是、redis线程可以返回处理其他操作、而不用一直等待。但是需要注意的是、调用accept()时、已经在监听套接字了。虽然redis线程不用再继续等待、但是总得有机制继续在监听套接字上等待后续链接请求、并在有请求时通知redis。这样才能保证redis线程、既不会像基本IO模型中一直在阻塞点等待、也不会导致redis无法处理实际到达的连接请求或者数据
6. 基于多路复用的高性能I/O模型
	1. Linux中的IO多路复用机制是指：一个线程处理多个IO流、就是我们经常听到的select/epoll机制。简单来说、在redis只运行单线程的情况下、该机制允许内核中、同时存在多个监听套接字和已连接套接字。内核会一直监听这些套接字上的链接请求或者数据请求。一旦有请求到达、就会交给redis线程处理、这就实现了一个redis线程处理多个IO流的效果。
	2. 基于多路复用的redis IO模型、图中的多个FD就是刚才所说的多个套接字。redis网络框架调用epool机制、让内核监听这些套接字。此时redis线程不会阻塞在某一个特定的监听或者以链接的套接字上。也就是说、不会阻塞在某一个特定的客户端请求处理上。
	![](/images/goredis/redis-value-data-011.jpg)
	说明：为了在请求到达时能通知到 Redis 线程，select/epoll 提供了基于事件的回调机制，即针对不同事件的发生，调用相应的处理函数。那么，回调机制是怎么工作的呢？其实，select/epoll 一旦监测到 FD 上有请求到达时，就会触发相应的事件。这些事件会被放进一个事件队列，Redis 单线程对该事件队列不断进行处理。这样一来，Redis 无需一直轮询是否有请求实际发生，这就可以避免造成 CPU 资源浪费。同时，Redis 在对事件队列中的事件进行处理时，会调用相应的处理函数，这就实现了基于事件的回调。因为 Redis 一直在对事件队列进行处理，所以能及时响应客户端请求，提升 Redis 的响应性能。