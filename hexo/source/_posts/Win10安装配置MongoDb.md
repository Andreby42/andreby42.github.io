---
title: Win10安装配置MongoDb
date: 2018-03-10 12:30:04
categories: [MongoDB]
tags: [MongoDB]
---

​	Win10下安装配置MongoDb的记录,省的用的时候找不到<!--more-->

 *  下载安装mongdodb,使用版本为3.6.3

    ```
    https://downloads.mongodb.com/win32/mongodb-win32-x86_64-enterprise-windows-64-3.6.3-signed.msi?_ga=2.244974149.1141576444.1520604797-212568712.1520604797
    ```

* 安装完成后目录为：D:\Tools\Mongo3.6.3

  在Mongo3.6.3下创建data文件夹，

  data文件夹下创建db和log两个文件夹，

  其中log文件夹中创建一个mongodb.log

  在Mongo3.6.3 目录下创建mongodb.config 内容如下

  ```
  dbpath=D:\Tools\Mongo3.6.3\data\db
  logpath=D:\Tools\Mongo3.6.3\data\log\mongodb.log
  logappend=true
  auth=false
  ```

* 编辑环境变量

  将D:\Tools\Mongo3.6.3\bin 加入到PATH变量中

* 注册为windows服务

  按WIN+X进入菜单，选择以管理员方式的powershell，进入D:\Tools\Mongo3.6.3\bin目录，

  执行命令：

  ```
  .\mongod --config D:\Tools\Mongo3.6.3\mongodb.config --serviceName MongoDB --install
  ```

  执行完毕 按WIN+R输入services.msc 查看MongoDB是否存在，

  在powershell中执行命令

  ```
  PS D:\Tools\Mongo3.6.3\bin> net start MongoDB
  MongoDB 服务正在启动 .
  MongoDB 服务已经启动成功。
  PS D:\Tools\Mongo3.6.3\bin>
  ```

  大功告成！

*  配置mongodb用户密码

    进入powershell 输入mogo登陆到mongodb

    ```
    use admin
    db.createUser({user:"andy",pwd:"123456",roles:[{role:"root",db:"admin"}]})
    ```

    如果想要开启验证那么就把config文件中的auth=false改为true，就开启密码验证了，修改完config中的内容需要进行重启服务

    ​

    ​