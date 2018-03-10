---
title: Mysql问题集锦
date: 2018-03-10 11:53:19
categories: [Mysql]
tags: [Mysql]
---

​	开发中遇到的mysql的问题的解决集锦<!--more-->

 *  The total number of locks exceeds the lock table size

    ```
    show variables like "%_buffer%";
    SET GLOBAL innodb_buffer_pool_size=67108864;
    ```

    ​