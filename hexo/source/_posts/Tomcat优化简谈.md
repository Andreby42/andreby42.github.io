---
title: Tomcat优化简谈
date: 2017-11-22 22:29:39
tags: [Linux,Tomcat]
categories: [Linux,Tomcat]
---

简单谈下Tomcat的配置优化<!--more-->

> 网上流传的tomcat优化配置,记录下

- 配置tomcat管理员账户

  这里的tomcat 就以虚拟机上的tomcat 为例,版本为8.0.45

  在conf/tomcat-users.xml下添加用户

  tomcat 目录如下:

  ```
  drwxr-xr-x. 2 root root  4096 8月   9 04:10 bin
  drwxr-xr-x. 2 root root  4096 6月  27 04:09 conf
  drwxr-xr-x. 2 root root  4096 8月   9 04:10 lib
  -rw-r--r--. 1 root root 57011 6月  27 04:09 LICENSE
  drwxr-xr-x. 2 root root     6 6月  27 04:06 logs
  -rw-r--r--. 1 root root  1444 6月  27 04:09 NOTICE
  -rw-r--r--. 1 root root  6741 6月  27 04:09 RELEASE-NOTES
  -rw-r--r--. 1 root root 16195 6月  27 04:09 RUNNING.txt
  drwxr-xr-x. 2 root root    29 8月   9 04:10 temp
  drwxr-xr-x. 7 root root    76 6月  27 04:08 webapps
  drwxr-xr-x. 2 root root     6 6月  27 04:06 work
  ```

  ```
  /usr/local/soft/tomcat-common/conf
  [root@localhost conf]# ll
  总用量 216
  -rw-------. 1 root root  13688 6月  27 04:09 catalina.policy
  -rw-------. 1 root root   7299 6月  27 04:09 catalina.properties
  -rw-------. 1 root root   1577 6月  27 04:09 context.xml
  -rw-------. 1 root root   3387 6月  27 04:09 logging.properties
  -rw-------. 1 root root   6458 6月  27 04:09 server.xml
  -rw-------. 1 root root   2164 6月  27 04:09 tomcat-users.xml
  -rw-------. 1 root root   2634 6月  27 04:09 tomcat-users.xsd
  -rw-------. 1 root root 168496 6月  27 04:09 web.xml
  ```

  ```
   <?xml version='1.0' encoding='utf-8'?>
     <!--
       Licensed to the Apache Software Foundation (ASF) under one or more
       contributor license agreements.  See the NOTICE file distributed with
       this work for additional information regarding copyright ownership.
       The ASF licenses this file to You under the Apache License, Version 2.0
       (the "License"); you may not use this file except in compliance with
      the License.  You may obtain a copy of the License at
    
          http://www.apache.org/licenses/LICENSE-2.0
    
      Unless required by applicable law or agreed to in writing, software
      distributed under the License is distributed on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
      See the License for the specific language governing permissions and
      limitations under the License.
    -->
    <tomcat-users xmlns="http://tomcat.apache.org/xml"
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
                  version="1.0">
    <!--
      NOTE:  By default, no user is included in the "manager-gui" role required
      to operate the "/manager/html" web application.  If you wish to use this app,
      you must define such a user - the username and password are arbitrary. It is
      strongly recommended that you do NOT use one of the users in the commented out
      section below since they are intended for use with the examples web
      application.
    -->
    <!--
      NOTE:  The sample user and role entries below are intended for use with the
      examples web application. They are wrapped in a comment and thus are ignored
      when reading this file. If you wish to configure these users for use with the
      examples web application, do not forget to remove the <!.. ..> that surrounds
      them. You will also need to set the passwords to something appropriate.
    -->
    <!--
      <role rolename="tomcat"/>
      <role rolename="role1"/>
      <user username="tomcat" password="<must-be-changed>" roles="tomcat"/>
      <user username="both" password="<must-be-changed>" roles="tomcat,role1"/>
      <user username="role1" password="<must-be-changed>" roles="role1"/>
    -->
    </tomcat-users>
  ```

  ```
  <role rolename="manager"/>
  <role rolename="manager-gui"/>
  <role rolename="admin"/>
  <role rolename="admin-gui"/>
  <user username="admin" password="tomcat" roles="admin-gui,admin,manager-gui,manager"/>

  ```

  现在启动tomcat ,登录查看信息

  进入界面后点击右侧**ManagerApp**  **HostManager ** **ServerStatus**


