---
title: FRP搭建使用
date: 2018-04-26 21:50:25
tags: [内网穿透]
categories: [Linux]
---

使用FRP进行内网穿透<!--more-->

* 首先要有一台拥有公网ip的服务器，和一台本地服务器

  * 在服务端搭建frpc环境,服务端为v家的小vps

    下载地址：``` https://github.com/fatedier/frp/releases/```

    ```
    wget https://github.com/fatedier/frp/releases/download/v0.17.0/frp_0.17.0_linux_amd64.tar.gz
    ```

    ```
    tar -zxvf frp_0.17.0_linux_amd64.tar.gz
    ```

    ```
    cd frp_0.17.0_linux_amd64
    ```

    ```
    外网作为服务端 无需客户端配置故可以删除
    rm -rf frpc.ini
    rm -rf frpc
    ```

    ```
    修改配置
    vim frps.ini

    bind_port 为服务绑定的端口
    [common]
    bind_port = 7000
    ```

    ```
    后台启动
    nohup ./frps -c ./frps.ini &
    ```

  * 在客户端就是本机环境搭建客户端环境 frps,客户端为centos 7x64

    ```
    wget https://github.com/fatedier/frp/releases/download/v0.17.0/frp_0.17.0_linux_amd64.tar.gz
    ```

    ```
    tar -zxvf frp_0.17.0_linux_amd64.tar.gz
    ```

    ```
    cd frp_0.17.0_linux_amd64
    客户端删掉服务端配置

    rm -rf frps

    rm -rf frps.ini
    ```

    ####重点来了####

    ```
    vim  frpc.ini

    [common]
    server_addr = x.x.x.x
    server_port = 7000

    [web]
    type = http
    local_port = 80
    custom_domains = www.yourdomain.com

    [ssh]
    type = tcp
    local_ip = 127.0.0.1
    local_port = 22
    remote_port = 6000

    server_addr 填写服务端的公网ip地址
    server_port 对应服务端的bind_port
    	
    web项
    type 填写http
    local_port填写80
    custom_domian填写外网域名或者外网ip即公网ip

    #ssh项 用来做通过ssh链接客户端 暂不介绍 官方仓库有

    	
    ```

    ```
    启动
    nohup ./frpc -c ./frpc.ini &
    ```

  * 设置开机启动

    ```
    vim /etc/systemd/system/frps.service
    ```

    ```
    编辑该文件
    [Unit]
    Description=frps daemon
    After=syslog.target  network.target
    Wants=network.target

    [Service]
    Type=simple
    ExecStart=/data/soft/frp_0.17.0_linux_amd64/frpc -c /data/soft/frp_0.17.0_linux_amd64/frpc.ini
    Restart= always
    RestartSec=1min
    ExecStop=/usr/bin/killall frpc

    [Install]
    WantedBy=multi-user.target
    ```

    ```
    $ sudo systemctl start frps
    $ sudo systemctl enable frps
    ```

  * 那么现在就可以通过外网ip访问本地的web应用了

    ​

    []: https://github.com/fatedier/frp

    ​