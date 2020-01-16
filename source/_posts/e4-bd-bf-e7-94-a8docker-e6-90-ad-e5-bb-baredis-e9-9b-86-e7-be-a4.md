---
title: " 使用docker搭建redis集群\t\t"
tags:
  - docker
url: 425.html
id: 425
comments: false
categories:
  - docker
date: 2019-09-12 12:08:05
---

参考： [docker redis4.0 集群（cluster）搭建](https://www.cnblogs.com/lianggp/p/8136222.html)

创建redis容器
---------

### 1.创建配置文件

要搭建集群的文件夹下创建redis-cluster.tmpl，内容为以下内容，根据需要更改我这里文件在`/root/redis-cluster`(后面会用到这个路径)下

    port ${PORT}
    protected-mode no
    cluster-enabled yes
    cluster-config-file nodes.conf
    cluster-node-timeout 5000
    cluster-announce-ip 39.10X.XX.XX //自己服务器IP
    cluster-announce-port ${PORT}
    cluster-announce-bus-port 1${PORT}
    appendonly yes
    

如果需要密码则加上配置

    masterauth 123456
    requirepass 123456
    

注：5.x以下版本不支持在创建集群的时候指定密码，所以要先创建好集群之后，再设置密码，5.x+版本支持在创建集群的时候指定参数`-a`来指定创建集群时候密码

### 2.docker创建集群的网络

    docker network create redis-net
    

### 3.生成配置信息

    for port in `seq 7000 7005`; do \
      mkdir -p ./${port}/conf \
      && PORT=${port} envsubst < ./redis-cluster.tmpl > ./${port}/conf/redis.conf \
      && mkdir -p ./${port}/data; \
    done
    

执行之后会在当前目录下生成 7000-7005的文件夹，对应不同的端口，里面包含conf和data文件夹

### 4.创建redis容器

    for port in `seq 7000 7005`; do \
      docker run -d -ti -p ${port}:${port} -p 1${port}:1${port} \
      -v /root/redis-cluster/${port}/conf/redis.conf:/usr/local/etc/redis/redis.conf \
      -v /root/redis-cluster/${port}/data:/data \
      --restart always --name redis-${port} --net redis-net \
      redis:4.0 redis-server /usr/local/etc/redis/redis.conf; \
    done
    

这里会启动6个redis 打开端口7000-7005,17000-17005并映射到了主机，将端口文件夹中的配置文件挂载到容器中，端口文件夹中的data文件夹挂载到容器中的/data下并自动重启 注：这里使用的是redis的4.0版本

集群
--

### 3.x版本

redis在4.x+版本提供了对docker的友好支持，如果需要redis能够使用外网访问，最好使用redis4.x+版本。3.x版本使用docker的默认网络配置只能够创建内网能够访问的docker redis集群，如果需要外网能够访问可能要更改redis-net的网络模式为host，并使用`redis-trib.rb`来创建集群，这里没有做尝试。

### 4.x版本

使用ruby来指定redis的`redis-trib.rb`来创建集群 注：5.0的redis版本似乎不支持使用这种方式（5.0使用redis-cli来创建集群，使用5.0的话不需要使用ruby，用其中一个redis实例来执行redis-cli创建集群的命令即可），所以这里的`redis-trib.rb`文件需要使用5.0以下的版本，参考文章中的获取的`redis-trib.rb`是属于stable版本的，最新stable版本的似乎已经是5.0的了，所以这里需要去到redis5.0版本以下的源码中去获取该文件。

#### 1.下载`redis-trib.rb`

下载redis4.x的源码文件（自行下载，这里不赘述），解压，我这里解压到`/root/redis-4.0.12`这个文件夹下的

#### 2.使用ruby + redis-trib.rb 创建集群

    echo yes | docker run -i --rm -v /root/redis-4.0.12/src/redis-trib.rb:/redis-trib.rb --net redis-net ruby sh -c '\
      gem install redis -v 4.0 \
      && ruby redis-trib.rb create --replicas 1 \
      '"$(for port in `seq 7000 7005`; do \
        echo -n "$(docker inspect --format '{ { (index .NetworkSettings.Networks "redis-net").IPAddress }}' "redis-${port}")":${port} ' ' ; \
      done)"
    

这里就是将`/root/redis-4.0.12/src/redis-trib.rb`该文件挂载到容器的`/redis-trib.rb`这个目录下，然后使用ruby来执行该文件创建集群

### 5.x版本

5.x使用redis-cli来创建集群，所以这里不需要去挂载`redis-trib.rb`,单独启动一个5.x的redis实例，指定redis-cli创建集群的命令就可以了

    echo yes | docker run -i --rm -p 6379 --net redis-net redis:5.0 \
       redis-cli --cluster create \
       $(for port in `seq 7000 7005`; do \
         echo -n "$(docker inspect --format '{ { (index .NetworkSettings.Networks "redis-net").IPAddress }}' "redis-${port}")":${port} ' ' ; \
         done) $(echo "--cluster-replicas 1")
    

如果使用了密码加上配置（5.x+版本支持，在配置文件中加`masterauth` `requirepass`属性）

    -a 123456
    

即

    echo yes | docker run -i --rm -p 6379 --net redis-net redis:5.0 \
       redis-cli --cluster create \
       $(for port in `seq 7000 7005`; do \
         echo -n "$(docker inspect --format '{ { (index .NetworkSettings.Networks "redis-net").IPAddress }}' "redis-${port}")":${port} ' ' ; \
         done) $(echo "--cluster-replicas 1 -a 123456")
    

注：

1.  操作环境为cento7
2.  注意各种路径
    
3.  执行之前要保证之前填的自己的IP的服务器的端口7000-7005,17000-17005端口是打开的，云服务器端口也要打开（如果需要外网连接的话），要不有可能一直卡在
    
        Waiting for the cluster to join...