---
title: " Redis集群\t\t"
tags:
  - redis
url: 107.html
id: 107
comments: false
categories:
  - redis
date: 2018-10-19 09:36:25
---

tags： Redis

* * *

[ruby1](https://ruby-china.org/wiki/rvm-guide) [ruby更新](http://www.jianshu.com/p/8169f5d7f364)

集群准备
----

Redis的安装不赘述，参考Linux下安装Redis 集群的官方参考文档[Redis cluster](https://redis.io/topics/cluster-tutorial) 准备一个redis配置文件redis.conf 修改下面的内容

    daemonize yes
    cluster-enabled yes
    cluster-config-file nodes.conf
    cluster-node-timeout 5000
    appendonly yes
    

`time-out`为参考 创建一个文件夹，作为Redis集群

    mkdir redis_cluster
    

创建几个文件夹，作为单个redis文件存放地，这里使用端口7000-7005作为redis的集群端口

    mkdir redis_700{0,1,2,3,4,5}
    

将上面的配置文件复制到上面单个redis的文件夹，并修改redis.conf中的`port`为对应端口，不赘述 分别切换到对应的端口文件夹

    redis-server redis_7000/1/2/3/4/5.conf
    

启动每个redis 使用`ps -ef |grep redis`查看会看到

    root     17703     1  0         00:00:19 redis-server 0.0.0.0:7000 [cluster]
    root     17708     1  0         00:00:19 redis-server 0.0.0.0:7001 [cluster]
    root     17713     1  0         00:00:19 redis-server 0.0.0.0:7002 [cluster]
    root     17720     1  0         00:00:19 redis-server 0.0.0.0:7003 [cluster]
    root     17730     1  0         00:00:19 redis-server 0.0.0.0:7004 [cluster]
    root     17735     1  0         00:00:19 redis-server 0.0.0.0:7005 [cluster]
    

redis都启动起来了 启动集群，切换到redis的安装目录下的`src`文件夹 执行

    ./redis-trib.rb  create --replicas 1 127.0.0.1.41:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
    

注：如果是用外网访问redis的话一定要注意下面几点

*   端口开放 除了开放上面集群的端口，外网访问的时候还要开放集群端口+10000端口，如开放了7000端口，还要开放17000端口
*   创建用的端口最好是服务器的公网ip，如果这里用127.0.0.1创建集群，会出现JedisCluster连接不上的情况，如果使用公网ip不开放+10000端口会出现`Waiting for the cluster to join...`一直阻塞的情况
*   配置文件中的bind属性最好注释掉，这个属性是指定可以访问redis的ip的，需要外网访问的时候要注释掉
*   配置文件中的`protected-mode no`保护模式可以关掉，不关掉会提示错误，这里如果bind了ip和设置了密码都可以忽略这个选项

如果提示需要ruby，执行下面的命令

    yum install ruby
    yum install rubygems
    

Ruby安装和更新
---------

使用上面的方式安装之后会发现ruby的版本为1.8.7，执行

    gem install  redis
    

会提示 提示执行上面的步骤需要ruby2.2.2的版本，这里可以通过ruby的版本管理工具rvm来升级，也可以去官网下gz包去，配置环境变量指定 安装之前先将yum中的ruby卸载

    yum remove -y ruby
    

访问[rvm官网](https://rvm.io/) 执行

    gpg --keyserver hkp：//keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
    

这个是官方提供的用来验证的公钥，以保证安全。

    \curl -sSL https://get.rvm.io | bash -s stable
    

安装稳定版本的rvm，会在看到下面的语句。

    * To start using RVM you need to run `source /etc/profile.d/rvm.sh`
        in all your open shell windows, in rare cases you need to reopen all shell windows.
    

执行上面的`source /etc/profile.d/rvm.sh`刷新就可以使用rvm命令了 列出已知版本

    rvm list known
    

安装ruby2.3

    rvm install 2.3
    

rvm支持多个版本的ruby同时在一个系统上，使用

    rvm use 2.3 或者 rvm use 2.3 --default 后面这个设置为默认
    

切换版本 使用

    rvm remove 2.3
    

卸载对应版本的ruby 使用安装包的形式安装ruby可以参考[安装ruby2.2.2](http://blog.csdn.net/coming789/article/details/42193531)

集群
--

安装和升级好ruby之后

    gem install  redis
    

在gem中安装redis，后面加`-v 版本号`可以指定安装的版本，使用`gem uninstall redis`卸载 如果安装很慢可以添加gem仓库

    gem sources -a https://ruby.taobao.org/
    gem sources -a https://gems.ruby-china.org/
    

使用`gem source`查看仓库，使用`gem source -r url`移除，`gem sources -u`更新 然后再执行

    ./redis-trib.rb  create --replicas 1 127.0.0.1.41:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
    

就可以看到

    >>> Creating cluster
    

在创建集群了 一些集群的操作参考：[集群命令](http://www.cnblogs.com/wxd0108/p/5798332.html)

* * *

集群删除节点

    ./redis-trib.rb del-node [node-id] [node-id]
    

这个node-id可以在删除的nodes.conf中看到 也可以连接到集群执行`cluster nodes`查看集群信息，执行 参考[redis集群动态增加或者删除节点](http://blog.csdn.net/xu470438000/article/details/42972123) 一些问题的解决方案参考[集群问题](http://blog.csdn.net/ownfire/article/details/46624005) 停掉指定端口的redis

    redis-cli -p port shutdown  //port为端口号
    

[集群](http://www.mamicode.com/info-detail-1225600.html) [集群](http://hot66hot.iteye.com/blog/2050676)

* * *

访问集群要使用

    redis-cli -c -p port
    

来访问，如果不行的话需要加上ip

    redis-cli -c -h ip -p port
    

如果一直出现连接问题，要修改`/etc/hosts`文件，详见[内网ip？？](http://blog.csdn.net/xlgen157387/article/details/52702659)