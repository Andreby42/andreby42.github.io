---
title: Redis集群搭建
date: 2018-10-16 00:20:18
tags: [Nosql]
categories: [Nosql]
---

Redis集群搭建了解下<!--more--->

今天在图灵群中，有人提到了redis集群的搭建，黑白老师说看郭大路吹牛，下班时候我在群里说：好今晚搞下给你看。

其实这个东西很早时候就玩过了，用不用说了，基本上是现在标配了，集群也是很早就搭过了。但是后来基本上都是运维来做，所以后来很少自己动手去做了

* 下载安装

  我们下载最新版的redis4.0.11

  ```
  wget http://download.redis.io/releases/redis-4.0.11.tar.gz
  tar -zxvf redis-4.0.11.tar.gz -c redis-4.0.11-build
  ```

  进行编译安装

  ```
  make prefix=/data/soft/redis-4.0.11 install
  mv redis-4.0.11 redis-4.0.11-6379
  ```

* 进行配置

  将``redis-4.0.11-build``目录下的redis.conf 复制一份到`` redis-4.0.11-6379/bin``目录下

  进行配置

  一些目录需要手动创建比如：dir log

  ```
  ...
  protected-mode no
  ...
  bind 127.0.0.1
  ...
  #这一行为新加
  masterauth nopassword
  requirepass nopassword 
  ...
  port 6379
  ...
  daemonize yes
  ...
  bind 127.0.0.1
  ...
  dir /data/soft/redis-4.0.11/data/
  ...
  pidfile /var/run/redis_6379.pid（pid 6379和port要对应）
  ...
  cluster-enabled yes
  ...
  cluster-config-file nodes6379.conf （6379和port要对应）
  ...
  cluster-node-timeout 15000
  ...
  appendonly yes
  ```

  修改完毕后进行复制出五个实例一共六个实例

  ```
  cp redis-4.0.11-6379/ redis-4.0.11-6380 -r 
  cp redis-4.0.11-6379/ redis-4.0.11-6381 -r 
  cp redis-4.0.11-6379/ redis-4.0.11-6382 -r 
  cp redis-4.0.11-6379/ redis-4.0.11-6383 -r 
  cp redis-4.0.11-6379/ redis-4.0.11-6384 -r 
  ```

  修改上述复制的实例中的redis.conf

  ```
  ...
  port 6379
  ...
  cluster-config-file nodes6379.conf （6379和port要对应）
  ...
  pidfile /var/run/redis_6379.pid（pid 6379和port要对应）
  ```

* 下载ruby

  ```
  apt-get install ruby -y
  apt-get install rubygems -y
  gem install redis
  ```

* 配置集群密码

  ```
  find / -name client.rb
  /usr/lib/ruby/2.3.0/xmlrpc/client.rb
  /var/lib/gems/2.3.0/gems/redis-4.0.2/lib/redis/client.rb
  vim /var/lib/gems/2.3.0/gems/redis-4.0.2/lib/redis/client.rb
  ```

  ```
  ...
   DEFAULTS = {
         :url => lambda { ENV["REDIS_URL"] },
         :scheme => "redis",
         :host => "127.0.0.1",
         :port => 6379,
         :path => nil,
         :timeout => 5.0,
         :password => "nopassword",##注意这里密码设置和主节点设置实例密码设置一致 而且用引号括住
         :db => 0,
         :driver => nil,
         :id => nil,
         :tcp_keepalive => 0,
         :reconnect_attempts => 1,
         :inherit_socket => false
        }
  
  ...
  ```

  如果不在每个实例中配置密码``masterauth nopassword``和``requirepass nopassword``,并且在client.rb中进行配置的话会报错：

  ```
  [ERR] Sorry, can't connect to node 127.0.0.1:6379
  ```

   这种情况的出现是由于 实例设置了密码 但是masterauth与client.rb没有设置密码。

* 编写启动脚本

  ```
  cd /data/soft
  vim startRedis6379.sh
  ```

  ```
   /data/soft/redis-4.0.11-6379/bin/redis-server  /data/soft/redis-4.0.11-6379/bin/redis.conf
  ```

  ```
  chmod 777 startRedis6379.sh
  ```

  ```
  cp startRedis6379.sh startRedis6380.sh
  cp startRedis6379.sh startRedis6381.sh
  cp startRedis6379.sh startRedis6382.sh
  cp startRedis6379.sh startRedis6383.sh
  cp startRedis6379.sh startRedis6384.sh
  ##记得修改sh脚本中的目录名称为各个实例的名称
  ```

* 开启防火墙，及安全组

  不仅需要开启各个实例的端口，而且要开启集群的总线（port+10000），如果不开启的话在构建集群的时候会一直waiting join redis....

