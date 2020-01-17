---
title: " JDK8 Stream API\t\t"
tags:
  - 杂
url: 91.html
id: 91
comments: false
categories:
  - 杂
date: 2018-10-19 08:46:21
---
参考：
[Java Stream API入门篇](https://www.cnblogs.com/CarpenterLee/p/6545321.html)
[Java8 Stream API介绍](https://blog.csdn.net/chaoer89/article/details/52389458)
[Java8之Stream类](https://www.jianshu.com/p/c53eb31752c4)

---

```java
    List<String> strings = Arrays.asList("a", "d", "ga", "dsagafd","adfsafdsa","a","d");
    strings.stream().filter(e->e.startsWith("a")).forEach(e-> System.out.println(e));
    System.out.println("-------------------");
    strings.stream().distinct().forEach(e-> System.out.println(e));
    Stream<String> sorted = strings.stream().sorted((a, b) -> b.length() - a.length());
    System.out.println("==================");
    sorted.forEach(a-> System.out.println(a));
    boolean a = strings.stream().anyMatch((s) -> s.equals("a"));
    boolean b = strings.stream().allMatch((s) -> s.equals("a"));
    boolean c = strings.stream().noneMatch((s) -> s.equals("a"));
    System.out.println(a);
    System.out.println(b);
    System.out.println(c);
    ---------------------结果--------------------------
    a
    adfsafdsa
    a
    -------------------
    a
    d
    ga
    dsagafd
    adfsafdsa
    ==================
    adfsafdsa
    dsagafd
    ga
    a
    d
    a
    d
    true
    false
    false
```
对stream的操作分为为两类，中间操作(intermediate operations)和结束操作(terminal operations)
中间操作是惰性化的，每次调用会对Stream做一定的处理，返回一个新的Stream，并没有真正开始计算
每个stream只能有一个结束操作，执行了结束操作之后，stream就被消费掉了。


常用方法：
中间操作：`concat() distinct() filter() flatMap() limit() map() peek() 
skip() sorted() parallel() sequential() unordered()`
结束操作：`allMatch() anyMatch() collect() count() findAny() findFirst() 
forEach() forEachOrdered() max() min() noneMatch() reduce() toArray()`

`sort()`有两种方式，对stream进行排序，默认排序和自定义的Comparable形式，例子参见上方
`map`和`flatMap` 都是对stream进行遍历操作，都会返回操作之后的stream，不同的是flatMap会将多个集合中的元素全部拿出来放到一个集合中，而map不会
如对两个集合[1,2,3],[8,9,7]
stream之后的map操作的结果是[1,2,3],[8,9,7]
stream之后的flatMap结果是[1,2,3,8,9,7]
`filter`可以对元素进行过滤，参考上面例子
`limit`截取stream中的元素
`skip`跳过stream中的元素
`distinct`去重，通过equals和hashcode方法，自定义类
`anyMatch `,`allMatch`,`noneMatch`,stream中的元素有匹配，全部匹配，全都不匹配。
`reduce`聚合操作