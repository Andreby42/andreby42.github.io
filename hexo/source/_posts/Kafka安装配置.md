---
title: Kafka安装配置
date: 2018-09-19 20:53:53
tags: [Kafka]
categories: [Kafka]
---

记录下Kafka<!--more-->

最近可能要用到canal,看了下版本更新了，官方出了个和kafka集成的服务端，那么记录下kafka单机安装配置，后续会继续写关于kafka的一些东西

### 环境概述

* linux Ubuntu16.04
* jdk1.8
* zookeeper 3.4.12
* git 2.7.4
* sbt 最新版

### Kafka安装

* 下载

  * 下载安装kafka

    ```
    wget http://mirrors.hust.edu.cn/apache/kafka/2.0.0/kafka_2.11-2.0.0.tgz
    ```

    ```
    tar -zxvf kafka_2.11-2.0.0.tgz -C kafka2.11
    ```

  * 修改配置文件

    ```
    ##解开下面的注释，并修改ip
    # The address the socket server listens on. It will get the value returned from 
    # java.net.InetAddress.getCanonicalHostName() if not configured.
    #   FORMAT:
    #     listeners = listener_name://host_name:port
    #   EXAMPLE:
    #     listeners = PLAINTEXT://your.host.name:9092
    listeners=PLAINTEXT://127.0.0.1:9092
    ##解开下面的注释，并修改ip，保证生产者和消费者能通信，
    ##hostname、port 都会广播给 producer、consumer。如果你没有配置了这个属性的话，则使用 ##listeners 的值，如果 listeners 的值也没有配置的话，则使用##java.net.InetAddress.getCanonicalHostName() 返回值 (这里也就是返回 localhost 了)。
    advertised.listeners=PLAINTEXT://127.0.0.1:9092
    ##修改zookeeper地址
    zookeeper.connect=127.0.0.1:2181
    ```

    server.properties这个关键文件中的一些其他配置说明：

    ```
    ##每一个 broker 在集群中的唯一表示，要求是正数。当该服务器的 IP 地址发生改变时，broker.id 没有##变化，则不会影响 consumers 的消息情况
    broker.id=0
    ## broker 处理消息的最大线程数，一般情况下数量为 cpu 核数
    num.network.threads=2
    ## broker 处理磁盘 IO 的线程数，数值为 cpu 核数 2 倍
    num.io.threads=8
    ## socket 的发送缓冲区，socket 的调优参数 SO_SNDBUFF
    socket.send.buffer.bytes=1048576
    ## socket 的接受缓冲区，socket 的调优参数 SO_RCVBUFF
    socket.receive.buffer.bytes=1048576
    ##socket 请求的最大数值，防止 serverOOM，message.max.bytes 必然要小于 ##socket.request.max.bytes，会被 topic 创建时的指定参数覆盖
    
    socket.request.max.bytes=104857600
    
    ##kafka 数据的存放地址，多个地址的话用逗号分割, 多个目录分布在不同磁盘上可以提高读写性能  ##/data/kafka-logs-1，/data/kafka-logs-2
    
    log.dirs=/tmp/kafka-logs
    
    ##每个 topic 的分区个数，若是在 topic 创建时候没有指定的话会被 topic 创建时的指定参数覆盖
    
    num.partitions=2
    
    ##数据文件保留多长时间， 存储的最大时间超过这个时间会根据 log.cleanup.policy 设置数据清除策略
    
    ##log.retention.bytes 和 log.retention.minutes 或 log.retention.hours 任意一个达到要求，都##会执行删除有两种删除数据文件方式：按照文件大小删除：log.retention.bytes，按照 不同时间粒度删##除：分别为分钟，小时
    
    log.retention.hours=168
     
    ##topic 的分区是以一堆 segment 文件存储的，这个控制每个 segment 的大小，会被 topic 创建时的指##定参数覆盖
    
    log.segment.bytes=536870912
    ##文件大小检查的周期时间，是否处罚 log.cleanup.policy 中设置的策略
    log.retention.check.interval.ms=60000
    ##是否开启日志清理
    log.cleaner.enable=false
     
    ##zookeeper 集群的地址，可以是多个，多个之间用逗号分割 ##hostname1:port1,hostname2:port2,hostname3:port3
     
    zookeeper.connect=127.0.0.1:2181
    
    ##ZooKeeper 的连接超时时间
    zookeeper.connection.timeout.ms=1000000
    ```

    就先记录这么多。

  * 启动

    ```
    cd /data/soft/kafka2.11
    
    bin/kafka-server-start.sh config/server.properties &
    ```

  * 联通测试

    创建一个topic，并进行msg发送，注意这里的broker-list要和server.properties中一致

    运行生产者脚本

    ```
    cd /data/soft/kafka2.11
    
    bin/kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic test
    ##出现了这个
    root@VM-0-17-ubuntu:/data/soft/kafka2.11# bin/kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic test
    >hello kafka
    ```

    新开一个窗口，运行消费者脚本，注意这里的bootstrap-server要和server.properties中一致

    ```
    bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic test --from-beginning
    ```

    **PS：**如果 使用`` bin/kafka-console-consumer.sh --zookeeper 127.0.0.1:2181 --topic test --from-beginning  ``会提示`` Using the ConsoleConsumer with old consumer is deprecated and will be removed in a future major release. Consider using the new consumer by passing [bootstrap-server] instead of [zookeeper].``

    两者的区别在于：

    Zookeeper 是直接连接的 zk，而 bootstrap-server 则是连接的 broker 的 ip. 因为在 0.9 之后，kafka 使用了新的 consumer API 进行消费。旧的 API 会逐步淘汰

