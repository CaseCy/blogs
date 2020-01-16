---
title: " 在Linux下安装文件\t\t"
tags:
  - linux
url: 14.html
id: 14
comments: false
categories:
  - linux
date: 2018-10-18 16:15:21
---

安装Java
------

### 通过压缩文件安装

下载Linux下的Java安装包[Java](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 解压缩

    tar -xzvf jre-8u144-linux-x64.tar.gz
    

这种方式解压出来的文件直接就可以使用，只需要配置一下环境变量，在下面统一

### 通过rpm安装

下载rpm文件 安装

    rpm -ivh jre-8u144-linux-x64.rpm
    

这种安装方式，安装之后默认是在`/usr/java`目录下的

### 设置环境变量

    vi /etc/profile
    

在末尾加上下面的内容

    #Java
    JAVA_HOME=/usr/local/soft/jdk1.8.0_144
    CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    PATH=$JAVA_HOME/bin:$PATH
    export JAVA_HOME CLASSPATH PATH
    

这里的`JAVA_HOME`对应的是Java的安装目录 让配置马上生效

    source /etc/profile
    

验证：

    java -version
    

* * *

安装Tomcat
--------

下载Tomcat在Linux的安装包[Tomcat](http://tomcat.apache.org/) 放到`/usr/local/src`下，然后解压缩

    tar -xzvf redis-3.2.9.tar.gz
    

解压完之后就可以通过`bin/startup.sh`启动了，通过`bin/shutdown.sh`关闭

* * *

安装MySQL
-------

### 安装前准备

这里以安装MySQL5.6为例 访问[MySQL官网](https://dev.mysql.com/downloads/repo/yum/)下载yum版本的MySQL存储库，因为这里用的是Centos，所以下载的是红帽企业版 参照[快速安装指南](https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/) 将下载下来的存储库`mysql57-community-release-el6-11.noarch.rpm`上传到服务器之后，执行下列命令

    sudo rpm -Uvh mysql57-community-release-el6-11.noarch.rpm
    

执行了上面的命令之后就可以执行下面的命令查看可以安装的MySQL版本信息

    yum repolist all | grep mysql
    

得到结果如下：

    mysql-cluster-7.5-community        MySQL Cluster 7.5 Community   disabled
    mysql-cluster-7.5-community-source MySQL Cluster 7.5 Community - disabled
    mysql-cluster-7.6-community        MySQL Cluster 7.6 Community   disabled
    mysql-cluster-7.6-community-source MySQL Cluster 7.6 Community - disabled
    mysql-connectors-community         MySQL Connectors Community    enabled:     42
    mysql-connectors-community-source  MySQL Connectors Community -  disabled
    mysql-tools-community              MySQL Tools Community         enabled:     51
    mysql-tools-community-source       MySQL Tools Community - Sourc disabled
    mysql-tools-preview                MySQL Tools Preview           disabled
    mysql-tools-preview-source         MySQL Tools Preview - Source  disabled
    mysql55-community                  MySQL 5.5 Community Server    disabled
    mysql55-community-source           MySQL 5.5 Community Server -  disabled
    mysql56-community                  MySQL 5.6 Community Server    disabled
    mysql56-community-source           MySQL 5.6 Community Server -  disabled
    mysql57-community                  MySQL 5.7 Community Server    enabled:    201
    mysql57-community-source           MySQL 5.7 Community Server -  disabled
    mysql80-community                  MySQL 8.0 Community Server    disabled
    mysql80-community-source           MySQL 8.0 Community Server -  disabled
    

可以看到这里后面为enabled的是可以安装的，因为下载的是5.7的仓库，所以默认是5.7为可以安装的，如果要改为5.6可以使用下面的方法

### 更改MySQL安装版本

#### 使用yum工具类更改安装版本

执行下面的方法可以将5.7变为5.6，以此类推

    sudo yum-config-manager --disable mysql57-community
    sudo yum-config-manager --enable mysql56-community
    

如果提示`sudo: yum-config-manager: command not found`说明没有安装yum的工具包，先执行下面的命令：

    yum -y install yum-utils
    

安装工具包之后执行上面的命令就可以了

#### 手动编辑文件更改版本

还可以通过手动编辑`/etc/yum.repos.d/mysql-community.repo`文件来选择一个系列

    [mysql57-community]
    name=MySQL 5.7 Community Server
    baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/6/$basearch/
    enabled=1
    gpgcheck=1
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
    

找到要配置的子链接条目，然后编辑该enabled选项。指定enabled=0禁用子广告素材，或 enabled=1启用子广告素材。例如，要安装MySQL 5.6，请确保您具有 enabled=0上述用于MySQL5.7的子功能表项，并具有enabled=15.6系列的条目 验证是否更改成功

    yum repolist enabled | grep mysql
    

看到要更改的版本变为`enabled`,5.7变为`disabled`就表示更改成功 ###安装MySQL 执行下列命令就可以安装了

    sudo yum install mysql-community-server
    

通过`rpm -qa |grep mysql`可以看到下面的东西已经在系统中了

    mysql-community-libs-5.6.37-2.el6.x86_64
    mysql-community-libs-compat-5.6.37-2.el6.x86_64
    mysql57-community-release-el6-11.noarch
    mysql-community-common-5.6.37-2.el6.x86_64
    mysql-community-client-5.6.37-2.el6.x86_64
    mysql-community-server-5.6.37-2.el6.x86_64
    

### 启动MySQL服务

启动/关闭/重启/检查状态

    sudo service mysqld start/stop/restart/status
    

也可以不加sudo，如果是root用户的话 一开始的MySQL是没有密码的，可以直接使用`mysql -uroot -p`直接登录 针对5.6版本可以使用下面的命令来更改一些比较敏感的内容如root用户密码

    mysql_secure_installation
    

也可以参考这个[网址](http://www.cnblogs.com/liufei88866/p/5619215.html)设置root用户的密码 如果是5.7会生成一个初始密码，可以通过下面的命令查看

    sudo grep 'temporary password' /var/log/mysqld.log
    

* * *

开放端口
----

### 开放防火墙端口

这里的开放端口是更改防火墙设置，让linux的对应端口可以通过外网访问 比如Tomcat的8080端口，如果不开放端口，就只能通过内网访问

    vi /etc/sysconfig/iptables
    

可以看到只开放了22端口

    -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
    

遵照这个内容，在下面加上

    -A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
    

开放8080(Tomcat)，3306(MySQL)，80端口 修改之后重启防护墙

    /etc/init.d/iptables restart
    

查看端口是否开放

    /sbin/iptables -L -n
    

云服务器开放端口的方式不同，参照对应的云服务器

### 开放MySQL的访问权限

开启了外网访问权限之后，MySQL还是不能访问，因为MySQL自身有一个访问权限(默认为127.0.0.1即本地) 参照[MySQL授权](http://www.cnblogs.com/fslnet/p/3143344.html) 先登录MySQL 可以用下面的方法将所有的权限都赋给root

    grant all privileges on  *.*  to root@'%'  identified by 'xxxx';
    

这里的xxxx是用户的密码 执行完之后刷新权限

    flush privileges;
    

就可以使用外网连接数据库了

安装Nginx
-------

下载nginx的[安装包](http://nginx.org/en/download.html) 解压缩

    tar -zxvf nginx-1.14.0
    

安装必备的组件

    yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
    

配置要安装的位置，安装的组件

    ./configure --prefix=/usr/local/nginx \
    --with-http_ssl_module \
    --with-http_realip_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_secure_link_module \
    --with-http_v2_module \
    --with-http_stub_status_module \
    --with-http_sub_module \
    --with-http_ssl_module
    

然后编译

    make
    

安装

    make install
    

### 错误

如果遇到

    make: *** No rule to make target `build', needed by `default'.  Stop.
    

一般是因为上面的必备组件没有安装，执行`./configure`导致的，安装之后重新`./configure`再执行后面的操作就可以了

安装Nodejs(centos)
----------------

到nodejs[官网](https://nodejs.org/en/download/)下载linux源码 上传到服务器 解压缩

    tar -zxvf node-v8.11.4.tar.gz
    

切换到解压后的文件夹配置路径

    ./configure --prefix=/usr/local/nodejs_v8.11.4
    

编译安装

    make & make install
    

配置环境变量

    vi /etc/profile
    

加入

    export NODE_HOME=/usr/local/nodejs_v8.11.4
    export PATH=$NODE_HOME/bin:$PATH
    

更新环境变量

    source /etc/profile
    

验证是否成功

    node -v
    

### 将源替换为[淘宝源](http://npm.taobao.org/)

    npm config set registry https://registry.npm.taobao.org
    

使用淘宝定制的cnpm

    npm install -g cnpm --registry=https://registry.npm.taobao.org