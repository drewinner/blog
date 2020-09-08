---
title: "Pprof"
date: 2020-09-01T10:59:46+08:00
draft: true
tags: ["golang优化"]
categories: ["golang"]
---

### 1. 前言

公司项目运行一段时间后发现虚拟内存一直不释放，并且随着运行时间的增加，虚拟内存也随之增加。使用pprof进行排查，最终解决，总结了部分使用方法，方便查阅

### 2. 开启pprof服务
```go
_ "net/http/pprof"

go func() {
	logger.Error(fmt.Sprintf("%s", http.ListenAndServe(":8889", nil)))
}()
```

### 3. 方式一：浏览器查看
对当先系统概况有个了解
```go
http://localhost:8889/debug/pprof/
```
![](/images/golang/pprof01.jpg)

|类型	|描述	|备注|
|----|----|----|
|allocs|内存分配情况的采样信息|A sampling of all past memory allocations
|blocks|	阻塞操作情况的采样信息	| 
|cmdline	|显示程序启动命令及参数	|显示启动命令
|goroutine	|当前所有协程的堆栈信息	|
|heap	|堆上内存使用情况的采样信息	|
|mutex	|锁争用情况的采样信息|锁情况
|profile	|CPU 占用情况的采样信息	| 
|threadcreate	|系统线程创建情况的采样信息	|
|trace	|程序运行跟踪信息	|[《Go trace》](https://mp.weixin.qq.com/s/I9xSMxy32cALSNQAN8wlnQ)

### 4. 方式二：交互式命令
1. 查看堆栈调用信息
go tool pprof http://localhost:8889/debug/pprof/heap
```go
进入交互模式
>top
Showing nodes accounting for 6385.52kB, 100% of 6385.52kB total
Showing top 10 nodes out of 37
      flat  flat%   sum%        cum   cum%
 3232.96kB 50.63% 50.63%  3232.96kB 50.63%  github.com/samuel/go-zookeeper/zk.(*Conn).recvLoop
 1616.48kB 25.31% 75.94%  1616.48kB 25.31%  github.com/samuel/go-zookeeper/zk.Connect


>list recvLoop # recvLoop是看到占用cpu较高的函数名称     
3.16MB     3.16MB (flat, cum) 50.63% of Total
         .          .    832:func (c *Conn) recvLoop(conn net.Conn) error {
         .          .    833:	sz := bufferSize
         .          .    834:	if c.maxBufferSize > 0 && sz > c.maxBufferSize {
         .          .    835:		sz = c.maxBufferSize
         .          .    836:	}
    3.16MB     3.16MB    837:	buf := make([]byte, sz)
         .          .    838:	for {
         .          .    839:		// package length
         .          .    840:		if err := conn.SetReadDeadline(time.Now().Add(c.recvTimeout)); err != nil {
         .          .    841:			c.logger.Printf("failed to set connection deadline: %v", err)
         .          .    842:		}

```
2. 查看 30 秒内的 CPU 信息
go tool pprof http://localhost:8889/debug/pprof/profile?seconds=30
3. 查看 goroutine 阻塞
go tool pprof http://localhost:8889/debug/pprof/block
4. 收集 5 秒内的执行路径
go tool pprof http://localhost:8889/debug/pprof/trace?seconds=5
5. 争用互斥持有者的堆栈跟踪
go tool pprof http://localhost:8889/debug/pprof/mutex

### 5. 方式三：web UI界面
1. 执行使用方式二相关命令，会生成对应的.pb.gz 文件
2. 启动服务：go tool pprof -http=${ip}:8080 xxx.pb.gz
3. 浏览器中访问：ip:8080

### 6. 参考文档
1. https://blog.wolfogre.com/posts/go-ppof-practice/
2. https://blog.csdn.net/sunxianghuang/article/details/93869683