---
layout:     post
title:      Zookeeper
subtitle:   知识积累
date:       2020-04-05
author:     guchaolong
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
category: Java
tags:
       - Java
---

> Zookeeper 开源的 为分布式应用程序提供协调服务的apache项目

# 一.Zookeeper介绍

#### 1.1 引言

> 1. 注册中心（取代springcloud的Eureka)
> 2. 配置集中管理  (取代springcloud的配置中心)
> 3. 集群管理
> 4. 分布式锁，分布式任务
> 5. 队列管理

#### 1.2 zookeeper介绍

> zookeeper本身是hadoop生态圈中的一个组件，但是由于强大的功能，在java分布式架构中，也频繁用到zookeeper、

> zookeeper就是一个文件系统+监听通知机制



# 二.Zookeeper安装

> docoker-compose.yml

```yml
version: "3.1"
services:
	zk:
  		image: daocloud.io/daocloud/zookeeper:latest\
  		restart: always
  		container_name: zk
  		ports:
  	  		- 2181:2181
```



# 三.Zookeeper架构

#### 3.1 

> 每一个节点都是一个znode
>
> 每一个znode中都可以存储数据
>
> 同一层级中znode名称不能重复

#### 3.2 znode类型

#### 3.3 zookeeper的监听通知机制

> 客户端可以监听zookeeper中的znode
>
> znode改变时，会通知监听当前znode的客户端





# 四.Zookeeper常用命令

> 1 查询

```sh
#查询当前节点下的全部子节点
ls 节点名称
#例子 ls /
```

```sh
#查询当前节点下的数据
get 节点名称
#例子 get /zookeeper
```





> 2 创建节点

```sh
create [-s] [-e] znode名称 znode数据

# -s : sequence，有序节点
# -e ：ephemeral,临时节点
```



```sh
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 1] get /zookeeper 

[zk: localhost:2181(CONNECTED) 2] create /qf xxx
Created /qf
[zk: localhost:2181(CONNECTED) 3] ls /
[qf, zookeeper]
[zk: localhost:2181(CONNECTED) 4] get /qf
xxx
[zk: localhost:2181(CONNECTED) 5] create -s /qf yyy
Created /qf0000000001
[zk: localhost:2181(CONNECTED) 6] ls /
[qf, qf0000000001, zookeeper]
[zk: localhost:2181(CONNECTED) 7] create -s /qf yyy
Created /qf0000000002
[zk: localhost:2181(CONNECTED) 8] ls /
[qf, qf0000000001, qf0000000002, zookeeper]
[zk: localhost:2181(CONNECTED) 9] 

[zk: localhost:2181(CONNECTED) 1] create -s -e /temp xxxxxxxx
Created /temp0000000003
[zk: localhost:2181(CONNECTED) 2] ls /
[qf, qf0000000001, qf0000000002, temp0000000003, zookeeper]
[zk: localhost:2181(CONNECTED) 3] create -s -e /temp xxxxxxxx
Created /temp0000000004
[zk: localhost:2181(CONNECTED) 4] create -s -e /temp xxxxxxxx
Created /temp0000000005
[zk: localhost:2181(CONNECTED) 5] create -s -e /temp xxxxxxxx
Created /temp0000000006
[zk: localhost:2181(CONNECTED) 6] ls /
[qf, qf0000000001, qf0000000002, temp0000000003, temp0000000004, temp0000000005, temp0000000006, zookeeper]
[zk: localhost:2181(CONNECTED) 7]

#以上创建了三个临时节点temp0000000003 temp0000000004 temp0000000005，如果输入quit 断开连接，再重新打开zkCli.sh重开一个客户端，临时节点就没有了

```



> 3 修改节点数据

```sh
set znode名称 新数据
```



```shell
[zk: localhost:2181(CONNECTED) 1] get /qf
xxx
[zk: localhost:2181(CONNECTED) 2] set /qf iiiiiiiiiiiiiiiiiii
[zk: localhost:2181(CONNECTED) 3] get /qf
iiiiiiiiiiiiiiiiiii
[zk: localhost:2181(CONNECTED) 4] 
```



> 4、删除节点数据

```sh
delete znode名称 #删除没有字节的znode
deleteall znode名称 #删除当前节点和全部字节点
```



# 五.Zookeeper集群

#### 5.1 Zookeeper集群架构图
![Image text](https://raw.githubusercontent.com/guchaolong/guchaolong.github.io/master/_posts_img/zk1.png)



#### 5.2 集群中节点的角色

> 1、Leader
>
> Master主节点
>
> 2、Flower （默认的从节点）
>
> 从节点， 参与选举全新的leader 投票
>
> 3、Observer
>
> 从节点，不参与投票 
>
> 4、Looking
>
> 正在找leader节点



#### 5.3 zookeeper投票策略

> 1、每一个zookeeper服务都会被分配一个全局唯一的myid,  myid是一个数字
>
> 2、zookeeper在执行写数据时，每一个节点都有一个自己的FIFO队列。保证写每一个数据的时候，顺序是不会乱的，zookeeper还会给每一个数据分配一个全局唯一的zxid，数据越新zxid就越大

> 选举Leader:
>
> 1. 选举出zxid最大的节点作为Leader
> 2. 在zxid相同的节点中，选举出一个myid最大的节点作为Leader




# 六.Java操作Zookeeper