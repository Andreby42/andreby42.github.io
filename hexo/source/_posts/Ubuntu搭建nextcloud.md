---
title: Ubuntu搭建nextcloud
date: 2017-11-19 00:25:59
tags: [Linux]
categories: [Linux]
---

在linux服务器上搭建私有云<!--more-->



## 配置LAMP 环境

- 更新软件列表

  ```
  sudo apt update
  ```

- 执行软件更新

  ```
  sudo apt -y upgrade
  ```

- 安装apache

  ` sudo apt -y upgrade`  

- 安装MariaDB 

  `  sudo apt install mariadb-server`  


- 安装PHP7.0

  ` sudo apt install libapache2-mod-php7.0`  

- 安装PHP扩展 

  `php7.0-gd php7.0-json php7.0-mysql php7.0-curl php7.0-mbstringphp7.0-intl php7.0-mcrypt php-imagick php7.0-xml php7.0-zip ` 

## 安装 nextCloud

- 下载源码包 


- 访问官网下载 <https://nextcloud.com/changelog/> 当前最新版本 12.0.3

  ​ `wget https://download.nextcloud.com/server/releases/nextcloud-12.0.3.tar.bz2 `

- 解压源码包 

  ` 	tar jxf nextcloud-12.0.3.tar.bz2 ` 

- 复制源代码到web服务器目录

  `sudo cp -r nextcloud /var/www/`

- 设置nextcloud目录权限  

  ` sudo chown -R www-data:www-data /var/www/nextcloud`

## 配置apache虚拟主机

- 创建nexcloud.conf 虚拟主机配置文件


- 在域名管理平台新增一条A记录 指向服务器公网ip 

- 让虚拟机生效

  ` sudo nano /etc/apache2/sites-available/nextcloud.conf`

  将一下信息粘贴到配置文件

  ```xml
    ServerName d.spacexplore.xyz
      DocumentRoot /var/www/nextcloud/
      <Directory /var/www/nextcloud/>

        Options +FollowSymlinks
        AllowOverride All
        
        <IfModule mod_dav.c>
          Dav off
        </IfModule>
        
        SetEnv HOME /var/www/nextcloud
        SetEnv HTTP_HOME /var/www/nextcloud
      </Directory>
    RewriteEngine on
    RewriteCond %{SERVER_NAME} =d.spacexplore.xyz
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
    </VirtualHost>
  ```

- 让虚拟主机生效

  ```shell
   sudo a2ensite nextcloud.conf
  ```

- 启用必要的apache模块

  ```
  sudo a2enmod rewrite headers env dir mime ssl
  ```

- 重启 apache 服务器

  ```
   sudo service apache2 restart
  ```

## mariaDB数据库设置

* 数据库初始化 

  - 通过命令对数据库做安全初始化设置

    ```
    sudo mysql_secure_installation
    ```

  - 创建nextcloud 数据库账号密码

    ```
    sudo mysql -uroot;
    CREATE DATABASE 'nextcloud';
    CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'password';
    GRANT ALL PRIVILEGES ON nextcloud.* TO'nextcloud'@'localhost';
    flush privileges;
    exit;
    ```

* 初始化nextcloud

  - 在浏览器中访问绑定的域名进行nextcloud初始化操作,暂不记录

## 启用ssl安全连接

* 安装certbot

  - 添加 certbot PPA软件源 

    ```
    sudo add-apt-repository -y ppa:certbot/certbo
    ```

  - 刷新软件列表

    ```
    sudo apt update	
    ```

* 安装apache的certbot

  ```
  sudo apt install python-certbot-apache  
  ```


- 获取ssl证书

- 运行certbot程序 

  ```
  sudo certbot --apache
  ```

- 提示输入email接收通知和重置秘钥

  ```
   Enter email address (used for urgent renewal and security notices) (Enter 'c' to
   cancel):test@torchtree.com
  ```

- 询问是否接收用户协议

  ```
      Please read the Terms of Service at
      https://letsencrypt.org/documents/LE-SA-v1.1.1-August-1-2016.pdf. You must agree
      in order to register with the ACME server at
      https://acme-v01.api.letsencrypt.org/directory
      -------------------------------------------------------------------------------
      (A)gree/(C)ancel: A
  ```

- 询问是否愿意分享email 否

  ```
   	-------------------------------------------------------------------------------
      Would you be willing to share your email address with the Electronic Frontier
      Foundation, a founding partner of the Let's Encrypt project and the non-profit
      organization that develops Certbot? We'd like to send you email about EFF and
      our work to encrypt the web, protect its users and defend digital rights.
      -------------------------------------------------------------------------------
      (Y)es/(N)o: N
  ```

- 询问为哪个域名获取ssl证书 1

  ```
   	Which names would you like to activate HTTPS for?
      -------------------------------------------------------------------------------
      1: d.spacexplore.xyz
      -------------------------------------------------------------------------------
      Select the appropriate numbers separated by commas and/or spaces, or leave input
      blank to select all options shown (Enter 'c' to cancel):1
  ```

- 询问使用模式 secure 

  ```
      Please choose whether HTTPS access is required or optional.
      -------------------------------------------------------------------------------
      1: Easy - Allow both HTTP and HTTPS access to these sites
      2: Secure - Make all requests redirect to secure HTTPS access
      -------------------------------------------------------------------------------
      Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
  ```


## 修改owncloud 上传文件存储目录 

>  如上所示 已经创建好了nextcloud应用

> 那么已经创建好了应用,需要进行更改文件存储位置

* 停止web服务 即将apache服务器停止 

  ```
   service apache2 stop
  ```

*  查看config/config.php文件中已有的datadirectory 如(/var/www/nextcloud/data)

  ```
  root@SkyEyE:/var/www/nextcloud/config# vim config.php
  ```

    当然我这里配置文件已经修改了

  ```
  <?php
    $CONFIG = array (
      'instanceid' => 'oclvivd8451p',
      'passwordsalt' => 'kukoaeTPA0rbukretD7I0G1IIfIb0e',
      'secret' => 'QoVb9IEJPAom7kZTzMsIzXmAMGiMYciuZLEfDRUSX/rmC5xA',
      'trusted_domains' =>
      array (

        0 => 'd.spacexplore.xyz',
      ),
      'datadirectory' => '/mnt/bs/nextcloud/data',
      'overwrite.cli.url' => 'http://d.spacexplore.xyz',
      'dbtype' => 'mysql',
      'version' => '12.0.3.3',
      'dbname' => 'nextcloud',
      'dbhost' => 'localhost:3306',
      'dbport' => '',
      'dbtableprefix' => 'oc_',
      'dbuser' => 'XXXXXX',
      'dbpassword' => 'XXXXXXX',
      'installed' => true,
    );

  ```

*  修改 config/config,php 文件中的datadirectory (/xxx/xxx)

* 将/var/www/nextcloud/data 目录下的所有文件移动到新的 /mnt/bs/data/ 目录下

*  修改  /mnt/bs/data/目录所属的组及用户与原 /var/www/nextcloud/data 目录相同, 例如将所属组和用户都修改为www

  ```
  chown -R www-data:www-data /media/usbdisk/ocdata
  ```

*  启动apache服务器 进行管理界面查看 试上传一个文件 进入linux系统查看上传的文件是否在新的存储目录

  ```
  service apache2 start
  ```

  ​