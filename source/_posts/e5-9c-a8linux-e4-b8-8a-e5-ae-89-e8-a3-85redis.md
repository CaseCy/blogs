---
title: " 在Linux上安装Redis\t\t"
tags:
  - redis
url: 89.html
id: 89
comments: false
categories:
  - redis
date: 2018-10-19 08:43:45
---

tags： Redis Linux Centos 引用：[Redis1](http://www.cnblogs.com/dongfanghao/p/7158282.html) [Redis2](http://blog.csdn.net/ludonqin/article/details/47211109)

* * *

安装
--

安装C语言的编译环境

    yum install gcc
    

下载Redis的安装包[Redis](https://redis.io/download) 放到`/usr/local/src`下面 解压缩

    tar -xzvf redis-3.2.9.tar.gz
    

到解压后的目录`/usr/local/src/redis-3.2.9` 编译打包

    make
    

常用命令

    make     　　　　　　　　　　// 编译redis
    make distclean　　   　　// 清除之前编译过程中残留的文件等,可在make前执行
    redis-server       // 默认启动redis服务器
    redis-server redis.conf    // 使用配置文件启动redis服务器
    

跳转到`src`目录执行,也可以手动拷贝redis-server、redis-cli、redis-check-aof、redis-check-dump等至/usr/local/bin目录下

    make install
    

跳转到`/usr/local/bin`下，会多出下面的文件

    [root@centos bin]# ll
    总用量 26348
    -rwxr-xr-x. 1 root root 5580423  redis-benchmark
    -rwxr-xr-x. 1 root root   22185  redis-check-aof
    -rwxr-xr-x. 1 root root 7830146  redis-check-rdb
    -rwxr-xr-x. 1 root root 5709355  redis-cli
    lrwxrwxrwx. 1 root root      12  redis-sentinel -> redis-server
    -rwxr-xr-x. 1 root root 7830146  redis-server
    

redis.conf
----------

在`/usr/local/src`下面建一个redis文件夹，用来存放配置文件和一些其他信息

    mkdir redis //创建redis文件夹
    cp ../redis-3.2.9/redis.conf . //将redis.conf复制到当前目录
    vi redis.conf
    

打开redis.conf设置

    `bind 127.0.0.1`
    protected-mode yes
    `port 6379`
    tcp-backlog 511
    timeout 0
    tcp-keepalive 300
    `daemonize yes`
    supervised no
    `pidfile /var/run/redis_6379.pid`
    loglevel notice
    `logfile ./redis.log`
    databases 16
    save 900 1
    save 300 10
    save 60 10000
    stop-writes-on-bgsave-error yes
    rdbcompression yes
    rdbchecksum yes
    dbfilename dump.rdb
    `dir ./`
    slave-serve-stale-data yes
    slave-read-only yes
    repl-diskless-sync no
    repl-diskless-sync-delay 5
    repl-disable-tcp-nodelay no
    slave-priority 100
    `appendonly yes`
    appendfilename "appendonly.aof"
    appendfsync everysec
    no-appendfsync-on-rewrite no
    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 64mb
    aof-load-truncated yes
    lua-time-limit 5000
    slowlog-log-slower-than 10000
    slowlog-max-len 128
    latency-monitor-threshold 0
    notify-keyspace-events ""
    hash-max-ziplist-entries 512
    hash-max-ziplist-value 64
    list-max-ziplist-size -2
    list-compress-depth 0
    set-max-intset-entries 512
    zset-max-ziplist-entries 128
    zset-max-ziplist-value 64
    hll-sparse-max-bytes 3000
    activerehashing yes
    client-output-buffer-limit normal 0 0 0
    client-output-buffer-limit slave 256mb 64mb 60
    client-output-buffer-limit pubsub 32mb 8mb 60
    hz 10
    aof-rewrite-incremental-fsync yes
    

几个常用的设置

    bind 127.0.0.1  //访问ip，0.0.0.0 表示任何人都可以访问
    port 6379 //默认的端口
    daemonize yes //运行模式，yes表示后台运行 no表示Console运行
    pidfile /var/run/redis_6379.pid 进程文件
    logfile ./redis.log  //log文件
    appendonly yes //是否持久化，yes会在当前目录生成文件
    

使用`i`编辑，`/`查找内容，`:wq`保存并退出 `:q!`不保存退出`n`查找下一个

启动和关闭
-----

使用配置文件启动 可以在新建的redis目录里执行

    redis-server redis.conf
    

使用客户端连接redis

    redis-cli
    

退出客户端

    Ctrl+c
    

关闭redis

    redis-cli shutdown
    #带密码的关闭(带密码连接一样，修改后面命令即可)
    redis-cli -a 密码 shutdown
    

或者

    ps -ef |grep redis
    kill -9 pid
    

配置Redis服务
---------

创建redis启动脚本 拷贝解压包下utils下redis启动脚本至`/etc/init.d/`

    cp redis_init_script /etc/init.d/
    

修改脚本名称(也可不修改)为redis_6379

    mv redis_init_script redis_6379
    
    vi redis_6379
    

将对应的参数修改

    REDISPORT=6379
    EXEC=/usr/local/bin/redis-server
    CLIEXEC=/usr/local/bin/redis-cli
    
    PIDFILE=/var/run/redis_${REDISPORT}.pid
    

CONF="/usr/local/src/redis/redis_${REDISPORT}.conf" 启动/关闭/重启

    /etc/init.d/redis_6379 start/stop/restart
    

或者

    service redis_6379 start/stop/restart
    

* * *

Redis的常用命令
----------

参考：[Redis常用命令](http://www.cnblogs.com/whoamme/p/3532129.html)

    redis-cli -p 6379  //客户端连接指定端口
    

返回值说明。默认返回`(integer) 0`表示false，即失败，返回`(integer) 1`表示true，即成功 字符串：

    set key value
    get key value
    setnx key value  //如果key存在就不设置，如果不存在就设置，
    mset key value key value...     //同时设置多个值，同时成功，同时失败
    msetnx key value key value...
    mget key1 key2 key3     //获取多个Key的值
    getset key newvalue     //获取key并设置为新值
    append key word         //给key追加word中的内容
    setrange key n repvalue    //将key对应value的第n个字符和这里的repvalue替换(从0开始)
    getrange key start end  //获取key的值从start到end
    strlen key      //返回key对应value值的长度
    

key-value

    keys *      //列出所有的键值
    rename key new key  //重命名键值
    type key    //判断key的类型
    exists key  //判断key是否存在
    del key     //删除key
    expire key second   //设置key的过期时间，单位为秒
    ttl key         //查看key的过期时间 -2为已过期 -1为没有设置过期
    persist key     //取消过期时间设置
    randomkey 随机返回一个key