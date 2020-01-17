---
title: "@Transactional标签的事务特性"
tags:
  - spring
url: 85.html
id: 85
comments: false
categories:
  - 未分类
date: 2018-10-19 08:39:14
---
# @Transactional标签的事务特性

## 使用方式
## 事务传播方式
```java
@Transactional(propagation=Propagation.REQUIRED)
```
默认的方式：如果有事务, 那么加入事务, 没有的话新建一个
```java
@Transactional(propagation=Propagation.NOT_SUPPORTED)
```
容器不为这个方法开启事务
```java
@Transactional(propagation=Propagation.REQUIRES_NEW)
```
不管是否存在事务,都创建一个新的事务,原来的挂起,新的执行完毕,继续执行老的事务
```java
@Transactional(propagation=Propagation.MANDATORY) 
```
必须在一个已有的事务中执行,否则抛出异常
```java
@Transactional(propagation=Propagation.NEVER) 
```
必须在一个没有的事务中执行,否则抛出异常(与Propagation.MANDATORY相反)
```java
@Transactional(propagation=Propagation.SUPPORTS) 
```
如果其他bean调用这个方法,在其他bean中声明事务,那就用事务.如果其他bean没有声明事务,那就不用事务
## 事务隔离级别
```java
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
```
读取未提交数据(会出现脏读, 不可重复读) 基本不使用
```java
@Transactional(isolation = Isolation.READ_COMMITTED)
```
读取已提交数据(会出现不可重复读和幻读)
```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
```
可重复读(会出现幻读)
```java
@Transactional(isolation = Isolation.SERIALIZABLE)
```
## 标签中的常用属性

 - readOnly
     该属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false。例如：`@Transactional(readOnly=true)`
 - rollbackFor
    该属性用于设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。例如：
    指定单一异常类：`@Transactional(rollbackFor=RuntimeException.class)`
    指定多个异常类：`@Transactional(rollbackFor={RuntimeException.class, Exception.class})`
 - rollbackForClassName
    该属性用于设置需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，则进行事务回滚。例如：
    指定单一异常类名称：`@Transactional(rollbackForClassName="RuntimeException")`
    指定多个异常类名称：`@Transactional(rollbackForClassName={"RuntimeException","Exception"})`
 - propagation
    该属性用于设置事务的传播行为
    例如：`@Transactional(propagation=Propagation.NOT_SUPPORTED,readOnly=true)`

## Attention:
@Transactional使用的时候同一个类的不同方法不会有事务的传播，不同的类会有事务的传播，比如
```java
Class Test{
    public void A(){
        B("a");
    }
    
    @Transactional(rollbackFor = Exception.class)
    public void B(String word){
        
        System.out.println(word);
        int a = 1/0;
    }
}
//在A中调用B当B中发生异常的时候，这里不会出现回滚，因为这个时候A不具有事务特性，在同一个类中的不同方法上不具有事务的传播特性
```