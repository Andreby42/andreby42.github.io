---
title: tomcat-native和APR
date: 2018-02-26 00:48:53
tags: [Tomcat]
categories: [Tomcat]
---

Tomcat开启Apr及Tomcat-native提高性能<!--more-->

### 安装JDK及tomcat8

tomcat8原生支持nio 这里以8.0.50版本为例,jdk以1.7为例,linux系统以CentOS-7-x86_64-Minimal-1708为例

```JAVA_HOME=/data/soft/jdk1.7.0_80
JAVA_HOME=/data/soft/jdk1.7.0_80
PATH=JAVA_HOME/bin:PATH
CLASSPATH=.:JAVA_HOME/lib/dt.jar:JAVA_HOME/lib/tools.jar
export JAVA_HOME
export PATH
export CLASSPATH
```

### 查看Openssl并更新

* 查看openssl版本

  ```
  openssl version
  ```

  如果版本低于1.0.2那么需要进行升级

* 升级openssl

  这里就升级为最新版

  The latest stable version is the 1.1.0 series 这是官方的note lts版本为1.0.2版本

  ```
  wget https://www.openssl.org/source/openssl-1.1.0g.tar.gz
  tar -zxvf openssl-1.1.0g.tar.gz
  cd openssl-1.1.0g
  ./config --prefix=/usr/local/openssl
  make && make install    
  //进行备份
  mv /usr/bin/openssl /usr/bin/openssl.bak
  mv /usr/include/openssl /usr/include/openssl.bak
  //做新的软连接
  ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl    
  ln -s /usr/local/openssl/include/openssl /usr/include/openssl

  echo "/usr/local/openssl/lib">>/etc/ld.so.conf    
  ldconfig -v 
  openssl version
  ```

  ​

### 安装APR

```
yum install apr-devel openssl-devel  
```

如果你的tomcat版本高的话 在安装native 时候会提示你你的apr版本必须高于1.46还是多少 但是yum源的版本又过低

这时你需要进行手动下载apr 并进行安装

```
yum -y install libtool
yum -y install autoconf
wget http://archive.apache.org/dist/apr/apr-1.6.3.tar.gz
cd apr-1.6.3.tar.gz
./configure --prefix=/usr/apr
make
make install
```



### 安装编译native

 *  进入tomcat的bin目录下

    ```
    cd /data/soft/tomcat-8.0.50/bin/
    ```

* 解压tomcat-native

  ```
  tar -zxvf tomcat-native-1.2.16-src/		
  ```

* 进行编译安装

  ```
  ./configure --with-apr=/usr/bin/apr-1-config --with-java-home=/data/soft/jdk1.7.0_80 --with-ssl=yes
  ```

  ```
  make && make install
  ```

  安装完成后会提示:

  ```
  Libraries have been installed in: 
  /usr/local/apr/lib
  ```

* 修改环境变量

  ```
  export LD_LIBRARY_PATH=/usr/local/apr/lib 
  ```

  ```
  source /etc/profile
  ```

* 修改tomcat环境变量

  在tomat的bin目录下的catalina.sh中添加:

  ```
  CATALINA_OPTS="-Djava.library.path=/usr/local/apr/lib"
  ```

  加在文件最前面 加在JAVA_OPTS后

* 修改server.xml

  把原来的配置需要进行注释或者修改

  ```
  <!--enableLookups=”false” 来关闭 DNS 反向查询提升性能-->
    <Connector executor="tomcatThreadPool"
                 port="8080" protocol="org.apache.coyote.http11.Http11AprProtocol"
                 URIEncoding="UTF-8" enableLookups="false" acceptCount="50"
                 connectionTimeout="1000" maxKeepAliveRequests="250"
                 redirectPort="8443" />
                 
      <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
          maxThreads="500" minSpareThreads="25"
          maxIdleTime="4000"/>
          <!--如果用不到SSL，则需要关闭，on改为off,否则启动时会报错-->
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="off" /> 
  ```

  具体的参数调优可参照之前

### 启动并查看日志

当日志中打印:

```
26-Feb-2018 08:44:16.121 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-apr-8080"]
26-Feb-2018 08:44:16.155 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["ajp-apr-8009"]
26-Feb-2018 08:44:16.184 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in 4630 ms
```

表示APR已经起作用了!