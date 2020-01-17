---
title: " Apache Pool\t\t"
tags:
  - 杂
url: 87.html
id: 87
comments: false
categories:
  - 杂
date: 2018-10-19 08:41:36
---

tags： Apache pool 对象池 引用：[对象池，包括带key的](http://www.cnblogs.com/jinzhiming/p/5120670.html)

* * *

导入pom.xml

```xml
<!--commons pool-->
<dependency>
    <groupId>commons-pool</groupId>
    <artifactId>commons-pool</artifactId>
    <version>1.6</version>
</dependency>
```


注：这里的pool不是pool2

* * *

不带key的对象池
---------

概述： 　　factory对象实现`PoolableObjectFactory`接口，完成其中的方法，可以在调用实例中将factory对象通过构造方法放入`GenericObjectPool`或者`StackObjectPool` 测试对象

```java
public class PoolTestObject {

    //记录次数
    private int num;
    private boolean active;
    //对象id
    private String uuid;

    public PoolTestObject() {
        this.uuid = UUID.randomUUID().toString();
        System.out.println("create a Object UUID:"+ uuid);
    }
    //略去getter/setter
}
```


不带key的poolFactory

```java
public class PoolTestFactory implements PoolableObjectFactory<PoolTestObject> {

    @Override
    public PoolTestObject makeObject() throws Exception {
        PoolTestObject testPoolObject = new PoolTestObject();
        System.out.println("创建对象,uuid:"+testPoolObject.getUuid());
        return testPoolObject;
    }

    @Override
    public void destroyObject(PoolTestObject testPoolObject) throws Exception {
        testPoolObject = null;
        System.out.println("摧毁对象！,uuid:"+testPoolObject.getUuid());
    }
```


​    
```java
    @Override
    public boolean validateObject(PoolTestObject testPoolObject) {
        System.out.println("验证对象！,uuid:"+testPoolObject.getUuid());
        //验证实例是否安全
        if (testPoolObject.isActive()){
            return true;
        }else {
            return false;
        }
    }

    //重新初始化实例返回池
    @Override
    public void activateObject(PoolTestObject testPoolObject) throws Exception {
        testPoolObject.setActive(true);
        System.out.println("激活对象！uuid:"+testPoolObject.getUuid());
    }

    //取消初始化实例返回到空闲对象池
    @Override
    public void passivateObject(PoolTestObject testPoolObject) throws Exception {
        testPoolObject.setActive(false);
        System.out.println("钝化对象 uuid:"+testPoolObject.getUuid());
    }
}
```


验证(使用的引用中的例子，并做了点修改)：

```java
@Test
    public void testPool(){
        PoolTestObject bo = null;
        PoolableObjectFactory factory = new PoolTestFactory();
        GenericObjectPool pool = new GenericObjectPool(factory);
        GenericObjectPool.Config config = new GenericObjectPool.Config();//连接池配置
        config.maxActive = 5;
        pool.setConfig(config);//设置连接池
        //这里两种池都可以
        //ObjectPool pool = new StackObjectPool(factory);
        try {
            for(int i = 0; i < 5; i++) {
                System.out.println("\n==========="+i+"===========");
                System.out.println("池中处于闲置状态的实例pool.getNumIdle()："+pool.getNumIdle());//拿到闲置状态的链接数
                //从池里取一个对象，新创建makeObject或将以前闲置的对象取出来
                bo = (PoolTestObject)pool.borrowObject();
                System.out.println(bo.getUuid());
                bo.setUuid("-----------------------------------------1");
                System.out.println("bo:"+bo);
                System.out.println(bo.getNum());;
                System.out.println("池中所有在用实例数量pool.getNumActive()："+pool.getNumActive());
                if((i%2) == 0) {
                    //用完之后归还对象
                    pool.returnObject(bo);
                    System.out.println("归还对象！！！！");
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if(bo != null) {
                    pool.returnObject(bo);
                }
                //关闭池
                pool.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
```


调用结果，可以看到factory中的各个方法的执行过程,以及返回的对象如果做了修改，会保存这个修改，归还了对象之后，这个对象会回到池中作为空闲对象，下次borrow的时候，如果有空闲对象，会先调用空闲对象，如果没有空闲对象了，会调用`makeObject`去自动创建对象

```java
===========0===========
池中处于闲置状态的实例pool.getNumIdle()：0
create a Object UUID:04ef2118-0b83-458c-b8e9-dc6d4b6548f8
创建对象,uuid:04ef2118-0b83-458c-b8e9-dc6d4b6548f8
激活对象！uuid:04ef2118-0b83-458c-b8e9-dc6d4b6548f8
04ef2118-0b83-458c-b8e9-dc6d4b6548f8
bo:cn.ws.apachepool.PoolTestObject@3d012ddd
0
池中所有在用实例数量pool.getNumActive()：1
钝化对象 uuid:-----------------------------------------1
归还对象！！！！

===========1===========
池中处于闲置状态的实例pool.getNumIdle()：1
激活对象！uuid:-----------------------------------------1
-----------------------------------------1
bo:cn.ws.apachepool.PoolTestObject@3d012ddd
0
池中所有在用实例数量pool.getNumActive()：1

===========2===========
池中处于闲置状态的实例pool.getNumIdle()：0
create a Object UUID:d979744b-fa76-4547-a34d-432e75049249
创建对象,uuid:d979744b-fa76-4547-a34d-432e75049249
激活对象！uuid:d979744b-fa76-4547-a34d-432e75049249
d979744b-fa76-4547-a34d-432e75049249
bo:cn.ws.apachepool.PoolTestObject@6f2b958e
0
池中所有在用实例数量pool.getNumActive()：2
钝化对象 uuid:-----------------------------------------1
归还对象！！！！

===========3===========
池中处于闲置状态的实例pool.getNumIdle()：1
激活对象！uuid:-----------------------------------------1
-----------------------------------------1
bo:cn.ws.apachepool.PoolTestObject@6f2b958e
0
池中所有在用实例数量pool.getNumActive()：2

===========4===========
池中处于闲置状态的实例pool.getNumIdle()：0
create a Object UUID:34bf5dce-6449-439c-ac92-5d26557d37b9
创建对象,uuid:34bf5dce-6449-439c-ac92-5d26557d37b9
激活对象！uuid:34bf5dce-6449-439c-ac92-5d26557d37b9
34bf5dce-6449-439c-ac92-5d26557d37b9
bo:cn.ws.apachepool.PoolTestObject@1eb44e46
0
池中所有在用实例数量pool.getNumActive()：3
钝化对象 uuid:-----------------------------------------1
归还对象！！！！
钝化对象 uuid:-----------------------------------------1
```


`GenericObjectPool`可以在创建了之后设置`Cofig`可以查看设置的源码 具体参数如下：

```java
public void setConfig(GenericObjectPool.Config conf) {
        synchronized(this) {
            this.setMaxIdle(conf.maxIdle); 
            this.setMinIdle(conf.minIdle);  
            this.setMaxActive(conf.maxActive); //能借出的最大值，为负数表示没限制
            this.setMaxWait(conf.maxWait);  //最大等待时间
            this.setWhenExhaustedAction(conf.whenExhaustedAction);//借出对象达到最大值时的行为
            this.setTestOnBorrow(conf.testOnBorrow);    //设定在借出对象时是否进行有效性检查
            this.setTestOnReturn(conf.testOnReturn);    //设定在还回对象时是否进行有效性检查
            this.setTestWhileIdle(conf.testWhileIdle);  //设定在进行后台对象清理时，是否还对没有过期的池内对象进行有效性检查。不能通过有效性检查的对象也将被回收
            this.setNumTestsPerEvictionRun(conf.numTestsPerEvictionRun);
            this.setMinEvictableIdleTimeMillis(conf.minEvictableIdleTimeMillis);    //设定在进行后台对象清理时，视休眠时间超过了多少毫秒的对象为过期。过期的对象将被回收。如果这个值不是正数，那么对休眠时间没有特别的约束。
            this.setTimeBetweenEvictionRunsMillis(conf.timeBetweenEvictionRunsMillis);  //设定间隔每过多少毫秒进行一次后台对象清理的行动。如果这个值不是正数，则实际上不会进行后台对象清理。
            this.setSoftMinEvictableIdleTimeMillis(conf.softMinEvictableIdleTimeMillis);
            this.setLifo(conf.lifo);    //池对象的放入和取出默认是后进先出的原则，默认是true，代表后进后出，设置为false代表先进先出。
        }
```


> 参数whenExhaustedA ction指定在池中借出对象的数目已达极限的情况下，调用它的borrowObject方法时的行为。可以选用的值有：

```java
GenericObjectPool.WHEN_EXHAUSTED_BLOCK，表示等待；
GenericObjectPool.WHEN_EXHAUSTED_GROW，表示创建新的实例（不过这就使maxActive参数失去了意义）；
GenericObjectPool.WHEN_EXHAUSTED_FAIL，表示抛出一个java.util.NoSuchElementException异常。
```

* * *

带key的对象池
--------

概述：factory对象实现`KeyedPoolableObjectFactory`接口，完成其中的方法，可以在调用实例中将factory对象通过构造方法放入`GenericKeyedObjectPool` 可以通过`keyPool.addObject("name");`往池中添加对象，也可以直接通过`keyPool.borrow("name");`如果没有，会自动创建一个再借出来 带key的对象池和不带key的对象池区别在于可以通过key去取到特定的对象(在添加对象的时候回去设置key)，具体参考[引用](http://www.cnblogs.com/jinzhiming/p/5120670.html)

* * *

其他
--

还可以实现`ObjectPool`接口，完成其中的方法，来自己定义一个Pool，这里的`GenericObjectPool`就是实现了`ObjectPool`接口。

```java
public interface ObjectPool<T> {
    T borrowObject() throws Exception, NoSuchElementException, IllegalStateException;

    void returnObject(T var1) throws Exception;

    void invalidateObject(T var1) throws Exception;

    void addObject() throws Exception, IllegalStateException, UnsupportedOperationException;

    int getNumIdle() throws UnsupportedOperationException;

    int getNumActive() throws UnsupportedOperationException;

    void clear() throws Exception, UnsupportedOperationException;

    void close() throws Exception;

    /** @deprecated */
    @Deprecated
    void setFactory(PoolableObjectFactory<T> var1) throws IllegalStateException, UnsupportedOperationException;
}
```