---
title: " Java线程池和队列\t\t"
tags:
  - 杂
url: 103.html
id: 103
comments: false
categories:
  - 杂
date: 2018-10-19 09:33:52
---
参考：
[Java ThreadPool](http://www.cnblogs.com/kuoAT/p/6714762.html)
[由浅入深理解Java线程池及线程池的如何使用](https://www.cnblogs.com/superfj/p/7544971.html)

***
## 线程池规则
线程池的线程执行规则跟任务队列有很大的关系。

下面都假设任务队列没有大小限制：

- 如果线程数量<=核心线程数量，那么直接启动一个核心线程来执行任务，不会放入队列中。
- 如果线程数量>核心线程数，但<=最大线程数，并且任务队列是LinkedBlockingDeque的时候，超过核心线程数量   的任务会放在任务队列中排队。
- 如果线程数量>核心线程数，但<=最大线程数，并且任务队列是SynchronousQueue的时候，线程池会创建新线程   执行任务，这些任务也不会被放在任务队列中。这些线程属于非核心线程，在任务完成后，闲置时间达到了超时时间就会被清除。
- 如果线程数量>核心线程数，并且>最大线程数，当任务队列是LinkedBlockingDeque，会将超过核心线程的任务   放在任务队列中排队。也就是当任务队列是LinkedBlockingDeque并且没有大小限制时，线程池的最大线程数设   置是无效的，他的线程数最多不会超过核心线程数。
- 如果线程数量>核心线程数，并且>最大线程数，当任务队列是SynchronousQueue的时候，会因为线程池拒绝添加   任务而抛出异常。

任务队列大小有限时

- 当LinkedBlockingDeque塞满时，新增的任务会直接创建新线程来执行，当创建的线程数量超过最大线程数量时会抛异常。
- SynchronousQueue没有数量限制。因为他根本不保持这些任务，而是直接交给线程池去执行。当任务数量超过最大线程数时会直接抛异常。
***

## 任务缓存队列

在前面我们多次提到了任务缓存队列，即workQueue，它用来存放等待执行的任务。

workQueue的类型为BlockingQueue<Runnable>，通常可以取下面三种类型：

- 有界任务队列ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小；

- 无界任务队列LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE；

- 直接提交队列synchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务