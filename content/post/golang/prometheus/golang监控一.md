---
title: "Golang监控一"
date: 2020-11-08T16:54:26+08:00
draft: true
tags: ["prometheus"]
categories: ["prometheus"]
---
1. 使用prometheus生成服务器的指标时、有两个典型的方法
	1. 内嵌地运行在一个服务里并在 HTTP 服务器上暴露一个 /metrics 端点
	2. 或者创建一个独立运行的进程，建立一个导出器。
	3. [向一个基于 worker 的 Go 服务添加指标](https://github.com/scotwells/prometheus-by-example/tree/master/job-processor " 向一个基于 worker 的 Go 服务添加指标")
2. 开始使用：Prometheus 程序库 提供了一个用 Golang 写成的健壮的插桩库，可以用来注册，收集和暴露服务的指标。在讲述如何在应用程序中暴露指标前，让我们先来探究一下 Prometheus 库提供的各种指标类型。

3. 指标类型：Prometheus 客户端公开了在暴露服务指标时能够运用的四种指标类型。查看 [Prometheus的文档](https://prometheus.io/docs/concepts/metric_types/ "Prometheus的文档") 以获得关于各种指标类型的深入信息。
	1. Counter（计数器）是一个累计的指标，代表一个单调递增的计数器，它的值只会增加或在重启时重置为零。例如，你可以使用 counter 来代表服务过的请求数，完成的任务数，或者错误的次数
	2. Gauge（计量器）代表一个数值类型的指标，它的值可以增或减。gauge 通常用于一些度量的值例如温度或是当前内存使用，也可以用于一些可以增减的“计数”，如正在运行的 Goroutine 个数
	3. Histogram（分布图）对观测值（类似请求延迟或回复包大小）进行采样，并用一些可配置的桶来计数。它也会给出一个所有观测值的总和
	4. Summary（摘要）跟 histogram 类似，summary 也对观测值（类似请求延迟或回复包大小）进行采样。同时它会给出一个总数以及所有观测值的总和，它在一个滑动的时间窗口上计算可配置的分位数

4. prometheus http服务器
在你的服务中集成 prometheus 的第一步就是初始化一个HTTP服务器用来提供 Prometheus的指标。这个服务器应该监听一个只在你的基础设施内可用的内部端口；通常是在 9xxx 范围内。Prometheus团队维护一个 [默认端口分配](https://github.com/prometheus/prometheus/wiki/Default-port-allocations "默认端口分配") 的列表，当你选择端口时可以参考。
	```go
	// create a new mux server
	server := http.NewServeMux()
	// register a new handler for the /metrics endpoint
	server.Handle("/metrics", promhttp.Handler())
	// start an http server using the mux server
	http.ListenAndServe(":9001", server)
    ```