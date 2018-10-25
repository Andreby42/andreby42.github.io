---
title: RabbitMq在Ubuntu中的安装配置
date: 2018-08-07 21:35:28
tags: [RabbitMq]
categories: [RabbitMq]
---

Ubuntu下安装Rabbitmq<!--more-->

windows安装比较麻烦，那么就在linux环境下安装个，ubuntu版本为16.04

#### 安装RabbitMq

* 使用apt-get进行安装

  ```
  sudo apt-get update
  sudo apt-get install rabbitmq-server
  ```

* 启动和停止服务

  ```
  service rabbitmq-server start
  service rabbitmq-server stop
  service rabbitmq-server status
  ```

* 查看当前用户列表

  * 查看用户列表

    ```
    rabbitmqctl list_users
    ```

  * 添加用户

    ```
    rabbitmqctl add_user skyeye nopassword
    ```

  * 删除用户

    ```
    rabbitmqctl delete_user skyeye
    ```

  * 修改密码

    ```
    rabbitmqctl change_password skyeye changepassword
    ```

  * 设置用户权限

    ```
    rabbitmqctl  set_permissions  -p  VHostPath  userName	  ConfP  WriteP  ReadP
    如：rabbitmqctl set_permissions -p "/" skyeye ".*" ".*" ".*"
    ```

  * 设置用户角色

    ```
    rabbitmqctl  set_user_tags  userName  RoleName
    ```

    

#### 开启RabbitMq页面控制台

* 进入安装目录

  * 查找目录（我也不知道安装到哪里了）

    ```
    find / -name rabbitmq
    /usr/share/rabbitmq
    /usr/lib/ocf/resource.d/rabbitmq
    /usr/lib/rabbitmq
    /etc/rabbitmq
    /var/lib/rabbitmq
    /var/log/rabbitmq
    ```

  * 进入目录

    ```
    cd /usr/lib/rabbitmq
    /usr/lib/rabbitmq
    root@VM-0-17-ubuntu:/usr/lib/rabbitmq# 
    ```

  * 查看已经安装的插件

    ```
    rabbitmq-plugins list
     Configured: E = explicitly enabled; e = implicitly enabled
     | Status:   * = running on rabbit@VM-0-17-ubuntu
     |/
    [  ] amqp_client                       3.5.7
    [  ] cowboy                            0.5.0-rmq3.5.7-git4b93c2d
    [  ] mochiweb                          2.7.0-rmq3.5.7-git680dba8
    [  ] rabbitmq_amqp1_0                  3.5.7
    [  ] rabbitmq_auth_backend_ldap        3.5.7
    [  ] rabbitmq_auth_mechanism_ssl       3.5.7
    [  ] rabbitmq_consistent_hash_exchange 3.5.7
    [  ] rabbitmq_federation               3.5.7
    [  ] rabbitmq_federation_management    3.5.7
    [  ] rabbitmq_management               3.5.7
    [  ] rabbitmq_management_agent         3.5.7
    [  ] rabbitmq_management_visualiser    3.5.7
    [  ] rabbitmq_mqtt                     3.5.7
    [  ] rabbitmq_shovel                   3.5.7
    [  ] rabbitmq_shovel_management        3.5.7
    [  ] rabbitmq_stomp                    3.5.7
    [  ] rabbitmq_test                     3.5.7
    [  ] rabbitmq_tracing                  3.5.7
    [  ] rabbitmq_web_dispatch             3.5.7
    [  ] rabbitmq_web_stomp                3.5.7
    [  ] rabbitmq_web_stomp_examples       3.5.7
    [  ] sockjs                            0.3.4-rmq3.5.7-git3132eb9
    [  ] webmachine                        1.10.3-rmq3.5.7-gite9359c7
    ```

  * 开启网页控制台

    ```
    rabbitmq-plugins enable rabbitmq_management
    ```

  * 重启 rabbitmq 服务

    ```
    service rabbitmq restart
    ```

  * 进入网页控制台

    ```
    http://x.x.x.x:15672/
    ```

  * 使用guest用户或skyeye用户无法登陆

    解决办法：

    ```
    //执行设置权限命令
    rabbitmqctl set_permissions -p / skyeye ".*" ".*" ".*"
    //执行设置角色命令
    rabbitmqctl set_user_tags skyeye administrator
    //设置完进行查看用户列表
     rabbitmqctl list_users
    ```

    设置完发现skyeye用户可以登陆但是guest用户依旧无法登陆

    进行google发现这么段话在大神的博客中：

    ```
    If rabbitmq.config doesn’t exist, it can be created manually. Set the RABBITMQ_CONFIG_FILEenvironment variable if you change the location. The Erlang runtime automatically appends the .config extension to the value of this variable
    ```

    意思大概是如果rabbitmq.config不存在，那么可以自己手动创建

    ```
    cd /etc/rabbitmq
    ll
    drwxr-xr-x  2 rabbitmq rabbitmq 4096 Aug  7 22:15 ./
    drwxr-xr-x 95 root     root     4096 Aug  7 21:33 ../
    -rw-r--r--  1 root     root       23 Aug  7 21:49 enabled_plugins
    -rw-r--r--  1 rabbitmq rabbitmq  570 Aug  7 22:15 rabbitmq-env.conf
    ```

    显然这么rabbitmq-env.conf 不是我们要的

    文档又说：有rabbitmq.config.example 这么个文件里面都是示例

    ```
    find / -name rabbitmq.config.example
    ```

    没找到

    这里有说明了：

    ```
    Example rabbitmq.config File
    RabbitMQ server source repository contains an example configuration file named rabbitmq.config.example. This example file contains an example of most of the configuration items you might want to set (with some very obscure ones omitted) along with documentation for those settings. All configuration items are commented out in the example, so you can uncomment what you need. Note that the example file is meant to be used as, well, example, and should not be treated as a general recommendation.
    
    In most distributions we place this example file in the same location as the real file should be placed (see above). However, for the Debian and RPM distributions policy forbids doing so; instead you can find it in /usr/share/doc/rabbitmq-server/ or /usr/share/doc/rabbitmq-server-3.6.10/ respectively.`
    ```

    ```
    cd /usr/shar/doc/rabbitmq-server
    ls
    changelog.Debian.gz  copyright  rabbitmq.config.example.gz
    gzip -d rabbitmq.config.example.gz
    ```

    这样示例就有了

    

    那么我们手动创建下

    ```
    cp rabbitmq.config.example  /usr/lib/rabbitmq/
    cd /usr/lib/rabbitmq/
    mv rabbitmq.config.example rabbitmq.config
    将 [<<“guest”>>] 修改为 []
    ```

    ```
    service rabbitmq-server stop
    service rabbitmq-server start
    ```

    还是不行 查询了下stackOverflow

    

[]: https://stackoverflow.com/questions/23669780/rabbitmq-3-3-1-can-not-login-with-guest-guest

​		就这样吧。回头研究下