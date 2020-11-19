---
title: "Quickstart"
date: 2020-11-18T21:40:26+08:00
draft: true
tags: ["kafka"]
categories: ["kafka"]
---

### 1. 定义：kafka是一个分布式的基于发布/订阅模式的消息队列
### 2. 使用消息队列的好处：解耦、可恢复性、缓冲、消峰、异步通信。
### 3. 消息队列的两种模式
1. 点对点模式：一对一、消费者主动拉取数据、消息收到后消息清除。消息生产者生产消息发送到queue中、然后消费者从queue中取出并且消费消息、消息被消费后、queue中不在存储、所以消息消费者不可能消费到已经消费的消息。queue支持存在多个消费者、但是对于同一个消息而言、只会有一个消费者可以消费。
![](/images/kafka/0001.jpg)
2. 发布/订阅模式：一对多、消费者消费数据之后不会清除消息。消息生产者(发布)将消息发布到topic中、同时有多个消费者(订阅)消费该消息。和点对点不同、发布到topic的消息会被所有订阅者消费
![](/images/kafka/0002.jpg)

### 4. 推模式和拉模式的优缺点
1. 推模式：消息队列主动将消息推送给客户端、客户端处理能力不同、但收到的消息速率一样、可能压块消费者
2. 拉模式：消费者主动拉取消息、长轮询、消耗客户端资源、kafka是基于拉模式

### 5. 架构图  -- 图片来源 尚硅谷
![](/images/kafka/0003.jpg)
1. produer：消息生产者、就是向kafka borkder发消息的客户端
2. consumer：消费者、向kafka broker取消息的客户端
3. consumer group(GC): 消费者组，由多个 consumer 组成。 消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不影响。 所
有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。
4. Broker:一台 kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker可以容纳多个 topic
5. Topic:可以理解为一个队列， 生产者和消费者面向的都是一个 topic；
6. Partition:为了实现扩展性，一个非常大的 topic 可以分布到多个 broker（即服务器）上，一个 topic 可以分为多个 partition，每个 partition 是一个有序的队列
7. Replica:副本为保证集群中的某个节点发生故障时， 该节点上的 partition 数据不丢失，且 kafka 仍然能够继续工作，kafka 提供了副本机制，一个 topic 的每个分区都有若干个副本一个 leader 和若干个 follower
8. Leader:每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是 leader
9. follower： 每个分区多个副本中的“从”，实时从 leader 中同步数据，保持和 leader 数据的同步。 leader 发生故障时，某个 follower 会成为新的 follower

### 6. 快速启动
1. tar tar -xzf kafka_2.13-2.6.0.tgz
2. ln -s kafka_2.13-2.6.0 kafka  软连接、便于以后切换版本
3. 启动zookeeper `cd kafka && bin/zookeeper-server-start.sh -daemon config/zookeeper.properties`
4. 启动kafka `bin/kafka-server-start.sh -daemon config/server.properties`
5. 停止kafka `bin/kafka-server-stop.sh stop`
6. 修改配置文件
	1. cd config && vim server.properties
	2. 配置说明
		```shell
		#broker全局编号、不能重复
		broker.id=0
		#处理网络请求的线程数量
		num.network.threads=3
		#用来处理磁盘 IO 的线程数量
		num.io.threads=8
		#发送套接字的缓冲区大小
		socket.send.buffer.bytes=102400
		#接收套接字的缓冲区大小
		socket.receive.buffer.bytes=102400
		#请求套接字的缓冲区大小
		socket.request.max.bytes=104857600
		#kafka 运行日志存放的路径
		log.dirs=/tmp/kafka-logs
		#topic 在当前 broker 上的分区个数
		num.partitions=1
		#用来恢复和清理 data 下数据的线程数量
		num.recovery.threads.per.data.dir=1
		#segment 文件保留的最长时间，超时将被删除
		log.retention.hours=168
		#配置连接 Zookeeper 集群地址
		zookeeper.connect=localhost:2181,ip2:2181
        ```
7. kafka命令行操作
	1. 查看服务器所有topic:`bin/kafka-topics.sh --bootstrap-server localhost:9092 --list or bin/kafka-topics.sh --zookeeper localhost:2181 --list`
	2. 创建topic
		```shell
			1. bin/kafka-topics.sh --create --topic quickstart-events-second --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1
			#--replication-factor 定义副本数
			#--partitions 定义分区数
			2. bin/kafka-topics.sh --create --topic quickstart-events-third --zookeeper localhost:2181 --replication-factor 1 --partitions 1
        ```
    3. 查看topic详情
	    ```shell
	    	1. bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic quickstart-events
	    	2. bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic quickstart-events
        ```
    4. 删除topic
	    ```shell
	    	1. bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic quickstart-events-second
	    	2. bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic quickstart-events-third
        ```
    5. 发送消息 `bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic quickstart-events`
    6. 消费消息 `bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events --from-beginning`
    7. 修改分区数`bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic quickstart-events --partitions 2`