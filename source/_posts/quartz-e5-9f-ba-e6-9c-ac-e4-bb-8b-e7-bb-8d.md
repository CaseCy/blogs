---
title: " Quartz基本介绍\t\t"
tags:
  - java
url: 398.html
id: 398
comments: false
categories:
  - java
date: 2019-04-04 10:29:32
---

参考引用： [Quartz官方文档](https://www.w3cschool.cn/quartz_doc/) [Quartz大致介绍](https://blog.csdn.net/u010648555/article/details/54863144) [Quartz-Trigger详解](https://blog.csdn.net/yangshangwei/article/details/78172788#calendarintervaltrigger)

基本概念
----

**_Quartz是一个完全由Java编写的开源作业调度框架_**，是OpenSymphony开源组织在Job scheduling领域又一个开源项目，可以创建，暂停，取消，同时运行，处理多个定时任务，在设定时间到达时自动执行任务，且非常易用。

核心概念
----

**Job** 表示一个工作，要执行的具体内容。此**_接口_**中只有一个方法，如下：

    void execute(JobExecutionContext context) 
    

**JobDetail** 表示一个具体的可执行的调度程序，Job 是这个可执行程调度程序所要执行的内容，另外 JobDetail 还包含了这个任务调度的方案和策略。 **Trigger** 代表一个调度参数的配置，什么时候去调。 **Scheduler** 代表一个调度容器，一个调度容器中可以注册多个 JobDetail 和 Trigger。当 Trigger 与 JobDetail 组合，就可以被 Scheduler 容器调度了。 注：**一个Job可以被多个Trigger 绑定，但是一个Trigger只能绑定一个Job！**

定时器的种类
------

Quartz 中五种类型的 Trigger：

*   SimpleTrigger：用来触发只需执行一次或者在给定时间触发并且重复N次且每次执行延迟一定时间的任务
    
*   CronTirgger：使用Cron表达式来触发，附：[Cron在线生成器](http://www.bejson.com/othertools/cron/)
    
*   DailyTimeIntervalTrigger：指定每天的某个时间段内，以一定的时间间隔执行任务。并且它可以支持指定星期。
    
*   NthIncludedDayTrigger：用于在每一间隔类型的第几天执行 Job
    
*   CalendarIntervalTrigger：类似于SimpleTrigger，指定从某一个时间开始，以一定的时间间隔执行的任务
    

存储方式
----

*   RAMJobStore：使用内存作为存储
    
*   JDBCJobStore ：使用数据库作为存储
    

**类型**

**优点**

**缺点**

RAMJobStore

不要外部数据库，配置容易，运行速度快

因为调度程序信息是存储在被分配给JVM的内存里面，所以，当应用程序停止运行时，所有调度信息将被丢失。另外因为存储到JVM内存里面，所以可以存储多少个Job和Trigger将会受到限制

JDBCJobStore

支持集群，因为所有的任务信息都会保存到数据库中，可以控制事物，还有就是如果应用服务器关闭或者重启，任务信息都不会丢失，并且可以恢复因服务器关闭或者重启而导致执行失败的任务

运行速度的快慢取决与连接数据库的快慢

注：使用数据库方式时，需要配置数据源，并导入Quartz提供的sql，需要下载对应版本的Quartz，[官网传送门](http://www.quartz-scheduler.org/downloads/)，sql位置：`org/quartz/impl/jdbcjobstore`，选择对应数据源的sql导入