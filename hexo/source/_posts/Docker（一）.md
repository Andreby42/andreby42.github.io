---
title: Docker（一）
date: 2018-11-02 20:11:10
tags: [Docker]
categories: [Linux]
---

Docker 一<!--more-->

#### Ubuntu Docker实战

* 安装

  安装Docker CE

  卸载老版本

  ```
  sudo apt-get remove docker docker-engine docker-ce docker.io
  ```

  更新apt

  ```
  sudo apt-get update
  sudo apt-get upgrade
  ```

  添加一些依赖

  ```
  sudo apt-get install  apt-transport-https ca-certificates  curl  software-properties-common
  ```

  添加docker官方的gpgkey

  ```
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  ```

  stable源添加

  ```
  sudo add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs) stable"
  ```

  ```
  apt-get update
  ```

* **开始安装DockerCE**

  ```
  sudo apt-get -y install docker-ce
  ```

* 配置镜像加速

* 概念简介

  * Cgroup: 资源控制

  * Namespace: 访问隔离

  * rootfs: 文件系统隔离

  * 容器引擎：生命周期控制

    

  

  

