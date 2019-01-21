---
title: CassandraLinux服务端安装配置
date: 2019-01-21 22:40:52
tags: [Cassandra]
categories: [Cassandra]
---

Cassandra服务端安装配置<!--more-->

#### 下载

使用安装包的方式安装

```
sudo apt-get update
```

```
wget http://httpd-mirror.sergal.org/apache/cassandra/3.11.3/apache-cassandra-3.11.3-bin.tar.gz
```

```
tar -zxvf apache-cassandra-3.11.3-bin.tar.gz
```

```
mv apache-cassandra-3.11.3 cassandra3.11.3
```

#### 配置

```
cd /data/soft/cassandra/conf
```

* 设置数据存储目录

  ```
  mkdir /data/cassandra/data
  mkdir /data/cassandra/commitlog
  mkdir /data/cassandra/saved_caches
  ```

* 修改`cassandra.yaml`

  ```
  
  # The name of the cluster. This is mainly used to prevent machines in
  # one logical cluster from joining another.
  # 集群的名称。 这主要用于防止一个逻辑集群中的机器加入另一个逻辑集群。
  cluster_name: 'LASER-CLUSTER'
  # This defines the number of tokens randomly assigned to this node on the ring
  # The more tokens, relative to other nodes, the larger the proportion of data
  # that this node will store. You probably want all nodes to have the same number
  # of tokens assuming they have equal hardware capability.
  #
  # If you leave this unspecified, Cassandra will use the default of 1 token for legacy 
  # compatibility,
  # and will use the initial_token as described below.
  #
  # Specifying initial_token will override this setting on the node's initial start,
  # on subsequent starts, this setting will apply even if initial token is set.
  #  
  # If you already have a cluster with 1 token per node, and wish to migrate to 
  # multiple tokens per node, see http://wiki.apache.org/cassandra/Operations
  # 定义了随机分配给环上该节点的令牌数。相对于其他节点，令牌数越多，该节点将存储的数据比例越大默认256
  num_tokens: 256
  # 默认注释的触发此节点的 num_tokens 令牌的自动分配。 分配算法尝试以优化数据中心中的节点上的复制负载# 的方式选择令牌，以用于指定的键空间使用的复制策略。仅支持 Murmur3Partitioner。默认值: KEYSPACE
  # allocate_tokens_for_keyspace: KEYSPACE
  # initial_token 允许您手动指定标记。 虽然您可以使用 vnodes（num_tokens> 1，上面） - 在这种情况
  # 下，您应该提供一个逗号分隔的列表 - 它主要用于将节点添加到未启用 vnode 的旧群集。
  # initial_token:
  # 默认true，启用全局
  hinted_handoff_enabled: true
  max_hint_window_in_ms: 10800000 # 3 hours
  # 每个传递线程的最大速度（KB）/ 秒。 这将与集群中的节点数成比例地减少
  hinted_handoff_throttle_in_kb：1024
  # 传递提示的线程数; 在进行多 dc（datacenter） 部署时，请考虑增加此数，cross-dc handoff tends to 
  # be slower
  max_hints_delivery_threads：2
  # 默认注释项，
  hints_directory
  # 内部缓冲区刷新到磁盘的频率。不会触发 fsync
  hints_flush_period_in_ms
  #单个文件的最大大小 默认128MB
  max_hints_file_size_in_mb：128
  # 默认注释项，压缩以应用于提示文件。 如果省略，hints 文件将被解压缩。支持 LZ4，Snappy 和 Deflate压 # 缩
  hints_compression
  
  # Maximum throttle in KBs per second, total. This will be
  # reduced proportionally to the number of nodes in the cluster.
  # 最大速率（KB）/ 秒。当集群中节点数成比例减少时她也会下降
  batchlog_replay_throttle_in_kb: 1024
  
  # 认证
  #authenticator: AllowAllAuthenticator
  authenticator: PasswordAuthenticator
  # 授权
  #authorizer: AllowAllAuthorizer
  authorizer: PasswordAuthenticator
  # 角色管理器
  role_manager: CassandraRoleManager
  # 角色缓存有效期
  roles_validity_in_ms: 2000
  # 默认注释 角色缓存的刷新间隔（如果已启用）默认为与 roles_validity_in_ms相同值
  # roles_update_interval_in_ms: 2000
  # 权限缓存时间间隔 当位0时候禁用
  permissions_validity_in_ms: 2000
  # 权限刷新时间间隔 默认注释
  # permissions_update_interval_in_ms: 2000
  # 凭证有效期
  credentials_validity_in_ms: 2000
  # 凭证更新周期
  # credentials_update_interval_in_ms: 2000
  # 分区器
  partitioner: org.apache.cassandra.dht.Murmur3Partitioner
  # Directories where Cassandra should store data on disk.  Cassandra
  # will spread data evenly across them, subject to the granularity of
  # the configured compaction strategy.
  # If not set, the default directory is $CASSANDRA_HOME/data/data.
  # 配置数据文件目录
  data_file_directories: /data/cassandra/data
  ...
  # commit log.  when running on magnetic HDD, this should be a
  # separate spindle than the data directories.
  # If not set, the default directory is $CASSANDRA_HOME/data/commitlog.
  # 配置日志文件目录
  commitlog_directory: /data/cassandra/commitlog
  # 每个节点上是否禁用cdc
  cdc_enabled: false
  # cdc raw 的目录
  # cdc_raw_directory: /var/lib/cassandra/cdc_raw
  # 磁盘错误策略 四种 自己看英文注释
  disk_failure_policy: stop
  #提交错误策略 四种 自己看英文注释
  commit_failure_policy: stop
  # 预编译缓存大小 默认位堆内存的1/256或者10mb，以两者中大者为准
  prepared_statements_cache_size_mb:
  #Thrift 预编译语句缓存的最大大小 默认同上
  thrift_prepared_statements_cache_size_mb:
  # 内存中密钥缓存的最大大小 默认位堆的5%或100mb以小者为准，0为禁用密钥缓存
  key_cache_size_in_mb:
  # 密钥缓存时间
  key_cache_save_period: 14400
  # 保存的秘钥数量 默认注释为保存所有
  # key_cache_keys_to_save: 100
  # 行缓存的实现类名（完全堆外行缓存实现默认）
  # row_cache_class_name: org.apache.cassandra.cache.OHCProvider
  # 行缓存大小 0为禁用
  row_cache_size_in_mb: 0
  # 行缓存周期 0位禁用
  row_cache_save_period: 0
  # 内存中计数器的缓存大小 堆的 2.5％和 50MB 中较小的值
  counter_cache_size_in_mb:
  # 内存计数器缓存时间
  counter_cache_save_period: 7200
  # 计数器缓存建的数量 注释为缓存所有
  # counter_cache_keys_to_save: 100
  # 缓存文件目录
  saved_caches_directory: /data/cassandra/saved_caches
  
  
  # periodic或batch默认batch 在批处理模式下，Cassandra 不会进行 ack 写操作，直到提交日志已经同步到
  # 磁盘。它将在同步期间等待 
  # commitlog_sync: batch
  # commitlog_sync_batch_window_in_ms: 2
  # 默认periodic 提交日志的同步刷新策略 及刷新时间间隔
  commitlog_sync: periodic
  commitlog_sync_period_in_ms: 10000
  # 各个commitlog 文件段的大小
  commitlog_segment_size_in_mb: 32
  
  
  seed_provider:
  	    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
        parameters:
            # seeds is actually a comma-delimited list of addresses.
            # Ex: "<ip1>,<ip2>,<ip3>"
            - seeds: "127.0.0.1"
  
  # 并行读写及计数器写的并发数
  concurrent_reads: 32
  concurrent_writes: 32
  concurrent_counter_writes: 32
  # 默认注释sstable池缓存及块缓存大小
  # file_cache_size_in_mb: 512
  # 当 sstable 缓冲池耗尽时是否使用heap
  # buffer_pool_use_heap_if_exhausted: true
  # ssd（对于固态磁盘，默认值）spin（用于机械磁盘） 默认注释 使用ssd
  # disk_optimization_strategy: ssd
  # 用于memtable的堆内存大小
  # memtable_heap_space_in_mb: 2048
  # memtable_offheap_space_in_mb: 2048
  # 不推荐使用 memtable_cleanup_threshold。默认计算是唯一合理的选择
  # memtable_cleanup_threshold: 0.11
  # 指定 Cassandra 分配和管理 memtable 内存的方式。选项包括：heap_buffers（堆nio缓冲区）
  # offheap_buffers(非堆直接缓冲区） offheap_objects（非堆对象）	
  memtable_allocation_type: heap_buffers
  # 提交日志的总分配大小 8192默认
  # commitlog_total_space_in_mb: 8192
  # memtable的写及刷新的线程数
  #memtable_flush_writers: 2
  # cdc总分配数
  # cdc_total_space_in_mb: 4096
  # 检查是否有任何新的 cdc-tracked 表空间可用的时间间隔
  # cdc_free_space_check_interval_ms: 250
  # sstable固定内存池大小 默认5%
  index_summary_capacity_in_mb:
  # 固定内存池重新分配时间间隔
  index_summary_resize_interval_in_minutes: 60
  # 顺序写入时将间隔时间设置为 fsync及刷新的kb大小阈值
  trickle_fsync: false
  trickle_fsync_interval_in_kb: 10240
  
  
  
  # TCP 端口，用于命令和数据, 出于安全原因不应将此端口公开到互联网如果必须公开到互联网需要打开防火墙
  storage_port: 7000
  # ssl端口 加密通信
  ssl_storage_port: 7001
  # 绑定地址或接口并告诉其他 Cassandra 节点连接到该地址或接口 设置 listen_address 或 
  # listen_interface 其中之一，不要对两者都进行设置
  listen_address: localhost
  # listen_interface: eth0
  # listen_interface_prefer_ipv6: false
  # 要广播到其他 Cassandra 节点的地址保留此空白将将其设置为与 listen_address 相同的值
  # broadcast_address: 1.2.3.4
  # listen_on_broadcast_address: false
  # Internode 后端认证
  # internode_authenticator: org.apache.cassandra.auth.AllowAllInternodeAuthenticator
  # 是否启动本地传输服务器。 请注意，本地传输绑定的地址与 rpc_address 相同。端口不同，下面指定。
  start_native_transport: true
  #  CQL本地传输监听客户端
  native_transport_port: 9042
  # 本地ssl传输端口
  # native_transport_port_ssl: 9142
  # 传输允许的帧的最大大小
  # native_transport_max_frame_size_in_mb: 256
  # 使用本地传输时处理请求的最大线程数
  # native_transport_max_threads: 128
  # 本地传输最大并发线程数 -1默认 默认不限制
  # native_transport_max_concurrent_connections: -1
  # 每个ip本地传输最大并发线程数 -1默认 默认不限制
  # native_transport_max_concurrent_connections_per_ip: -1
  
  
  #是否启动 thrift rpc 服务器
  start_rpc: false
  rpc_address: localhost
  # rpc_interface_prefer_ipv6: false
  rpc_port: 9160
  # broadcast_rpc_address: 1.2.3.4
  rpc_keepalive: true
  # sync同步 hsha半同步半异步
  rpc_server_type: sync
  # rpc_min_threads: 16
  # rpc_max_threads: 2048
  # rpc_send_buff_size_in_bytes:
  # rpc_recv_buff_size_in_bytes:
  # internode_send_buff_size_in_bytes:
  # internode_recv_buff_size_in_bytes:
  thrift_framed_transport_size_in_mb: 15
  
  
  incremental_backups: false
  snapshot_before_compaction: false
  # 删除数据前是否自动快照
  auto_snapshot: true
  # 分区中行的排序规则索引的粒度
  column_index_size_in_kb: 64
  # 每个超过此大小的索引的索引缓存条目（上述内存中的排序规则索引）不会在堆上保留
  column_index_cache_size_in_kb: 2
  # 并行允许压缩的数量
  #concurrent_compactors: 1
  # 调节压缩到整个系统的给定总吞吐量
  compaction_throughput_mb_per_sec: 16
  # 
  sstable_preemptive_open_interval_in_mb: 50
  # stream_throughput_outbound_megabits_per_sec: 200
  # inter_dc_stream_throughput_outbound_megabits_per_sec: 200
  
  
  # 读超时
  read_request_timeout_in_ms: 5000
  # How long the coordinator should wait for seq or index scans to complete
  # 索引排序超时
  range_request_timeout_in_ms: 10000
  # 写超时
  # How long the coordinator should wait for writes to complete
  write_request_timeout_in_ms: 2000
  # 计数器超时
  # How long the coordinator should wait for counter writes to complete
  counter_write_request_timeout_in_ms: 5000
  # cas竞争超时
  # How long a coordinator should continue to retry a CAS operation
  # that contends with other proposals for the same row
  cas_contention_timeout_in_ms: 1000
  # 删除截断超时
  # How long the coordinator should wait for truncates to complete
  # (This can be much longer, because unless auto_snapshot is disabled
  # we need to flush first so we can snapshot before removing the data.)
  truncate_request_timeout_in_ms: 60000
  # 其他操作默认超时
  # The default timeout for other, miscellaneous operations
  request_timeout_in_ms: 10000
  # 节点间信息交换超时 默认false
  cross_node_timeout: false
  # 设置流的保持活动周期
  # streaming_keep_alive_period_in_secs: 300
  # 
  # phi_convict_threshold: 8
  # 
  endpoint_snitch: SimpleSnitch
  # 控制执行主机分数计算的较耗费资源部分的频率
  dynamic_snitch_update_interval_in_ms: 100
  # 控制重置所有主机分数的频率，允许坏主机可恢复
  dynamic_snitch_reset_interval_in_ms: 600000
  # 
  dynamic_snitch_badness_threshold: 0.1
  
  # 处理客户端请求的调度类：NoScheduler无选项，RoundRobin.throttle_limit限制请求数超过排队，RoundRobin.default_weight默认权重，RoundRobin.weights使用权重 （RoundRobinPolicy）
  request_scheduler: org.apache.cassandra.scheduler.NoScheduler
  # request_scheduler_options:
  #    throttle_limit: 80
  #    default_weight: 5
  #    weights:
  #      Keyspace1: 1
  #      Keyspace2: 5
  # 默认的keyspace
  # request_scheduler_id -- An identifier based on which to perform
  # the request scheduling. Currently the only valid option is keyspace.
  # request_scheduler_id: keyspace
  
  
  
  # server加密
  server_encryption_options:
      internode_encryption: none
      keystore: conf/.keystore
      keystore_password: cassandra
      truststore: conf/.truststore
      truststore_password: cassandra
      # More advanced defaults below:
      # protocol: TLS
      # algorithm: SunX509
      # store_type: JKS
      # cipher_suites: [TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_DHE_RSA_WITH_AES_128_CBC_SHA,TLS_DHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA]
      # require_client_auth: false
      # require_endpoint_verification: false
  
  # enable or disable client/server encryption.
  # 客户端加密
  client_encryption_options:
      enabled: false
      # If enabled and optional is set to true encrypted and unencrypted connections are handled.
      optional: false
      keystore: conf/.keystore
      keystore_password: cassandra
      # require_client_auth: false
      # Set trustore and truststore_password if require_client_auth is true
      # truststore: conf/.truststore
      # truststore_password: cassandra
      # More advanced defaults below:
      # protocol: TLS
      # algorithm: SunX509
      # store_type: JKS
      # cipher_suites: [TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_DHE_RSA_WITH_AES_128_CBC_SHA,TLS_DHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA]
  
  #  控制节点之间的流量是否被压缩。 可以是:all（所有），dc(dc之间压缩)，none(不压缩)
  internode_compression: dc
  # 
  inter_dc_tcp_nodelay: false
  # 
  # TTL for different trace types used during logging of the repair process.
  tracetype_query_ttl: 86400
  tracetype_repair_ttl: 604800
  
  #
  enable_user_defined_functions: false
  #
  enable_scripted_user_defined_functions: false
  # 
  enable_materialized_views: true
  # 
  windows_timer_interval: 1
  
  # 静态数据加密
  transparent_data_encryption_options:
      enabled: false
      chunk_length_kb: 64
      cipher: AES/CBC/PKCS5Padding
      key_alias: testing:1
      # CBC IV length for AES needs to be 16 bytes (which is also the default size)
      # iv_length: 16
      key_provider:
        - class_name: org.apache.cassandra.security.JKSKeyProvider
          parameters:
            - keystore: conf/.keystore
              keystore_password: cassandra
              store_type: JCEKS
              key_password: cassandra
    
    
  # 
  tombstone_warn_threshold: 1000
  tombstone_failure_threshold: 100000
  batch_size_warn_threshold_in_kb: 5
  batch_size_fail_threshold_in_kb: 50
  unlogged_batch_across_partitions_warn_threshold: 10
  compaction_large_partition_warning_threshold_mb: 100
  gc_warn_threshold_in_ms: 1000
  # SSTables 中的任何值的最大大小
  # max_value_size_in_mb: 256
  # 背压
  back_pressure_enabled: false
  #背压策略 看英文注释
  back_pressure_strategy:
      - class_name: org.apache.cassandra.net.RateBasedBackPressure
        parameters:
          - high_ratio: 0.90
            factor: 5
            flow: FAST
  
  ```

