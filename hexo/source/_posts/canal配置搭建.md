---
title: canal配置搭建
date: 2018-07-06 00:08:43
tags: [中间件]
categories: [中间件]
---

Canal搭建配置及代码客户端了解下<!--more-->开始想写的时候，canal的版本还是1.0.2x,现在已经1.1.0了，所以要做什么事情要尽快。

### 环境概述：

* linux系统版本：Ubuntu16.04 
* java版本：1.8.0_171
* zk版本: 3.4.12
* mysql版本：5.7.22 (**这个就不安装了**)
* canal版本：1.1.0
* kafka版本：2.11
* maven版本：3.5.4
* git 版本: 2.7.4

### 安装git

* 使用apt-get安装

  ```
  apt-get update -y
  apt-get install -y git
  ```

### 安装maven

* 下载

  ```
  wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
  tar -zxvf apache-maven-3.5.4-bin.tar.gz -C maven
  ```

* 配置环境变量

  ```
  vim /etc/profile
  ```

  ```
  JAVA_HOME=/data/soft/jdk1.8.0_171
  JAVA_BIN=/data/soft/jdk1.8.0_171/bin
  PATH=$JAVA_HOME/bin:$PATH:$JSTORM_HOME/bin:$ZOOKEEPER_HOME/bin
  ZOOKEEPER_HOME=/data/soft/zookeeper-3.4.12
  CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$ZOOKEEPER_HOME/lib
  JSTORM_HOME=/data/soft/jstorm-2.2.1
  ##MAVEN start
  MAVEN_HOME=/data/soft/maven
  export MAVEN_HOME
  export PATH=${PATH}:${MAVEN_HOME}/bin
  ##MAVEN end
  export ZOOKEEPER_HOME
  export JAVA_HOME
  export PATH
  export CLASSPATH
  export JSTORM_HOME
  ```

  上面是我的环境变量配置文件,修改完后进行source

  ```
  source /etc/profile
  ```

  ```
  mvn -version
  
  Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-18T02:33:14+08:00)
  Maven home: /data/soft/maven
  Java version: 1.8.0_171, vendor: Oracle Corporation, runtime: /data/soft/jdk1.8.0_171/jre
  Default locale: en_US, platform encoding: UTF-8
  OS name: "linux", version: "4.4.0-127-generic", arch: "amd64", family: "unix"
  ```

  安装成功

### 安装zookeeper

* 下载,安装zookeeper

  ```
  wget http://archive.apache.org/dist/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz
  tar -zxvf zookeeper-3.4.12 -C zookeeper
  cd /data/soft/zookeeper/conf
  cp  zoo_sample.cfg   zoo_sample.cfg.bak
  mv  zoo_sample.cfg zoo.cfg
  vim zoo.cfg
  ```

  ```
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
  ##修改这里，如果没有就添加这行
  dataDir=/data/soft/zookeeper/data
  # the port at which the clients will connect
  clientPort=2181
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
  ##添加日志目录
  dataLogDir=/data/soft/zookeeper-3.4.5/logs
  ```

* 启动 zookeeper

  ```
  cd /data/soft/zookeeper
  ./bin/zkServer.sh start ./conf/zoo.cfg
  ```

* 安装 zkui

  * 下载

    ```
    git clone https://github.com/DeemOpen/zkui.git
    ```

  * 修改配置文件

    ```
    cd /data/soft/zkui
    vim config.cfg
    ```

    将下面的进行修改

    ```
    serverPort=9090     #指定端口
    zkServer=192.168.1.110:2181
    sessionTimeout=300000
    ```

  * 进行maven编译生成war包

    ```
    cd /data/soft/zkui
    mvn clean install
    ```

  * 启动

    ```
    nohup java -jar target/zkui-2.0-SNAPSHOT-jar-with-dependencies.jar & 
    ```

  * 访问

    默认密码为 admin/manager appconfig/appconfig ,在cfg文件中可以配置，appconfig为只读权限中户

    ```
    http://127.0.0.1:9090/
    ```

