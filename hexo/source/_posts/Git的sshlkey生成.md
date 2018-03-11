---
title: Git的sshlkey生成
date: 2018-03-10 13:44:25
tags: [Git]
categories: [Git]
---

​	git生成sshkey 省的忘<!--more-->

 * git config --global user.name "github名"
 * git config --global user.email "邮箱地址"
 * ssh-keygen -t rsa -C "邮箱地址"  