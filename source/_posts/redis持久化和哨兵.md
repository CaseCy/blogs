---
title: redis持久化和哨兵
date: 2020-01-15 9:05:38
categories:
 - redis
tags: redis
---

## 持久化

使用持久化能够保证数据不会程序退出等造成数据丢失的情况，重启时利用持久化文件就可以实现数据恢复。

redis支持两种持久化方式：

- RDB：RDB是把当前进程数据生成快照保存到硬盘的过程，可手动触发和自动触发
- AOF（append only file）：以独立的日志方式记录每次写命令。重启时重新执行AOF文件中的命令以恢复数据

### RDB

#### 手动触发

- save命令
- bgsave命令

save命令会阻塞当前Redis服务器，直到RDB过程完成为止，对于内存比较大的实例会造成长时间的阻塞

bgsave命令是针对save命令做的优化，采用子线程去执行RDB操作，只会在fork操作创建子线程的阻塞，一般比较短，fork操作完成后会输出一段话：

```
Background saving started
```

### AOF

开启AOF需要设置配置`appendonly`为yes

AOF工作流程分为几部：

命令写入-》AOF缓冲-》AOF文件

重启的时候会去加载目录下的AOF文件

#### 重写机制

命令不断写入AOF会使文件越来越大，所以redis引入AOF重写机制压缩文件体积，**AOF重写是把redis进程内的数据转化为写命令同步到新AOF文件的过程**

重写分为手动触发和自动触发：

- 手动触发：调用`bgrewriteaof`命令

- 自动触发：根据`auto-aof-rewrite-min-size`和`auto-aof-rewrite-percentage`参数确定自动触发的时机

  自动 触发 时机= aof_ current_ size> auto- aof- rewrite- min- size&&（ aof_ current_ size- aof_ base_ size）/ aof_ base_ size>= auto- aof- rewrite- percentage


### 路径和文件名称

RDB文件的生成路径通过配置文件中的`dir`字段设置，AOF也是如此

RDB文件的名称通过`dbfilename`配置

AOF文件的名称通过`appendfilename`配置

redis默认会对RDB文件进行压缩处理，默认开启，可以通过`rdbcopression`开启和关闭

### 重启加载

![](/static/redis重启加载流程图.png)

### 其他
#### 动态设置配置

可在redis运行过程中通过

```
config set [参数名] [参数值]
config get [参数名]
```

设置参数值，但是有效范围只在当次运行过程中，重启后失效

#### 磁盘满了的情况

可以通过config set dir [rdb的文件位置]，在运行时修改生成文件的位置，然后执行bgsave命令进行磁盘的切换

#### 优缺点

- RDB：

  - 优点：

    RDB代表redis在某个时间点的快照，适用于备份，全量复制

    redis加载RDB文件的速度远远快于AOF的方式
  
  - 缺点：
  
    RDB方式没办法做到实时持久化，因为RDB操作执行成本比较高
  
    **RDB文件在redis版本更迭中有多个格式的RDB文件，存在老版本Redis无法兼容新版RDB格式的问题**

- AOF：

## 哨兵

哨兵是为了解决redis主从复制过程中，主节点因为故障不能够提供服务，自动的将从节点升为主节点，并通知应用方更新主节点的解决架构

---

## 关于redis的一些问题

 https://juejin.im/post/5db66ed9e51d452a2f15d833