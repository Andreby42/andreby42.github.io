---
title: zookeeper模拟集群搭建
date: 2019-09-14 10:56:37
tags: [Linux]
---

zookeeper集群搭建<!--more-->

### 下载 安装

`wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz`

` tar -zxvf zookeeper-3.4.12.tar.gz`

```
vim /etc/profile
export ZOOKEEPER_HOME=/data/soft/zookeeper-3.4.12
export PATH=.:$HADOOP_HOME/bin:$ZOOKEEPER_HOME/bin:$JAVA_HOME/bin:$PATH

#修改配置文件
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/data/soft/zk/data
dataLogDir=/data/soft/zk/dataLog 
# the port at which the clients will connect
clientPort=2181

server.1=localhost:2887:3887
server.2=localhost:2888:3888
server.3=localhost:2889:3889
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

配置文件介绍

`server.A=B:C:D`

A表示是几号服务器 (myid文件要在data目录下创建并在文件中写入对应的A)

B表示该服务器的IP地址

C表示的是这个服务器与集群中leader服务器交换信息的端口

D表示leader服务器如果挂了，那么follower服务器进行选举的端口

选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。
如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，
所以要给它们分配不同的端口号。

* tickTime：zookeeper中使用的基本时间单位, 毫秒值。

* initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个 tickTime 时间间隔数。这里设置为5表名最长容忍时间为 5 * 2000 = 10 秒。
* syncLimit：这个配置标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2 * 2000 = 4 秒。
* dataDir 和 dataLogDir 看配置就知道干吗的了，不用解释。
* clientPort：监听client连接的端口号，这里说的client就是连接到Zookeeper的代码程序。
* server.{myid}={ip}:{leader服务器交换信息的端口}:{当leader服务器挂了后, 选举leader的端口}
* maxClientCnxns：对于一个客户端的连接数限制，默认是60，这在大部分时候是足够了。但是在我们实际使用中发现，在测试环境经常超过这个数，经过调查发现有的团队将几十个应用全部部署到一台机器上，以方便测试，于是这个数字就超过了。
  