* 启动

  ```
  sh startRedis6379.sh
  sh startRedis6380.sh
  sh startRedis6381.sh
  sh startRedis6382.sh
  sh startRedis6383.sh
  sh startRedis6384.sh
  ```

* 进入源码目录创建集群

  ```
  cd /data/soft/redis-4.0.11-build
  /data/soft/redis-4.0.11-build/src/redis-trib.rb create --replicas 1 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384
  ```

* 然后就可以看到集群构建成功的日志了

  ```
  [OK] All nodes agree about slots configuration.
  >>> Check for open slots...
  >>> Check slots coverage...
  [OK] All 16384 slots covered.
  ```

* 使用cli进行链接

  ```
  root@VM-0-17-ubuntu:/data/soft/redis-4.0.11-6379/bin# ./redis-cli -h 127.0.0.1 -p 6379 -c
  127.0.0.1:6379> auth nopassword
  OK
  127.0.0.1:6379> cluster nodes
  015e1ed80d9c7bc5aeaeec8a44052622f463c37c 127.0.0.1:6381@16381 master - 0 1539623481808 3 connected 10923-16383
  1907a9da5d164b601179ec290a39e6d0e3d3d00f 127.0.0.1:6383@16383 slave 015e1ed80d9c7bc5aeaeec8a44052622f463c37c 0 1539623482811 5 connected
  54a81120760e6fc80d29791dae723f6722c833a3 127.0.0.1:6379@16379 myself,master - 0 1539623479000 1 connected 0-5460
  aaa56f4aa0c9256f36729e5c9cad26f5bde38e8a 127.0.0.1:6384@16384 slave 54a81120760e6fc80d29791dae723f6722c833a3 0 1539623482000 6 connected
  43a6919c8409ae1df9b55bc2544d789ecf862365 127.0.0.1:6380@16380 master - 0 1539623480806 2 connected 5461-10922
  1e32a678d297a326e1655905010120ca6105a589 127.0.0.1:6382@16382 slave 43a6919c8409ae1df9b55bc2544d789ecf862365 0 1539623481000 4 connected
  127.0.0.1:6379> set 9 9
  -> Redirected to slot [10106] located at 127.0.0.1:6380
  (error) NOAUTH Authentication required.
  127.0.0.1:6380> nopassword
  (error) ERR unknown command `nopassword`, with args beginning with: 
  127.0.0.1:6380> auth nopassword
  OK
  127.0.0.1:6380> set 9 9
  OK
  127.0.0.1:6380> exit
  
  ```

  在登陆6379获取这个key

  ```
  root@VM-0-17-ubuntu:/data/soft/redis-4.0.11-6379/bin# ./redis-cli -h 127.0.0.1 -p 6379 -c
  127.0.0.1:6379> auth nopassword
  OK
  127.0.0.1:6379> get 9
  -> Redirected to slot [10106] located at 127.0.0.1:6380
  (error) NOAUTH Authentication required.
  127.0.0.1:6380> auth nopassword
  OK
  127.0.0.1:6380> get 9
  "9"
  127.0.0.1:6380> 
  ```

  查看集群信息

  ```
  127.0.0.1:6380> cluster info
  cluster_state:ok
  cluster_slots_assigned:16384
  cluster_slots_ok:16384
  cluster_slots_pfail:0
  cluster_slots_fail:0
  cluster_known_nodes:6
  cluster_size:3
  cluster_current_epoch:6
  cluster_my_epoch:2
  cluster_stats_messages_ping_sent:4003
  cluster_stats_messages_pong_sent:3910
  cluster_stats_messages_meet_sent:5
  cluster_stats_messages_sent:7918
  cluster_stats_messages_ping_received:3910
  cluster_stats_messages_pong_received:4008
  cluster_stats_messages_received:7918
  ```

  **这样集群就搭建成功了**

* 当中遇到的问题

  * bind 问题

    每个实例bind 的ip 必须和redis-trib.rb 那个命令中的 实例 host:ip对应上

  * 集群搭建至少6个实例

  * 当在构建集群时候，如果碰到如下问题：

    ```
    [ERR] Node 127.0.0.1:6380 is not empty. Either the nodealready knows other nodes (check with CLUSTER NODES) or contains some key in database 0.
    ```

    那么这么办：

    * 将该节点下的aof,rdb备份文件删除 ，我的在redis-4.0.11-6380/data目录下
    * 将该节点的redis.conf中的cluster-config-file的文件nodes6380.conf删除掉，我的也在data目录下
    * 如果还是报错，就登陆该节点cli，进行``flushdb``操作
    * 完成上述任何一项，进行重启redis实例，重新构建集群

  

  **先写到这里了，具体的集群原理后面补充。**

  