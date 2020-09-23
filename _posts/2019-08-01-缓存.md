---
layout:     post
title:      缓存中间件
subtitle:   
date:       2019-08-01
author:     guchaolong
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
category: 缓存
tags:
       - 缓存
---
>缓存 Redis、MongoDB


# Redis

## Redis安装

## Redis集群搭建
> redis集群至少需要三个master节点，并且给每个master加一个slave节点,一共6个实例

```java

mkdir -p /usr/local/redis-cluster
cd redis-cluster/
mkdir 8001
mkdir 8002
mkdir 8003
mkdir 8004
mkdir 8005
mkdir 8006

cp /usr/local/redis/etc/redis.conf /usr/local/redis-cluster/8001
修改为
daemonize yes
port 8001
bind 192.xxx.xxx.xx (本机)
dir /usr/local/redis-cluster/8001/
cluster-enabled yes
cluster-config-file nodes-8001.conf
cluster-node-timeout 5000
appendonly yes

修改后的redis.conf分表copy到各文件夹下，8001改为对应的800x
vim redis.conf  
：%s/8001/8002/g
:wq






```



## Redis命令
 ```java

PING

Config相关
config get *
config get port
config set k v

String相关
set k v1
get k
getset k v                  设新返旧
mget k k2
mset k1 v1 k2 v2
setbit k 10 0
getbit k 10
setrange k 2 v
getrange k 0 33
setnx k v
msetnx k1 v1 k2 v2          所有key都不存在时设置
setex k 5 v				    秒
psetex k 5000 v             毫秒
incr k
incrby k 342		
incrbyfloat k 23.8
decr k
decrby k 20
append k v 

	
Hash相关
hmset k f1 v1 f2 v2
hget k f2


List相关
lpush k v1 v2 v3 v4
rpush k v5 v6
range k 0 2000


Set相关
sadd k v1 v2 v3 v4
smembers k


ZSet相关
zadd k 0 v1
zadd k 3 v2
zrange k 0 10
zrangebyscout k 0 10


通用
keys *
del k
dump k
exists k
expire k 10
persist k
pttl k
ttl k
rename k nk
type k

```