- tomcat 的三种运行模式

  - bio

    默认的模式,性能非常低下,没有经过任何优化处理和支持

  - nio

    nio(new I/O),是Java SE 1.4 及 后续版本提供的一种新的I/O操作方式(即java.nio包及其子包).

    java nio是一个基于缓冲区 并能提供非阻塞I/O操作的JavaApi,因此nio也被看成是non-blockingI/O的

    缩写.它拥有比传统I/O操作(bio)更好的并发运行性能

  - apr

    安装起来最困难,但是从操作系统级别来解决异步的IO问题,大幅度的提高性能

- 启动NIO模式

  - 修改server.xml里的Connector节点,修改protocol为org.apache.coyote.http11.Http11NioProtocol 

    ```
    [root@localhost tomcat-common]# cd conf
    [root@localhost conf]# vim server.xml
    ```

    修改前

    ```
      <Connector port="8080" protocol="HTTP/1.1"
                    connectionTimeout="20000"
                     redirectPort="8443" />
    ```

    修改后

    ```
    <Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol"
                     connectionTimeout="20000"
                     redirectPort="8443" />
    ```

- 执行器(线程池)

  - 开启线程池

    配置,解开这段注释

    ```
    <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
            maxThreads="150" minSpareThreads="4"/>
    ```

    修改Connector节点

    ```
    <Connector port="8080" executor="tomcatThreadPool" protocol="org.apache.coyote.http11.Http11NioProtocol"
                    connectionTimeout="20000"
                    redirectPort="8443" />
    ```

    重新启动

  - 参数说明

    ![](.Tomcat优化简谈/参数说明.png)

  - 最佳实践

    ```
    <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
            maxThreads="800" minSpareThreads="100" maxQueueSize="100" prestartminSpareThreads="true"/>
    ```

