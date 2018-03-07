---
title: Tomcat启动报Warn
date: 2017-12-05 22:21:15
tags: [Tomcat]
categories: [Tomcat]

---



​	Tomcat启动报内存溢出的Warn记录<!--more-->



​	最近在启动项目时候出现了This is very likely to create a memory leak的Warn错误

​	记录下解决方案(这里以1g内存为例):

​	常见的内存溢出大概有三种:

  *   OutOfMemoryError： Java heap space 

  *   OutOfMemoryError： PermGen space 

  *   OutOfMemoryError： unable to create new native thread. 

      对于前两种 在应用本身没有内存泄露的情况下 进行TomcatJvm参数配置来解决

       * -Xms -Xmx -XX：PermSize -XX：MaxPermSize

      最后一种需要进行调整操作系统和tomcat jvm参数同时调整

      ## 第一种:堆内存溢出

      Linux服务器下修改 bin目录下的catalina.sh

      添加

      ```
      JAVA_OPTS='-Xms512m -Xmx1024m' 
      ```

      或者

      ```
      JAVA_OPTS="-server -Xms800m -Xmx800m   -XX:MaxNewSize=256m" 
      ```

      或者

      ```
      CATALINA_OPTS="-server -Xms256m -Xmx300m"
      ```

      Windows服务器下 在catalina.bat最前面加入

      ```
      set JAVA_OPTS=-Xms128m -Xmx350m 
      ```

      或者

      ```
      set CATALINA_OPTS=-Xmx300M -Xms256M 
      ```

      这JAVA_OPTS和CATALINA_OPTS的区别在于前者直接设置jvm内存后者设置的是tomcat内存

      第二种解决方案

      Linux服务器

      ​	修改TOMCAT_HOME/bin/catalina.sh

      在“echo "Using CATALINA_BASE: $CATALINA_BASE"" 上面加入以下行

      ```
      JAVA_OPTS="$JAVA_OPTS -server -Xms800m -Xmx800m -XX:MaxNewSize=256m" 
      ```

       ##  永久保存区域溢出

​		PermGen space 的全称是 Permanent Generation space, 是指内存的永久保存区域

​		如果webapp下使用了大量的第三方jar,大小超过了jvm默认的大小4m 就会出现这种问题

​		解决办法:

​		Linux服务器下:

​		在catalina.sh的第一行增加: 

```
JAVA_OPTS= 
-Xms64m 
-Xmx256m 
-XX:PermSize=128M 
-XX:MaxNewSize=256m 
-XX:MaxPermSize=256m 
```

​		或在"“echo "Using CATALINA_BASE:   $CATALINA_BASE"”"上面加入以下行

```
JAVA_OPTS="-server -XX:PermSize=64M -XX:MaxPermSize=128m 
```

​		第三种是无法创建新的线程

​		

<!---->