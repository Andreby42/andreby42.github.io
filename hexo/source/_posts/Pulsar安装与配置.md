---
title: Pulsar安装与配置
date: 2018-12-02 22:32:47
tags: [Pulsar]
categories: [Pulsar]
---

Pulsar的安装与配置<!--more-->

### 环境概述

* Linux环境Ubuntu16.04
* java 1.8
* zookeeper 3.4.13

### 安装

* 下载

  ```
  wget https://www.apache.org/dyn/mirrors/mirrors.cgi?action=download&filename=pulsar/pulsar-2.2.0/apache-pulsar-2.2.0-bin.tar.gz
  ```

* 解压

  ```
  tar -zxvf  apache-pulsar-2.2.0-bin.tar.gz -C pulsar
  ```

* 目录介绍

  ```
  drwxr-xr-x  8 root root   4096 Dec  2 15:01 ./
  drwxr-xr-x 15 root root   4096 Dec  2 15:33 ../
  drwxr-xr-x  3  501 staff  4096 Oct 13 06:01 bin/
  drwxr-xr-x  4  501 staff  4096 Dec  2 22:35 conf/
  drwxr-xr-x  3 root root   4096 Dec  2 15:01 examples/
  drwxr-xr-x  3 root root   4096 Dec  2 15:01 instances/
  drwxr-xr-x  3 root root  20480 Dec  2 15:01 lib/
  -rw-r--r--  1  501 staff 28876 Oct 13 06:01 LICENSE
  drwxr-xr-x  2  501 staff  4096 Oct 13 06:01 licenses/
  -rw-r--r--  1  501 staff  5857 Oct 13 06:01 NOTICE
  -rw-r--r--  1  501 staff  1269 Oct 13 06:01 README
  
  ```

  * bin：Pulsar 的命令行工具，例如 pulsar 和 pulsar-admin 
  * conf: Pulsar 的配置文件，包括 broker 配置、zookeeper 配置等等 
  * examples: Pulsar function 的示例，有java的jar 和yaml配置文件
  * instances：为Pulsar function 所创建的一些组件
  * lib ： Pulsar依赖的jar
  * LICENSE：许可证文件
  * licenses：一些依赖的许可证文件的目录

  一旦你开始运行 Pulsar，下面的这些目录将会被创建：

  * data ：ZooKeeper 和 BookKeeper 使用的数据存储目录 
  * instances ：为 Pulsar Function 创建的 Artifact 
  * logs :安装时创建的 log  

* 单机模式启动

  运行`pulsar help` 查看帮助命令

  ```
  Usage: pulsar <command>
  where command is one of:
  
      broker              Run a broker server
      bookie              Run a bookie server
      zookeeper           Run a zookeeper server
      configuration-store Run a configuration-store server
      discovery           Run a discovery server
      proxy               Run a pulsar proxy
      websocket           Run a web socket proxy server
      functions-worker    Run a functions worker server
      sql-worker          Run a sql worker server
      sql                 Run sql CLI
      standalone          Run a broker server with local bookies and local zookeeper
  
      initialize-cluster-metadata     One-time metadata initialization
      compact-topic       Run compaction against a topic
      zookeeper-shell     Open a ZK shell client
  
      help                This help message
  
  or command is the full name of a class with a defined main() method.
  
  Environment variables:
     PULSAR_LOG_CONF               Log4j configuration file (default /data/soft/pulsar/conf/log4j2.yaml)
     PULSAR_BROKER_CONF            Configuration file for broker (default: /data/soft/pulsar/conf/broker.conf)
     PULSAR_BOOKKEEPER_CONF        Configuration file for bookie (default: /data/soft/pulsar/conf/bookkeeper.conf)
     PULSAR_ZK_CONF                Configuration file for zookeeper (default: /data/soft/pulsar/conf/zookeeper.conf)
     PULSAR_CONFIGURATION_STORE_CONF         Configuration file for global configuration store (default: /data/soft/pulsar/conf/global_zookeeper.conf)
     PULSAR_DISCOVERY_CONF         Configuration file for discovery service (default: /data/soft/pulsar/conf/discovery.conf)
     PULSAR_WEBSOCKET_CONF         Configuration file for websocket proxy (default: /data/soft/pulsar/conf/websocket.conf)
     PULSAR_PROXY_CONF             Configuration file for Pulsar proxy (default: /data/soft/pulsar/conf/proxy.conf)
     PULSAR_WORKER_CONF            Configuration file for functions worker (default: /data/soft/pulsar/conf/functions_worker.yml)
     PULSAR_STANDALONE_CONF        Configuration file for standalone (default: /data/soft/pulsar/conf/standalone.conf)
     PULSAR_PRESTO_CONF            Configuration directory for Pulsar Presto (default: /data/soft/pulsar/conf/presto)
     PULSAR_EXTRA_OPTS             Extra options to be passed to the jvm
     PULSAR_EXTRA_CLASSPATH        Add extra paths to the pulsar classpath
     PULSAR_PID_DIR                Folder where the pulsar server PID file should be stored
     PULSAR_STOP_TIMEOUT           Wait time before forcefully kill the pulsar server instance, if the stop is not successful
  
  These variable can also be set in conf/pulsar_env.sh
  
  ```

  使用standalone  `Run a broker server with local bookies and local zookeeper`

  ```
  bin/pulsar standalone
  ```

  报错了

  ```
  Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x0000000080000000, 2147483648, 0) failed; error='Cannot allocate memory' (errno=12)
  #
  # There is insufficient memory for the Java Runtime Environment to continue.
  # Native memory allocation (mmap) failed to map 2147483648 bytes for committing reserved memory.
  # An error report file with more information is saved as:
  # /data/soft/pulsar/hs_err_pid10684.log
  ```

  ```
  ...
  There is insufficient memory for the Java Runtime Environment to continue.
  ...
  ```

  你说气人不气人，我的小鸡内存不足了。