- 连接器(Connector)

  Connector 是Tomcat接收请求的入口,每个Connector有自己专属的监听端口

  Connector 有两种:HTTP Connector 和AJP Connector

  - 连接器的通用属性(注意高亮部分)

    |          属性           | 描述                                       |
    | :-------------------: | :--------------------------------------- |
    |      allowTrace       | A boolean value whichcan be used to enable or disable the TRACE HTTP method. If not specified, this attribute is set to false.                       如果需要服务器能够处理用户的HAED/TRACE请求，这个值应该设置为true，默认值是false |
    |     asyncTimeOut      | The default timeout forasynchronous requests in milliseconds. If not specified, this attribute is setto 10000 (10 seconds).                                             默认超不时候以毫秒为单位的异步恳求。若是没有指定，该属性被设置为10000（10秒） |
    |   **enableLookUps**   | Set to true if you want calls to request.getRemoteHost() to perform DNS lookups in orderto return the actual host name of the remote client. Set to false to skip the DNS lookup and returnthe IP address in String form instead (thereby improving performance). Bydefault, DNS lookups are disabled.若是你想request.getRemoteHost（）的调用 履行，以便返回的长途客户端的实际主机名的DNS查询，则设置为true。设置为false时跳过DNS查找，并返回字符串情势的IP地址（从而提高性能）。默认景象下，禁用DNS查找 |
    |    maxHeaderCount     | The maximum number of headers in a request that are allowed by the container. Arequest that contains more headers than the specified limit will be rejected. Avalue of less than 0 means no limit. If not specified, a default of 100 is used.容器允许的请求头字段的最大数目。请求中包含比指定的限制更多的头字段将被拒绝。值小于0表示没有限制。如果没有指定，默认设置为100 |
    |   maxparameterCount   | The maximum number of parameter and value pairs (GET plus POST) which will beautomatically parsed by the container. Parameter and value pairs beyond thislimit will be ignored. A value of less than 0 means no limit. If not specified,a default of 10000 is used. Note that FailedRequestFilter [filter](http://127.0.0.1:8080/docs/config/filter.html)can be used to reject requests that hit thelimit.将被容器自动解析的最大数量的参数和值对（GET加上POST）。参数值对超出此限制将被忽略。值小于0表示没有限制。如果没有指定，默认为10000。请注意， FailedRequestFilter 过滤器可以用来拒绝达到了极限值的请求 |
    |    **maxPostSize**    | 将被容器以FORMURL参数形式处理的最大长度（以字节为单位）的POST。通过设置此属性的值小于或等于0可以禁用该限制。如果没有指定，该属性被设置为2097152（2兆字节） |
    |    maxSavePostSize    | 将被容器在FORM或CLIENT-CERT认证中保存/缓冲的POST的最大尺寸（以字节为单位）。对于这两种类型的身份验证，在用户身份验证之 前，POST将被保存/缓冲。对于POST CLIENT-CERT认证，处理该请求的SSL握手和缓冲清空期间，POST将被缓存。对于Form认证，POST将被保存，同时用户将被重定向到登陆 表单。POST将被一直保留直到用户成功认证或者认证请求关联的会话超时。将此属性设置为-1可以禁用此限制。将此属性设置为0，POST数据在身份验证 过程中将不被保存。如果没有指定，该属性设置为4096(4千字节) |
    |   parseBodyMethods    | 以逗号分隔的HTTP方法列表，通过方法列表，等同于POST方法，request 正文将被解析成请求参数。这在RESTful应用程序要支持以POST式的语义解析PUT请求中是非常有用的。需要注意的是设置其他值（不是POST）会导致Tomcat的行为违反servlet规范的目的。在这里为了符合HTTP规范明确禁止HTTP方法TRACE。默认值是POST |
    |       **port**        | TheTCP port number on which this **Connector** will create a server socket and awaitincoming connections. Your operating system will allow only one serverapplication to listen to a particular port number on a particular IP address.If the special value of 0 (zero) is used, then Tomcat will select a free portat random to use for this connector. This is typically only useful in embeddedand testing applications.TCP端口号，连接器利用该端口号将创建一个服务器套接字，并等待传入的连接。你的操作系统将只允许一个服务器应用程序在一个特定的IP地址侦听特定的端口号。如果使用特殊值0（零），则Tomcat将为连接器随机选择一个空闲的端口。这是通常只用在嵌入式和测试应用程序 |
    |     **protocol**      | 设置协议来处理传入流量。默认值是 HTTP/1.1，将使用自动切换机制来选择阻塞的基于Java的连接器或APR /native 为基础的连接器。                      如果PATH（Windows）或LD_LIBRARY_PATH（在大多数Unix系统）的环境变量包含在Tomcat的本地库里，APR /native 连接器将被使用。如果在本地库中无法找到，阻断基于Java的连接器将被使用。需要注意的是使用HTTPS比Java连接器与APR /native 连接器有不同的设置。一个明确的协议，而不是依靠上述自动切换机构，可用以下值:  org.apache.coyote.http11.Http11Protocol -阻塞式的Java连接器,  org.apache.coyote.http11.Http11NioProtocol -不阻塞Java连接器, org.apache.coyote.http11.Http11AprProtocol的 -的APR / native 连接器 |
    |       proxyName       | 如果这个连接正在使用的代理服务器配置，配置该属性指定的服务器的名称，可以调用request.getServerName（）返回。有关更多信息，请参见代理支持. |
    |       proxyPort       | 如果这个连接正在使用的代理服务器配置，配置该属性指定服务器端口，可以调用request.getServerPort（）返回。有关更多信息，请参见代理支持. |
    |     redirectPort      | 如果该连接器支持非SSL请求，并且接收到的请求为满足安全约束需要SSL传输， Catalina 将自动将请求重定向到指定的端口号 |
    |        scheme         | 如果你想调用request.isSecure（）收到此连接器的请求返回true，请该该属性设置为true。您希望SSL连接器或非SSL连接器接收数据通过一个SSL加速器，像加密卡，SSL设备，甚至一个web服务器。默认值是false |
    |    **URIEncoding**    | 这将指定使用的字符编码，来解码URI字符。如果没有指定，ISO-8859-1将被使用 |
    | useBodyEncodignForURI | 这指定是否应该用于URI查询参数，而不是使用URIEncoding contentType中指定的编码。此设置兼容性Tomcat 4.1.x版（该版在contentType中指定编码，或者使用request.setCharacterEncoding的方法显式设置（参数为 URL传来的值）。默认值false。 |
    |      xpoweredBy       | 将此属性设置为true会导致Tomcat支持使用Servlet规范的通知，（在规范中推荐使用头字段）。默认值是假的 |
    |      useIPVHosts      | 将该属性设置为true会导致Tomcat使用收到请求的IP地址，来确定将请求发送到哪个主机。默认值是false |

    **ToBeContinue**