### Mysql配置

 * 配置开启binlog

   本机使用apt-get安装的mysql

   列出所有的 my.cnf 文件 

   ```
   locate my.cnf
   
   /etc/alternatives/my.cnf
   /etc/mysql/my.cnf
   /etc/mysql/my.cnf.fallback
   /var/lib/docker/aufs/diff/46b0e64a8cd2c560c66b86b89918ef4ca60c761b87ba166b52b32d1f94191d09/etc/alternatives/my.cnf
   /var/lib/docker/aufs/diff/46b0e64a8cd2c560c66b86b89918ef4ca60c761b87ba166b52b32d1f94191d09/etc/mysql/my.cnf
   /var/lib/docker/aufs/diff/46b0e64a8cd2c560c66b86b89918ef4ca60c761b87ba166b52b32d1f94191d09/etc/mysql/my.cnf.fallback
   /var/lib/docker/aufs/diff/46b0e64a8cd2c560c66b86b89918ef4ca60c761b87ba166b52b32d1f94191d09/var/lib/dpkg/alternatives/my.cnf
   /var/lib/docker/aufs/diff/d07cc37ba4daa1bba8e1cefb641912781ad304503ce4c3818d3cbf4c17b0587e/etc/mysql/my.cnf
   /var/lib/dpkg/alternatives/my.cnf
   ```

   查看是否使用了指定目录的 my.cnf

   ```
   ps aux|grep mysql|grep 'my.cnf'
   ```

   显然没有,那么查看mysql使用的配置文件

   ```
   mysql --help|grep 'my.cnf'
   
   /etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf 
   ```

   挨个查看文件。使用的是 `` /etc/mysql/my.cnf``

   ```
   vim  /etc/mysql/my.cnf
   
   # The MySQL database server configuration file.
   #
   # You can copy this to one of:
   # - "/etc/mysql/my.cnf" to set global options,
   # - "~/.my.cnf" to set user-specific options.
   # 
   # One can use all long options that the program supports.
   # Run program with --help to get a list of available options and with
   # --print-defaults to see which it would actually understand and use.
   #
   # For explanations see
   # http://dev.mysql.com/doc/mysql/en/server-system-variables.html
   
   #
   # * IMPORTANT: Additional settings that can override those from this file!
   #   The files must end with '.cnf', otherwise they'll be ignored.
   #
   
   !includedir /etc/mysql/conf.d/
   !includedir /etc/mysql/mysql.conf.d/
   ```

   查看 ``/etc/mysql/conf.d/`` 和``/etc/mysql/mysql.conf.d/`` 目录

   ```
   root@VM-0-17-ubuntu:/data/soft/zkui# cd /etc/mysql/mysql.conf.d/
   root@VM-0-17-ubuntu:/etc/mysql/mysql.conf.d# ls
   mysqld.cnf  mysqld_safe_syslog.cnf
   root@VM-0-17-ubuntu:/etc/mysql/mysql.conf.d# vim mysqld.cnf
   ```

   在这里，那么在[mysqld]下面加入配置

   ```
   [mysqld]
   ##canal config
   log-bin=mysql-bin #添加这一行就ok
   binlog-format=ROW #选择row模式
   server_id=1 #配置mysql replaction需要定义，不能和canal的slaveId重复,这个在1.1.0里面已经不用设置
   ```

   重启

   ```
   service mysql restart
   ```

    * 创建canal使用的mysql账号

      ```
      CREATE USER canal IDENTIFIED BY 'canal';  
      GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
      -- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
      FLUSH PRIVILEGES;
      ```

### Canal安装配置

* 下载

  我们下载``canal.deployer-1.1.0.tar.gz ``,先不配置kafka.

  ```
  https://github.com/alibaba/canal/releases
  ```

  ```
  tar -zxvf canal.deployer-1.1.0.tar.gz -C canal-1.1.0
  ```

