---
title: "一、快速入门"
date: 2020-10-24T07:23:20+08:00
draft: true
tags: ["docker"]
categories: ["docker"]
---
### 1. docker作用
各种软件打包成镜像、分发、独立运行

### 2. 卸载旧版docker

```shell
yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-engine
```
### 3. 安装：
1. 操作系统要求：>= centos 7
2. 安装yum-config-manager 工具 `yum -y install yum-utils`
3. 添加yum源 
	```shell
	yum-config-manager \
	--add-repo \
	https://download.docker.com/linux/centos/docker-ce.repo yum -y install yum-utils
    ```
    
### 4. 安装最新版本 
`yum install docker-ce docker-ce-cli containerd.io`

### 5. 安装指定版本
1. 列出所有版本：`yum list docker-ce --showduplicates | sort -r`
2. 安装：
	```shell
	yum install \
	docker-ce-${VERSION_STRING} \
	docker-ce-cli-${VERSION_STRING} \
	containerd.io
    ```

### 4. 启动docker
`systemctl start docker`

### 5. 查看是否启动docker进程
`ps -ef|grep docker`

### 6. 运行hello-world
1. docker run hello-world 检查是否安装成功、如果成功会输出Hello from Docker!并简要说明了执行流程
2. docker client 与docker daemon 建立连接
3. docker daemon 从Docker Hub拉取hello-world镜像
4. docker daemon 根据此镜像创建了新的用来执行输出字符串的容器
5. docker daemon 以输出流的形式输出给client、打印到终端

### 7. chroot
可以改变某个进程的根目录、使这个程序不能访问目录之外的其它目录。chroot并没有在网络层面上进行隔离chroot并不能完全保证系统安全，在很多层面上chroot并没有进行完全隔离

### 8. docker利用linux的三大机制：
1. Namespace：是linux内核功能、主要用来对资源隔离、使得容器中的进程都可以在单独的命名空间中运行、并且可以访问当前容器命名空间的资源、Namespace可以隔离进程ID、主机名、用户ID、文件名、网络访问和进程间通信等相关资源。
2. Cgroups ： Cgroups 是一种 Linux 内核功能，可以限制和隔离进程的资源使用情况（CPU、内存、磁盘 I/O、网络等）。在容器的实现中，Cgroups 通常用来限制容器的 CPU 和内存等资源的使用
3. 联合文件系统：用于镜像构建和容器运行环境。

### 9. docker主要的五种命名空间
1. pid namespace : 用于隔离进程
2. net namespace:隔离网络接口、在虚拟的net namespace内、用户可以拥有自己独立的IP、路由、端口等。
3. mnt namespace:文件系统挂载点隔离。
4. ipc namesapce:信号量、消息队列和共享内存的隔离。
5. uts namespace：主机名和域名的隔离。
### 10. 三大概念
镜像、容器、仓库

### 11. 镜像
是一个只读的文件和文件夹的组合。包含了容器运行时所需的所有基础文件和配置信息、是容器启动的基础、镜像是Docker容器启动的先决条件。
### 12. 如何使用镜像
1. 自己创建镜像。通常一个镜像是基于一个基础镜像构建的、可以在基础镜像上添加一些用户自定义的内容。
2. 拉取别人制作好的镜像。
### 13. 容器

容器是镜像运行的实体。镜像是静态的只读文件、而容器带有运行时需要的可写文件层。并且容器中的进程属于运行状态。即容器运行着真正的应用进程。
### 14. 容器五种状态

初建、运行、停止、暂停、删除
### 15. 容器与主机上进程的本质区别
在容器内部无法看到主机上的进程、环境变量、网络等信息。
### 16. 仓库：存储和分发docker镜像。

### 17. docker架构图
![](/images/docker/0001.jpeg)
1. 客户端是一种泛称。其中docker命令是docker用户与docker服务端交互的主要方式。除了使用docker命令行的方式、还可以直接请求REST API的方式与docker服务端交互、甚至可以使用各种语言的SDK与docker服务端交互
2. 服务端：是所有后台服务的统称。其中dockerd是一个非常重要的后台管理进程。它负责响应和处理来自docker客户端的请求、然后将客户端请求转化为docker的具体操作。
3. dockerd通过grpc与containerd进行通信
### 18. docker重要组件
```shell
-rwxr-xr-x. 1 root root    53221576 Sep  9 14:33 containerd
-rwxr-xr-x. 1 root root     7168000 Sep  9 14:33 containerd-shim
-rwxr-xr-x. 1 root root    26958888 Sep  9 14:33 ctr
-rwxr-xr-x. 1 root root    84977456 Sep 16 13:04 docker
-rwxr-xr-x. 1 root root   102131240 Sep 16 13:03 dockerd
-rwxr-xr-x. 1 root root      849104 Sep 16 13:03 docker-init
-rwxr-xr-x. 1 root root     3741600 Sep 16 13:02 docker-proxy
-rwxr-xr-x. 1 root root    12893816 Sep  9 14:33 runc
```

重点：runc 和containerd
1. runc 是一个真正用来运行容器的轻量级工具、是真正用来运行容器的。
2. containerd 通过containerd-shim启动并管理runc、可以说containerd真正管理了容器的生命周期。

### 19. docker各组件之间的关系
1. 启动busybox容器：docker run -d busybox sleep 3600
2. 查看dockerd的PID:ps aux |grep containerd `root      4728  0.0  0.7 108712  7156`
3. pstree -a -l -A 4728 如果没有pstree centos安装：yum -y install psmisc
![](/images/docker/0002.jpeg)