* 配置项

  ```
  cluster_name: 'xxx' #集群名
  data_file_directories: #数据目录
      - /data/cassandra/data
      
  commitlog_directory: /data/cassandra/commitlog #提交日志目录
  
  saved_caches_directory: /data/cassandra/saved_caches #缓存目录
  
  - seeds: "127.0.0.1" #集群种子节点 多个用，隔开
  
  listen_address: 127.0.0.1 #监听的ip或者主机的ip
  
  rpc_address: 127.0.0.1 #监听客户端链接地址，设置0.0.0.0的话放开broadcast_rpc_address注释
  ```

  

* 启动

  ```
  ./cassandra -R
  ```

* 报错

  问题1

  ```
  4691; No single argument constructor found for class [Ljava.lang.String;;  in 'reader', line 10, column 1:
      cluster_name: LASER-CLUSTER
  
  ```

  解决

  ```
  #配置数据文件目录
  data_file_directories:
  - /data/cassandra/data
  ```

  问题2

  ```
  Exception (java.lang.ClassCastException) encountered during startup: org.apache.cassandra.auth.PasswordAuthenticator cannot be cast to org.apache.cassandra.auth.IAuthorizer
  java.lang.ClassCastException: org.apache.cassandra.auth.PasswordAuthenticator cannot be cast to org.apache.cassandra.auth.IAuthorizer
  	at org.apache.cassandra.utils.FBUtilities.newAuthorizer(FBUtilities.java:473)
  	at org.apache.cassandra.auth.AuthConfig.applyAuth(AuthConfig.java:75)
  	at org.apache.cassandra.config.DatabaseDescriptor.daemonInitialization(DatabaseDescriptor.java:143)
  	at org.apache.cassandra.service.CassandraDaemon.applyConfig(CassandraDaemon.java:647)
  	at org.apache.cassandra.service.CassandraDaemon.activate(CassandraDaemon.java:582)
  	at org.apache.cassandra.service.CassandraDaemon.main(CassandraDaemon.java:691)
  ERROR [main] 2019-01-21 08:46:16,115 CassandraDaemon.java:708 - Exception encountered during startup
  java.lang.ClassCastException: org.apache.cassandra.auth.PasswordAuthenticator cannot be cast to org.apache.cassandra.auth.IAuthorizer
  	at org.apache.cassandra.utils.FBUtilities.newAuthorizer(FBUtilities.java:473) ~[apache-cassandra-3.11.3.jar:3.11.3]
  	at org.apache.cassandra.auth.AuthConfig.applyAuth(AuthConfig.java:75) ~[apache-cassandra-3.11.3.jar:3.11.3]
  	at org.apache.cassandra.config.DatabaseDescriptor.daemonInitialization(DatabaseDescriptor.java:143) ~[apache-cassandra-3.11.3.jar:3.11.3]
  	at org.apache.cassandra.service.CassandraDaemon.applyConfig(CassandraDaemon.java:647) [apache-cassandra-3.11.3.jar:3.11.3]
  	at org.apache.cassandra.service.CassandraDaemon.activate(CassandraDaemon.java:582) [apache-cassandra-3.11.3.jar:3.11.3]
  	at org.apache.cassandra.service.CassandraDaemon.main(CassandraDaemon.java:691) [apache-cassandra-3.11.3.jar:3.11.3]
  ```

  解决:照着注释改

  ```
  
  # Authentication backend, implementing IAuthenticator; used to identify users
  # Out of the box, Cassandra provides org.apache.cassandra.auth.{AllowAllAuthenticator,
  # PasswordAuthenticator}.
  #
  # - AllowAllAuthenticator performs no checks - set it to disable authentication.
  # - PasswordAuthenticator relies on username/password pairs to authenticate
  #   users. It keeps usernames and hashed passwords in system_auth.roles table.
  #   Please increase system_auth keyspace replication factor if you use this authenticator.
  #   If using PasswordAuthenticator, CassandraRoleManager must also be used (see below)
  #authenticator: AllowAllAuthenticator
  authenticator: PasswordAuthenticator
  # Authorization backend, implementing IAuthorizer; used to limit access/provide permissions
  # Out of the box, Cassandra provides org.apache.cassandra.auth.{AllowAllAuthorizer,
  # CassandraAuthorizer}.
  #
  # - AllowAllAuthorizer allows any action to any user - set it to disable authorization.
  # - CassandraAuthorizer stores permissions in system_auth.role_permissions table. Please
  #   increase system_auth keyspace replication factor if you use this authorizer.
  #authorizer: AllowAllAuthorizer
  authorizer: CassandraAuthorizer
  
  ```

#### 登录

```
./cqlsh -ucassandra -pcassandra
```

* 创建超级用户

  ```
  CREATE USER andy WITH PASSWORD '123456' SUPERUSER;   
  ```

* 创建普通用户

  ```
  CREATE USER test1 WITH PASSWORD '123456' NOSUPERUSER;   
  ```

* 修改用户

  ```
   ALTER USER test WITH PASSWORD '654321';
  ```

* 删除用户

  ```
  DROP USER cassandra; //不能删除当前登录的用户
  ```

  

**恭喜您！安装成功！下一篇讲java客户端与cqlsh。**