* 下载内置连接器

  ```
  $ wget https://archive.apache.org/dist/pulsar/pulsar-2.2.0/apache-pulsar-io-connectors-2.2.0-bin.tar.gz
  ```

  下载完成后在 pulsar 目录中，解压缩 io-connectors 软件包并复制连接器，如`connectors` pulsar 目录中所示： 

  ```
  tar xvfz /path/to/apache-pulsar-io-connectors-2.2.0-bin.tar.gz
  // you will find a directory named `apache-pulsar-io-connectors-2.2.0` in the pulsar directory
  // then copy the connectors
  
  $ cd apache-pulsar-io-connectors-2.2.0/connectors connectors
  
  $ ls connectors
  pulsar-io-aerospike-2.2.0.nar
  pulsar-io-cassandra-2.2.0.nar
  pulsar-io-kafka-2.2.0.nar
  pulsar-io-kinesis-2.2.0.nar
  pulsar-io-rabbitmq-2.2.0.nar
  pulsar-io-twitter-2.2.0.nar
  ...
  ```

  

### 配置

* BrokerKeeper ,是一个复制的日志存储系统，Pulsar 使用它来持久存储所有消息。

  | 名称                             | 描述                                                         | 默认值                                                   |
  | -------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
  | bookiePort                       | bookie 服务器侦听的端口。                                    | 3181                                                     |
  | allowLoopback                    | Whether the bookie is allowed to use a loopback interface as its primary interface (i.e. the interface used to establish its identity). By default, loopback interfaces are not allowed as the primary interface. Using a loopback interface as the primary interface usually indicates a configuration error. For example, it’s fairly common in some VPS setups to not configure a hostname or to have the hostname resolve to `127.0.0.1`. If this is the case, then all bookies in the cluster will establish their identities as `127.0.0.1:3181` and only one will be able to join the cluster. For VPSs configured like this, you should explicitly set the listening interface. | false                                                    |
  | listeningInterface               | bookie 侦听的网络接口。如果没有设置，bookie 将监听所有接口。 | eth0                                                     |
  | journalDirectory                 | The directory where Bookkeeper outputs its write-ahead log (WAL) | data/bookkeeper/journal                                  |
  | ledgerDirectories                | The directory where Bookkeeper outputs ledger snapshots. This could define multiple directories to store snapshots separated by comma, for example `ledgerDirectories=/tmp/bk1-data,/tmp/bk2-data`. Ideally, ledger dirs and the journal dir are each in a different device, which reduces the contention between random I/O and sequential write. It is possible to run with a single disk, but performance will be significantly lower. | data/bookkeeper/ledgers                                  |
  | ledgerManagerType                | The type of ledger manager used to manage how ledgers are stored, managed, and garbage collected. See [BookKeeper Internals](http://bookkeeper.apache.org/docs/latest/getting-started/concepts) for more info | hierarchical                                             |
  | zkLedgersRootPath                | The root ZooKeeper path used to store ledger metadata. This parameter is used by the ZooKeeper-based ledger manager as a root znode to store all ledgers. | /ledgers                                                 |
  | ledgerStorageClass               | Ledger storage implementation class                          | org.apache.bookkeeper.bookie.storage.ldb.DbLedgerStorage |
  | entryLogFilePreallocationEnabled | Enable or disable entry logger preallocation                 |                                                          |
  | logSizeLimit                     | Max file size of the entry logger, in bytes. A new entry log file will be created when the old one reaches the file size limitation. | 2147483648                                               |
  | minorCompactionThreshold         | Threshold of minor compaction. Entry log files whose remaining size percentage reaches below this threshold will be compacted in a minor compaction. If set to less than zero, the minor compaction is disabled. | 0.2                                                      |
  | minorCompactionInterval          | Time interval to run minor compaction, in seconds. If set to less than zero, the minor compaction is disabled. | 3600                                                     |
  | majorCompactionThreshold         | The threshold of major compaction. Entry log files whose remaining size percentage reaches below this threshold will be compacted in a major compaction. Those entry log files whose remaining size percentage is still higher than the threshold will never be compacted. If set to less than zero, the minor compaction is disabled. | 0.5                                                      |
  | majorCompactionInterval          | The time interval to run major compaction, in seconds. If set to less than zero, the major compaction is disabled. | 86400                                                    |
  | compactionMaxOutstandingRequests | Sets the maximum number of entries that can be compacted without flushing. When compacting, the entries are written to the entrylog and the new offsets are cached in memory. Once the entrylog is flushed the index is updated with the new offsets. This parameter controls the number of entries added to the entrylog before a flush is forced. A higher value for this parameter means more memory will be used for offsets. Each offset consists of 3 longs. This parameter should not be modified unless you’re fully aware of the consequences. | 100000                                                   |
  | compactionRate                   | The rate at which compaction will read entries, in adds per second. | 1000                                                     |
  | isThrottleByBytes                | Throttle compaction by bytes or by entries.                  | false                                                    |
  | compactionRateByEntries          | The rate at which compaction will read entries, in adds per second. | 1000                                                     |
  | compactionRateByBytes            | Set the rate at which compaction will readd entries. The unit is bytes added per second. | 1000000                                                  |
  | journalMaxSizeMB                 | Max file size of journal file, in megabytes. A new journal file will be created when the old one reaches the file size limitation. | 2048                                                     |
  | journalMaxBackups                | The max number of old journal filse to keep. Keeping a number of old journal files would help data recovery in special cases. | 5                                                        |
  | journalPreAllocSizeMB            | How space to pre-allocate at a time in the journal.          | 16                                                       |
  | journalWriteBufferSizeKB         | 用于日志的写缓冲区。                                         | 64                                                       |
  | journalRemoveFromPageCache       | Whether pages should be removed from the page cache after force write. | true                                                     |
  | journalAdaptiveGroupWrites       | Whether to group journal force writes, which optimizes group commit for higher throughput. | true                                                     |
  | journalMaxGroupWaitMSec          | The maximum latency to impose on a journal write to achieve grouping. | 1                                                        |
  | journalAlignmentSize             | All the journal writes and commits should be aligned to given size | 4096                                                     |
  | journalBufferedWritesThreshold   | Maximum writes to buffer to achieve grouping                 | 524288                                                   |
  | journalFlushWhenQueueEmpty       | If we should flush the journal when journal queue is empty   | false                                                    |
  | numJournalCallbackThreads        |                                                              |                                                          |

   

* pulsar_env.sh

  默认的启动pulsar broker的配置

  ```
  
  # default settings for starting pulsar broker
  
  # Log4j configuration file
  # PULSAR_LOG_CONF=
  
  # Logs location
  # PULSAR_LOG_DIR=
  
  # Configuration file of settings used in broker server
  # PULSAR_BROKER_CONF=
  
  # Configuration file of settings used in bookie server
  # PULSAR_BOOKKEEPER_CONF=
  
  # Configuration file of settings used in zookeeper server
  # PULSAR_ZK_CONF=
  
  # Configuration file of settings used in global zookeeper server
  # PULSAR_GLOBAL_ZK_CONF=
  
  # Extra options to be passed to the jvm
  PULSAR_MEM=" -Xms1g -Xmx1g -XX:MaxDirectMemorySize=1g"
  
  # Garbage collection options GC参数的配置
  PULSAR_GC=" -XX:+UseG1GC -XX:MaxGCPauseMillis=10 -XX:+ParallelRefProcEnabled -XX:+UnlockExperimentalVMOptions -XX:+AggressiveOpts -XX:+DoEscapeAnalysis -XX:ParallelGCThreads=32 -XX:ConcGCThreads=32 -XX:G1NewSizePercent=50 -XX:+DisableExplicitGC -XX:-ResizePLAB"
  
  # Extra options to be passed to the jvm
  PULSAR_EXTRA_OPTS="${PULSAR_EXTRA_OPTS} ${PULSAR_MEM} ${PULSAR_GC} -Dio.netty.leakDetectionLevel=disabled -Dio.netty.recycler.maxCapacity.default=1000 -Dio.netty.recycler.linkCapacity=1024"
  
  # Add extra paths to the bookkeeper classpath
  # PULSAR_EXTRA_CLASSPATH=
  
  #Folder where the Bookie server PID file should be stored
  #PULSAR_PID_DIR=
  
  #Wait time before forcefully kill the pulser server instance, if the stop is not successful
  #PULSAR_STOP_TIMEOUT=
  
  ```

* pulsar_tools_env.sh

  ```
  #!/usr/bin/env bash
  #
  # Licensed to the Apache Software Foundation (ASF) under one
  # or more contributor license agreements.  See the NOTICE file
  # distributed with this work for additional information
  # regarding copyright ownership.  The ASF licenses this file
  # to you under the Apache License, Version 2.0 (the
  # "License"); you may not use this file except in compliance
  # with the License.  You may obtain a copy of the License at
  #
  #   http://www.apache.org/licenses/LICENSE-2.0
  #
  # Unless required by applicable law or agreed to in writing,
  # software distributed under the License is distributed on an
  # "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  # KIND, either express or implied.  See the License for the
  # specific language governing permissions and limitations
  # under the License.
  #
  
  # Set JAVA_HOME here to override the environment setting
  # JAVA_HOME=
  
  # default settings for starting pulsar broker
  
  # Log4j configuration file
  # PULSAR_LOG_CONF=
  
  # Logs location
  # PULSAR_LOG_DIR=
  
  # Configuration file of settings used in broker server
  # PULSAR_BROKER_CONF=
  
  # Configuration file of settings used in bookie server
  # PULSAR_BOOKKEEPER_CONF=
  
  # Configuration file of settings used in zookeeper server
  # PULSAR_ZK_CONF=
  
  # Configuration file of settings used in global zookeeper server
  # PULSAR_GLOBAL_ZK_CONF=
  
  # Extra options to be passed to the jvm
  PULSAR_MEM=${PULSAR_MEM:-"-Xmx256m -XX:MaxDirectMemorySize=256m"}
  
  # Garbage collection options
  PULSAR_GC=" -client "
  
  # Extra options to be passed to the jvm
  PULSAR_EXTRA_OPTS="${PULSAR_EXTRA_OPTS} ${PULSAR_MEM} ${PULSAR_GC} -Dio.netty.leakDetectionLevel=disabled"
  
  # Add extra paths to the bookkeeper classpath
  # PULSAR_EXTRA_CLASSPATH=
  
  #Folder where the Bookie server PID file should be stored
  #PULSAR_PID_DIR=
  
  #Wait time before forcefully kill the pulser server instance, if the stop is not successful
  #PULSAR_STOP_TIMEOUT=
  ```

* broker.conf 定义broker的一些配置

  ```
  #
  # Licensed to the Apache Software Foundation (ASF) under one
  # or more contributor license agreements.  See the NOTICE file
  # distributed with this work for additional information
  # regarding copyright ownership.  The ASF licenses this file
  # to you under the Apache License, Version 2.0 (the
  # "License"); you may not use this file except in compliance
  # with the License.  You may obtain a copy of the License at
  #
  #   http://www.apache.org/licenses/LICENSE-2.0
  #
  # Unless required by applicable law or agreed to in writing,
  # software distributed under the License is distributed on an
  # "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  # KIND, either express or implied.  See the License for the
  # specific language governing permissions and limitations
  # under the License.
  #
  
  ### --- General broker settings --- ###
  
  # Zookeeper quorum connection string
  zookeeperServers=172.17.83.72:2181
  
  # Global Zookeeper quorum connection string
  globalZookeeperServers=172.17.83.72:2181
  
  # Broker data port
  brokerServicePort=6650
  
  # Broker data port for TLS
  brokerServicePortTls=6651
  
  # Port to use to server HTTP request
  webServicePort=8080
  
  # Port to use to server HTTPS request
  webServicePortTls=8443
  
  # Hostname or IP address the service binds on, default is 0.0.0.0.
  bindAddress=0.0.0.0
  
  # Hostname or IP address the service advertises to the outside world. If not set, the value of InetAddress.getLocalHost().getHostName() is used.
  advertisedAddress=172.17.83.72
  
  # Name of the cluster to which this broker belongs to
  clusterName=pulsar-cluster-1
  
  # Enable cluster's failure-domain which can distribute brokers into logical region
  failureDomainsEnabled=false
  
  # Zookeeper session timeout in milliseconds
  zooKeeperSessionTimeoutMillis=30000
  
  # Time to wait for broker graceful shutdown. After this time elapses, the process will be killed
  ##定义优雅关机超时时间
  brokerShutdownTimeoutMs=3000
  
  # Enable backlog quota check. Enforces action on topic when the quota is reached
  backlogQuotaCheckEnabled=true
  
  # How often to check for topics that have reached the quota
  backlogQuotaCheckIntervalInSeconds=60
  
  # Default per-topic backlog quota limit
  backlogQuotaDefaultLimitGB=10
  
  # Enable the deletion of inactive topics
  brokerDeleteInactiveTopicsEnabled=true
  
  # How often to check for inactive topics
  brokerDeleteInactiveTopicsFrequencySeconds=60
  
  # How frequently to proactively check and purge expired messages
  messageExpiryCheckIntervalInMinutes=5
  
  # How long to delay rewinding cursor and dispatching messages when active consumer is changed
  activeConsumerFailoverDelayTimeMillis=1000
  
  # Set the default behavior for message deduplication in the broker
  # This can be overridden per-namespace. If enabled, broker will reject
  # messages that were already stored in the topic
  brokerDeduplicationEnabled=false
  
  # Maximum number of producer information that it's going to be
  # persisted for deduplication purposes
  brokerDeduplicationMaxNumberOfProducers=10000
  
  # Number of entries after which a dedup info snapshot is taken.
  # A bigger interval will lead to less snapshots being taken though it would
  # increase the topic recovery time, when the entries published after the
  # snapshot need to be replayed
  brokerDeduplicationEntriesInterval=1000
  
  # Time of inactivity after which the broker will discard the deduplication information
  # relative to a disconnected producer. Default is 6 hours.
  brokerDeduplicationProducerInactivityTimeoutMinutes=360
  
  # When a namespace is created without specifying the number of bundle, this
  # value will be used as the default
  defaultNumberOfNamespaceBundles=3
  
  # Enable check for minimum allowed client library version
  clientLibraryVersionCheckEnabled=false
  
  # Path for the file used to determine the rotation status for the broker when responding
  # to service discovery health checks
  statusFilePath=
  
  # If true, (and ModularLoadManagerImpl is being used), the load manager will attempt to
  # use only brokers running the latest software version (to minimize impact to bundles)
  preferLaterVersions=false
  
  # Max number of unacknowledged messages allowed to receive messages by a consumer on a shared subscription. Broker will stop sending
  # messages to consumer once, this limit reaches until consumer starts acknowledging messages back.
  # Using a value of 0, is disabling unackeMessage limit check and consumer can receive messages without any restriction
  maxUnackedMessagesPerConsumer=50000
  
  # Max number of unacknowledged messages allowed per shared subscription. Broker will stop dispatching messages to
  # all consumers of the subscription once this limit reaches until consumer starts acknowledging messages back and
  # unack count reaches to limit/2. Using a value of 0, is disabling unackedMessage-limit
  # check and dispatcher can dispatch messages without any restriction
  maxUnackedMessagesPerSubscription=200000
  
  # Max number of unacknowledged messages allowed per broker. Once this limit reaches, broker will stop dispatching
  # messages to all shared subscription which has higher number of unack messages until subscriptions start
  # acknowledging messages back and unack count reaches to limit/2. Using a value of 0, is disabling
  # unackedMessage-limit check and broker doesn't block dispatchers
  maxUnackedMessagesPerBroker=0
  
  # Once broker reaches maxUnackedMessagesPerBroker limit, it blocks subscriptions which has higher unacked messages
  # than this percentage limit and subscription will not receive any new messages until that subscription acks back
  # limit/2 messages
  maxUnackedMessagesPerSubscriptionOnBrokerBlocked=0.16
  
  # Default messages per second dispatch throttling-limit for every topic. Using a value of 0, is disabling default
  # message dispatch-throttling
  dispatchThrottlingRatePerTopicInMsg=0
  
  # Default bytes per second dispatch throttling-limit for every topic. Using a value of 0, is disabling
  # default message-byte dispatch-throttling
  dispatchThrottlingRatePerTopicInByte=0
  
  # Default dispatch-throttling is disabled for consumers which already caught-up with published messages and
  # don't have backlog. This enables dispatch-throttling for non-backlog consumers as well.
  dispatchThrottlingOnNonBacklogConsumerEnabled=false
  
  # Max number of concurrent lookup request broker allows to throttle heavy incoming lookup traffic
  maxConcurrentLookupRequest=10000
  
  # Max number of concurrent topic loading request broker allows to control number of zk-operations
  maxConcurrentTopicLoadRequest=5000
  
  # Max concurrent non-persistent message can be processed per connection
  maxConcurrentNonPersistentMessagePerConnection=1000
  
  # Number of worker threads to serve non-persistent topic
  numWorkerThreadsForNonPersistentTopic=8
  
  # Enable broker to load persistent topics
  enablePersistentTopics=true
  
  # Enable broker to load non-persistent topics
  enableNonPersistentTopics=true
  
  # Enable to run bookie along with broker
  enableRunBookieTogether=false
  
  # Enable to run bookie autorecovery along with broker
  enableRunBookieAutoRecoveryTogether=false
  
  ### --- Authentication --- ###
  # Role names that are treated as "proxy roles". If the broker sees a request with
  #role as proxyRoles - it will demand to see a valid original principal.
  proxyRoles=
  
  # If this flag is set then the broker authenticates the original Auth data
  # else it just accepts the originalPrincipal and authorizes it (if required).  
  authenticateOriginalAuthData=false
  
  # Enable TLS
  tlsEnabled=true
  
  # Path for the TLS certificate file
  tlsCertificateFilePath=/search/data/pulsar/cert/broker/broker.cert.pem
  
  # Path for the TLS private key file
  tlsKeyFilePath=/search/data/pulsar/cert/broker/broker.key-pk8.pem
  
  # Path for the trusted TLS certificate file
  tlsTrustCertsFilePath=/search/data/pulsar/cert/ca/certs/ca.cert.pem
  
  # Accept untrusted TLS certificate from client
  tlsAllowInsecureConnection=false
  
  ### --- Authentication --- ###
  
  # Enable authentication
  authenticationEnabled=true
  
  # Autentication provider name list, which is comma separated list of class names
  authenticationProviders=org.apache.pulsar.broker.authentication.AuthenticationProviderTls
  
  # Enforce authorization
  authorizationEnabled=true
  
  # Authorization provider fully qualified class-name
  authorizationProvider=org.apache.pulsar.broker.authorization.PulsarAuthorizationProvider
  
  # Allow wildcard matching in authorization
  # (wildcard matching only applicable if wildcard-char:
  # * presents at first or last position eg: *.pulsar.service, pulsar.service.*)
  authorizationAllowWildcardsMatching=false
  
  # Role names that are treated as "super-user", meaning they will be able to do all admin
  # operations and publish/consume from all topics
  superUserRoles=admin
  
  # Authentication settings of the broker itself. Used when the broker connects to other brokers,
  # either in same or other clusters
  brokerClientAuthenticationPlugin=
  brokerClientAuthenticationParameters=
  
  # Supported Athenz provider domain names(comma separated) for authentication
  athenzDomainNames=
  
  # When this parameter is not empty, unauthenticated users perform as anonymousUserRole
  anonymousUserRole=
  
  ### --- BookKeeper Client --- ###
  
  # Authentication plugin to use when connecting to bookies
  bookkeeperClientAuthenticationPlugin=
  
  # BookKeeper auth plugin implementatation specifics parameters name and values
  bookkeeperClientAuthenticationParametersName=
  bookkeeperClientAuthenticationParameters=
  
  # Timeout for BK add / read operations
  bookkeeperClientTimeoutInSeconds=30
  
  # Speculative reads are initiated if a read request doesn't complete within a certain time
  # Using a value of 0, is disabling the speculative reads
  bookkeeperClientSpeculativeReadTimeoutInMillis=0
  
  # Enable bookies health check. Bookies that have more than the configured number of failure within
  # the interval will be quarantined for some time. During this period, new ledgers won't be created
  # on these bookies
  bookkeeperClientHealthCheckEnabled=true
  bookkeeperClientHealthCheckIntervalSeconds=60
  bookkeeperClientHealthCheckErrorThresholdPerInterval=5
  bookkeeperClientHealthCheckQuarantineTimeInSeconds=1800
  
  # Enable rack-aware bookie selection policy. BK will chose bookies from different racks when
  # forming a new bookie ensemble
  bookkeeperClientRackawarePolicyEnabled=true
  
  # Enable bookie isolation by specifying a list of bookie groups to choose from. Any bookie
  # outside the specified groups will not be used by the broker
  bookkeeperClientIsolationGroups=
  
  ### --- Managed Ledger --- ###
  
  # Number of bookies to use when creating a ledger
  managedLedgerDefaultEnsembleSize=1
  
  # Number of copies to store for each message
  managedLedgerDefaultWriteQuorum=1
  
  # Number of guaranteed copies (acks to wait before write is complete)
  managedLedgerDefaultAckQuorum=1
  
  # Amount of memory to use for caching data payload in managed ledger. This memory
  # is allocated from JVM direct memory and it's shared across all the topics
  # running  in the same broker
  managedLedgerCacheSizeMB=1024
  
  # Threshold to which bring down the cache level when eviction is triggered
  managedLedgerCacheEvictionWatermark=0.9
  
  # Rate limit the amount of writes per second generated by consumer acking the messages
  managedLedgerDefaultMarkDeleteRateLimit=1.0
  
  # Max number of entries to append to a ledger before triggering a rollover
  # A ledger rollover is triggered on these conditions
  #  * Either the max rollover time has been reached
  #  * or max entries have been written to the ledged and at least min-time
  #    has passed
  managedLedgerMaxEntriesPerLedger=50000
  
  # Minimum time between ledger rollover for a topic
  managedLedgerMinLedgerRolloverTimeMinutes=10
  
  # Maximum time before forcing a ledger rollover for a topic
  managedLedgerMaxLedgerRolloverTimeMinutes=240
  
  # Max number of entries to append to a cursor ledger
  managedLedgerCursorMaxEntriesPerLedger=50000
  
  # Max time before triggering a rollover on a cursor ledger
  managedLedgerCursorRolloverTimeInSeconds=14400
  
  # Max number of "acknowledgment holes" that are going to be persistently stored.
  # When acknowledging out of order, a consumer will leave holes that are supposed
  # to be quickly filled by acking all the messages. The information of which
  # messages are acknowledged is persisted by compressing in "ranges" of messages
  # that were acknowledged. After the max number of ranges is reached, the information
  # will only be tracked in memory and messages will be redelivered in case of
  # crashes.
  managedLedgerMaxUnackedRangesToPersist=10000
  
  # Max number of "acknowledgment holes" that can be stored in Zookeeper. If number of unack message range is higher
  # than this limit then broker will persist unacked ranges into bookkeeper to avoid additional data overhead into
  # zookeeper.
  managedLedgerMaxUnackedRangesToPersistInZooKeeper=1000
  
  # Skip reading non-recoverable/unreadable data-ledger under managed-ledger's list. It helps when data-ledgers gets
  # corrupted at bookkeeper and managed-cursor is stuck at that ledger.
  autoSkipNonRecoverableData=false
  
  ### --- Load balancer --- ###
  
  # Enable load balancer
  loadBalancerEnabled=true
  
  # Percentage of change to trigger load report update
  loadBalancerReportUpdateThresholdPercentage=10
  
  # maximum interval to update load report
  loadBalancerReportUpdateMaxIntervalMinutes=15
  
  # Frequency of report to collect
  loadBalancerHostUsageCheckIntervalMinutes=1
  
  # Load shedding interval. Broker periodically checks whether some traffic should be offload from
  # some over-loaded broker to other under-loaded brokers
  loadBalancerSheddingIntervalMinutes=5
  
  # Prevent the same topics to be shed and moved to other broker more that once within this timeframe
  loadBalancerSheddingGracePeriodMinutes=30
  
  # Usage threshold to allocate max number of topics to broker
  loadBalancerBrokerMaxTopics=50000
  
  # Interval to flush dynamic resource quota to ZooKeeper
  loadBalancerResourceQuotaUpdateIntervalMinutes=15
  
  # enable/disable namespace bundle auto split
  loadBalancerAutoBundleSplitEnabled=true
  
  # enable/disable automatic unloading of split bundles
  loadBalancerAutoUnloadSplitBundlesEnabled=true
  
  # maximum topics in a bundle, otherwise bundle split will be triggered
  loadBalancerNamespaceBundleMaxTopics=1000
  
  # maximum sessions (producers + consumers) in a bundle, otherwise bundle split will be triggered
  loadBalancerNamespaceBundleMaxSessions=1000
  
  # maximum msgRate (in + out) in a bundle, otherwise bundle split will be triggered
  loadBalancerNamespaceBundleMaxMsgRate=30000
  
  # maximum bandwidth (in + out) in a bundle, otherwise bundle split will be triggered
  loadBalancerNamespaceBundleMaxBandwidthMbytes=100
  
  # maximum number of bundles in a namespace
  loadBalancerNamespaceMaximumBundles=128
  
  # Override the auto-detection of the network interfaces max speed. 
  # This option is useful in some environments (eg: EC2 VMs) where the max speed
  # reported by Linux is not reflecting the real bandwidth available to the broker.
  # Since the network usage is employed by the load manager to decide when a broker
  # is overloaded, it is important to make sure the info is correct or override it 
  # with the right value here. The configured value can be a double (eg: 0.8) and that
  # can be used to trigger load-shedding even before hitting on NIC limits.
  loadBalancerOverrideBrokerNicSpeedGbps=
  
  # Name of load manager to use
  loadManagerClassName=org.apache.pulsar.broker.loadbalance.impl.ModularLoadManagerImpl
  
  ### --- Replication --- ###
  
  # Enable replication metrics
  replicationMetricsEnabled=true
  
  # Max number of connections to open for each broker in a remote cluster
  # More connections host-to-host lead to better throughput over high-latency
  # links.
  replicationConnectionsPerBroker=16
  
  # Replicator producer queue size
  replicationProducerQueueSize=1000
  
  # Replicator prefix used for replicator producer name and cursor name
  replicatorPrefix=pulsar.repl
  
  # Enable TLS when talking with other clusters to replicate messages
  replicationTlsEnabled=false
  
  # Default message retention time
  defaultRetentionTimeInMinutes=0
  
  # Default retention size
  defaultRetentionSizeInMB=0
  
  # How often to check whether the connections are still alive
  keepAliveIntervalSeconds=30
  
  # How often broker checks for inactive topics to be deleted (topics with no subscriptions and no one connected)
  brokerServicePurgeInactiveFrequencyInSeconds=60
  
  ### --- WebSocket --- ###
  
  # Enable the WebSocket API service in broker
  webSocketServiceEnabled=false
  
  # Number of IO threads in Pulsar Client used in WebSocket proxy
  webSocketNumIoThreads=8
  
  # Number of connections per Broker in Pulsar Client used in WebSocket proxy
  webSocketConnectionsPerBroker=8
  
  
  ### --- Metrics --- ###
  
  # Enable topic level metrics
  exposeTopicLevelMetricsInPrometheus=true
  ```

* client.conf 定义客户端的一些配置

  ```
  # Configuration for pulsar-client and pulsar-admin CLI tools
  
  # URL for Pulsar REST API (for admin operations)
  # For TLS:
  # webServiceUrl=https://localhost:8443/
  webServiceUrl=http://localhost:8080/
  
  # URL for Pulsar Binary Protocol (for produce and consume operations)
  # For TLS:
  # brokerServiceUrl=pulsar+ssl://localhost:6651/
  brokerServiceUrl=pulsar://localhost:6650/
  
  # Authentication plugin to authenticate with servers
  # e.g. for TLS
  # authPlugin=org.apache.pulsar.client.impl.auth.AuthenticationTls
  authPlugin=
  
  # Parameters passed to authentication plugin.
  # A comma separated list of key:value pairs.
  # Keys depend on the configured authPlugin.
  # e.g. for TLS
  # authParams=tlsCertFile:/path/to/client-cert.pem,tlsKeyFile:/path/to/client-key.pem
  authParams=
  
  # Allow TLS connections to servers whose certificate cannot be
  # be verified to have been signed by a trusted certificate
  # authority.
  tlsAllowInsecureConnection=false
  
  # Whether server hostname must match the common name of the certificate
  # the server is using.
  tlsEnableHostnameVerification=false
  
  # Path for the trusted TLS certificate file.
  # This cert is used to verify that any cert presented by a server
  # is signed by a certificate authority. If this verification
  # fails, then the cert is untrusted and the connection is dropped.
  tlsTrustCertsFilePath=
  
  ```

  