* 修改配置

  * 修改canal.properties，这个是全局的配置

    ```
    cd  canal-1.1.0/conf
    
    vim canal.properties
    ```

    ```
    ################################################
    #########               common argument         ############# 
    #################################################
    canal.id= 1
    ##这里可以指定ip，但是本人在腾讯云上不配置的时候，读取到的是腾讯云的内网ip
    canal.ip=
    ##canal的端口
    canal.port=11111
    canal.metrics.pull.port=11112
    ##zookeeper的IP要配置，多个用逗号隔开
    canal.zkServers=127.0.0.1:2181
    # flush data to zk
    canal.zookeeper.flush.period = 1000
    canal.withoutNetty = false
    # flush meta cursor/parse position to file
    canal.file.data.dir = ${canal.conf.dir}
    canal.file.flush.period = 1000
    ## memory store RingBuffer size, should be Math.pow(2,n)
    canal.instance.memory.buffer.size = 16384
    ## memory store RingBuffer used memory unit size , default 1kb
    canal.instance.memory.buffer.memunit = 1024 
    ## meory store gets mode used MEMSIZE or ITEMSIZE
    canal.instance.memory.batch.mode = MEMSIZE
    
    ## detecing config
    canal.instance.detecting.enable = false
    #canal.instance.detecting.sql = insert into retl.xdual values(1,now()) on duplicate 	# 	key update x=now()
    canal.instance.detecting.sql = select 1
    canal.instance.detecting.interval.time = 3
    canal.instance.detecting.retry.threshold = 3
    canal.instance.detecting.heartbeatHaEnable = false
    
    # support maximum transaction size, more than the size of the transaction will be 		# cut into multiple transactions delivery
    canal.instance.transaction.size =  1024
    # mysql fallback connected to new master should fallback times
    canal.instance.fallbackIntervalInSeconds = 60
    
    # network config
    canal.instance.network.receiveBufferSize = 16384
    canal.instance.network.sendBufferSize = 16384
    canal.instance.network.soTimeout = 30
    # binlog filter config
    canal.instance.filter.druid.ddl = true
    canal.instance.filter.query.dcl = false
    canal.instance.filter.query.dml = false
    canal.instance.filter.query.ddl = false
    canal.instance.filter.table.error = false
    canal.instance.filter.rows = false
    canal.instance.filter.transaction.entry = false
    
    # binlog format/image check
    canal.instance.binlog.format = ROW,STATEMENT,MIXED 
    canal.instance.binlog.image = FULL,MINIMAL,NOBLOB
    
    # binlog ddl isolation
    canal.instance.get.ddl.isolation = false
    
    # parallel parser config 
    ##单核cpu的机器上下面这个参数设置为true会报错，官方is解释说这是个bug，如果单核机器跑就设置false
    canal.instance.parser.parallel = true
    ## concurrent thread number, default 60% available processors, suggest not to exceed Runtime.getRuntime().availableProcessors()
    #canal.instance.parser.parallelThreadSize = 16
    ## disruptor ringbuffer size, must be power of 2
    canal.instance.parser.parallelBufferSize = 256
    
    # table meta tsdb info 这个是1.1.0新加的目前不知道这是干嘛的，好像是存metadata的
    canal.instance.tsdb.enable=true
    canal.instance.tsdb.dir=${canal.file.data.dir:../conf}/${canal.instance.destination:}
    canal.instance.tsdb.url=jdbc:h2:${canal.instance.tsdb.dir}/h2;CACHE_SIZE=1000;MODE=MYSQL;
    canal.instance.tsdb.dbUsername=canal
    canal.instance.tsdb.dbPassword=8Kvk,"qlm-%<#h
    
    # rds oss binlog account
    canal.instance.rds.accesskey =
    canal.instance.rds.secretkey =
    
    #################################################
    #########               destinations            ############# 
    #################################################
    ## 多个example在此处进行配置,
    canal.destinations= example
    # conf root dir
    canal.conf.dir = ../conf
    # auto scan instance dir add/remove and start/stop instance
    canal.auto.scan = true
    canal.auto.scan.interval = 5
    
    canal.instance.tsdb.spring.xml=classpath:spring/tsdb/h2-tsdb.xml
    #canal.instance.tsdb.spring.xml=classpath:spring/tsdb/mysql-tsdb.xml
    
    canal.instance.global.mode = spring 
    canal.instance.global.lazy = false
    #canal.instance.global.manager.address = 127.0.0.1:1099
    #canal.instance.global.spring.xml = classpath:spring/memory-instance.xml
    #canal.instance.global.spring.xml = classpath:spring/file-instance.xml
    ##我们使用zk实现ha 那么用下面这个spring的模式
    canal.instance.global.spring.xml = classpath:spring/default-instance.xml
    ```

  * 修改 instance.properties

    ```
    vim /data/soft/canal-1.1.0/conf/example/instance.properties
    ```

    ```
    
    #################################################
    ## mysql serverId , v1.0.26+ will autoGen 
    # canal.instance.mysql.slaveId=0
    
    # enable gtid use true/false 这个是新加的特性
    canal.instance.gtidon=false
    
    # position info 配置要伪装的mysql的地址
    canal.instance.master.address=127.0.0.1:3306
    ##canal.instance.master.journal.name +  canal.instance.master.position :  精确指定一个 ##binlog 位点，进行启动
    canal.instance.master.journal.name=
    canal.instance.master.position=
    ##canal.instance.master.timestamp :  指定一个时间戳，canal 会自动遍历 mysql binlog，找到对##应时间戳的 binlog 位点后，进行启动
    canal.instance.master.timestamp=
    canal.instance.master.gtid=
    
    # rds oss binlog
    canal.instance.rds.accesskey=
    canal.instance.rds.secretkey=
    canal.instance.rds.instanceId=
    
    # table meta tsdb info
    canal.instance.tsdb.enable=true
    ##匹配
    #canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/skyeye
    #canal.instance.tsdb.dbUsername=canal
    #canal.instance.tsdb.dbPassword=8Kvk,"qlm-%<#h
    
    #canal.instance.standby.address =
    #canal.instance.standby.journal.name =
    #canal.instance.standby.position = 
    #canal.instance.standby.timestamp =
    #canal.instance.standby.gtid=
    
    # username/password
    ##配置为自己的数据库的账号和密码。
    canal.instance.dbUsername=canal
    canal.instance.dbPassword=8Kvk,"qlm-%<#h
    ##代表数据库的编码方式对应到 java 中的编码类型，比如 UTF-8，GBK , ISO-8859-1
    canal.instance.connectionCharset=UTF-8
    
    # table regex 1.1.0去掉了默认db的设置只能用正则去过滤。
    canal.instance.filter.regex=skyeye\\..*
    # table black regex
    canal.instance.filter.black.regex=
    #################################################
    ```

* 启动和停止

  * 启动

    ```
    cd /data/soft/canal-1.1.0/bin
    
    ./startup.sh
    ```

  * 日志查看 ,canal目录为服务的日志，example里为节点实例日志

    ```
    cd /data/soft/canal-1.1.0/logs
    ll
    drwxrwxrwx 4 root root 4096 Sep 17 15:59 ./
    drwxr-xr-x 6 root root 4096 Sep 17 15:32 ../
    drwxr-xr-x 3 root root 4096 Sep 18 11:02 canal/
    drwxr-xr-x 3 root root 4096 Sep 18 09:45 example/
    ```

  * 停止

    ```
    ./stop.sh
    ```

#### TODO:joy:今天就到这里吧。下次补充Kafka

[]: https://github.com/alibaba/canal/wiki	"canal官档"

