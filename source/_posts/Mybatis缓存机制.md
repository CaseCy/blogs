---
title: Mybatis缓存机制
categories:
  - mybatis
date: 2020-02-21 18:54:25
tags: 
  - mybatis
  - 数据库
---

Mybatis缓存

mybatis分为一级缓存和二级缓存，**在默认的情况下，Mybatis只会开启一级缓存**

## 一级缓存

一级缓存是一次会话（SqlSession）的缓存，相当于本地缓存

对于同一个SqlSession对象，在参数和SQL完成一样的情况下，如果缓存没有过期，只执行一次SQL语句

一级缓存的生命周期:

- 开启一个新的会话时会创建一个新的SqlSession对象，里面会存在一个新的Excutor对象，Excutor对象持有一个新的PerpetualCache对象，会话结束时，Excutor对象和PerpetualCache对象一起释放掉

- SqlSession调用close方法后会清掉PerpetualCache，一级缓存就不可用了

- 调用clearCache，会清空PerpetualCache对象，但是可用

- 该SqlSession中执行insert，delete，update语句后会清除掉PerpetualCache中的数据

完全相同的查询：

-  传入的statementId（接口的形式就是方法名）

- 查询时的结果集中的结果范围

-  这次查询所产生的最终要传递给JDBC java.sql.Preparedstatement的Sql语句字符串（boundSql.getSql()没有参数）

- 传递给java.sql.Statement要设置的参数值

## 二级缓存

Mybatis的二级缓存是全局缓存，他可以提高对数据库的查询效率，提高应用的性能

默认是不开启的，如果要开启，Mybatis要求返回的POJO是可序列化的，如果配置了二级缓存，就意味着：

- 映射语句文件中的所有select语句将会被缓存。 
- 映射语句文件中的所欲insert、update和delete语句会刷新缓存。 
- 缓存会使用默认的Least Recently Used（LRU，最近最少使用的）算法来收回。 
- 根据时间表，比如No Flush Interval,（CNFI没有刷新间隔），缓存不会以任何时间顺序来刷新。 
- 缓存会存储列表集合或对象(无论查询方法返回什么)的1024个引用 
- 缓存会被视为是read/write(可读/可写)的缓存，意味着对象检索不是共享的，而且可以安全的被调用者修改，不干扰其他调用者或线程所做的潜在修改。 

二级缓存开启的方式：

1. 在xml映射文件中声明

   ```xml
   <!--开启本mapper的namespace下的二级缓存-->
       <!--
           eviction:代表的是缓存回收策略，目前MyBatis提供以下策略。
           (1) LRU,最近最少使用的，一处最长时间不用的对象
           (2) FIFO,先进先出，按对象进入缓存的顺序来移除他们
           (3) SOFT,软引用，移除基于垃圾回收器状态和软引用规则的对象
           (4) WEAK,弱引用，更积极的移除基于垃圾收集器状态和弱引用规则的对象。这里采用的是LRU，
                   移除最长时间不用的对形象
   
           flushInterval:刷新间隔时间，单位为毫秒，这里配置的是100秒刷新，如果你不配置它，那么当
           SQL被执行的时候才会去刷新缓存。
   
           size:引用数目，一个正整数，代表缓存最多可以存储多少个对象，不宜设置过大。设置过大会导致内存溢出。
           这里配置的是1024个对象
   
           readOnly:只读，意味着缓存数据只能读取而不能修改，这样设置的好处是我们可以快速读取缓存，缺点是我们没有办法修改缓存，他的默认值是false，不允许我们修改
   		type:指定自定义缓存的全限定名（需要实现Cache接口）
   		blocking:若缓存中找不到对应的key，是否会一直blocking，直到有对应的数据进入缓存
       -->
       <cache eviction="LRU" flushInterval="100000" readOnly="true" size="1024"/>
   <!--可以通过设置useCache来规定这个sql是否开启缓存，ture是开启，false是关闭-->
       <select id="selectAllStudents" resultMap="studentMap" useCache="true">
           SELECT id, name, age FROM student
       </select>
       <!--刷新二级缓存
       <select id="selectAllStudents" resultMap="studentMap" flushCache="true">
           SELECT id, name, age FROM student
       </select>
       -->
   ```
```
   
在配置文件中开启
   
   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <settings>
           <!--这个配置使全局的映射器(二级缓存)启用或禁用缓存-->
           <setting name="cacheEnabled" value="true" />
           .....
       </settings>
       ....
   </configuration>
```

   >  当二级缓存开启后，同一个命名空间(namespace) 所有的操作语句，都影响着一个**共同的 cache**，也就是二级缓存被多个 SqlSession 共享，是一个**全局的变量**。当开启缓存后，数据的查询执行的流程就是 二级缓存 -> 一级缓存 -> 数据库。 



---

参考：

 https://www.cnblogs.com/happyflyingpig/p/7739749.html 

 https://blog.csdn.net/weixin_37139197/article/details/82908377 

 https://www.cnblogs.com/cxuanBlog/p/11333021.html 