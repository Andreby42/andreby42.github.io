---
title: CentOs7安装Docker
date: 2018-03-09 21:44:07
categories: [Linux]
tags: [Linux]
---

CentOs7安装docker记录<!--more-->

> 记录docker在centOs7的安装,以centos7minimal 1708为例

### 安装

 *  ```
    sudo yum install -y yum-utils device-mapper-persistent-data lvm2
    ```

 *  ```
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    ```

* ```
  sudo yum makecache fast

  ```

* ```
  sudo yum install -y docker-ce
  ```

  ​