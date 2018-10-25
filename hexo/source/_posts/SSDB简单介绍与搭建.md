---
title: SSDB简单介绍与搭建
date: 2018-09-29 20:32:08
tags: [NoSql]
categories: [Nosql]
---

ssdb了解下<!--more-->

一个高性能的支持丰富数据结构的 NoSQL 数据库, 用于替代 Redis.与redis不同，基于硬盘存储，redis可以无缝迁移至ssdb

### 搭建

* 下载

  ```
  wget --no-check-certificate https://github.com/ideawu/ssdb/archive/master.zip
  unzip master
  cd ssdb-master
  make
  # optional, install ssdb in /usr/local/ssdb
  sudo make install
  ```

  我们在make的时候会报错

  ```
  
  ERROR! autoconf required! install autoconf first
  
  Makefile:4: build_config.mk: No such file or directory
  make: *** No rule to make target 'build_config.mk'.  Stop.
  ```

  那么我们先下载安装autoconf，然后再make就行了

  ```
  apt-get install autoconf -y
  ```

* 安装

  如果想把ssdb安装在别的目录 那么在ssdb的根目录下执行

  ```
  sudo make install PREFIX=/data/soft/ssdb
  ```

* 启动 & 为后台运行 -d 也可以后台运行

  ```
  cd /data/soft/ssdb
  ./ssdb-server ssdb.conf &
  ssdb-server 1.9.6
  Copyright (c) 2012-2015 ssdb.io
  ```

  ```
  ./ssdb-server ssdb.conf -d
  ```

  重启

  ```
  ./ssdb-server ssdb.conf -s -d restart 
  ```

* 设置随机启动

  先找到ssdb.sh

  ```
   find / -name ssdb.sh
   /data/soft/ssdb-master/tools
  ```

  

  根据官方文档，将ssdb.sh放置在./etc/init.d下

  ```
  cp /data/soft/ssdb-master/tools /etc/init.d
  ```

  修改ssdb.sh将以下改为你的ssdb.conf的位置

  ```
  configs="/data/soft/ssdb/ssdb.conf"
  ```

  官方文档的说明,关于集群的设置

  ```
  Change /data/ssdb_data/test/ssdb.conf to the location of your SSDB config file. If you have more than one SSDB instances, put all config files in one line, separated by spaces:
  # each config file for one instance
  configs=/data/ssdb_data/test/ssdb.conf /data/ssdb_data/demo/ssdb.conf
  ```

  添加权限

  ```
  sudo chmod a+x /etc/init.d/ssdb
  sudo update-rc.d ssdb defaults
  ```

* 配置修改

  * 监听网络端口,在ssdb.conf中，注意**配置文件中的缩进使用tab**，我这里已经修改为0.0.0.0,端口修改为7379，0.0.0.0意思为所有的ip都可以进行访问

    可以利用allow|denys 来控制ip的访问，如下面的注释里所讲

    ```
    server:
              ip: 0.0.0.0
              port: 7379
              # bind to public ip
              #ip: 0.0.0.0
              # format: allow|deny: all|ip_prefix
              # multiple allows or denys is supported
              #deny: all
              #allow: 127.0.0.1
              #allow: 192.168
              # auth password must be at least 32 characters
              #auth: very-strong-password
              #readonly: yes
              # in ms, to log slowlog with WARN level
              #slowlog_timeout: 5
    
    ```

* Redis迁移到ssdb

  在 `tools` 目录中的 `redis-import.php` PHP 脚本可以用来将 Redis 服务器上的数据, 拷贝到 SSDB 服务器上.

  ```
  php redis-import.php redis_host redis_port redis_db ssdb_host ssdb_port
  ```

  **请确保你的 PHP Redis 模块 <https://github.com/nicolasff/phpredis> 已经安装.**

  TODO:先写到这里，明天写下代码客户端，这个东西可玩度很高，类似的产品还有360的pika...

   

[]: http://ssdb.io/docs/zh_cn/index.html	"ssdb-Wiki"