###  Kafka-Manager搭建

   服务端搭建完了，要有个地方可以可以监控，类似zkui，dubboadmin，但是官方的好像没有，找了找，大概有 

​    webconsole，manager，决定用manager,但是manager没有权限管理。

 * 下载kafka-manager的源码。

   ```
   git clone https://github.com/yahoo/kafka-manager
   ```

   或者(比较慢)

   ```
   mkdir kafka-manager
   
   wget https://github.com/yahoo/kafka-manager/archive/1.3.3.18.tar.gz
   
   tar -zxvf 1.3.3.18.tar.gz -C kafka-manager
   
   ```

   或者手动下载，再上传到服务器

*  修改配置

   ```
   cd /data/soft/kafka-manager/conf
   
   vim application.conf
   ```

   修改以下

   ```
    ##多个zookeeper 逗号隔开
    kafka-manager.zkhosts="127.0.0.1:2181"
   ```

*  下载安装sbt

   默认的kafka-manager里面有一个sbt文件可以用

   ```
   cd /data/soft/kafka-manager
   ./sbt clean dist
   ##比较慢
   Downloading sbt launcher for 0.13.9:
     From  http://repo.typesafe.com/typesafe/ivy-releases/org.scala-sbt/sbt-launch/0.13.9/sbt-launch.jar
       To  /root/.sbt/launchers/0.13.9/sbt-launch.jar
   Getting org.scala-sbt sbt 0.13.9 ...
   ```

   也可以自己上``https://www.scala-sbt.org/download.html``下载sbt，就是scala build tool吧，然后再``/etc/profile``中配置好环境变量就行。（这种比较快）

   执行完成后在/data/soft/kafka-manager/target中会生成一个zip包，

   ```
   mv /data/soft/kafka-manager /data/soft/kafka-manager-build
   
   cd /data/soft/kafka-manager-build/target
   
   cp  kafka-manager-1.3.3.18.zip /data/soft/
   
   unzip kafka-manager-1.3.3.18.zip
   ```

*  启动

   ```
   nohup bin/kafka-manager -Dconfig.file=conf/application.conf -Dhttp.port 9093 &
   ```

*  访问

   ```
   http://127.0.0.1:9093
   ```

   直接就进去界面了，首次访问需要进行设置kafka的cluster配置





