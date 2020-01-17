---
title: " RESTful风格\t\t"
tags:
  - 杂
url: 105.html
id: 105
comments: false
categories:
  - 杂
date: 2018-10-19 09:34:40
---
参考：[怎样用通俗的语言解释REST](https://www.zhihu.com/question/28557115)
　　　[理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)
　　　[Roy Fielding的毕业论文-REST章节](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
　　　[RESTful 架构风格概述](https://blog.igevin.info/posts/restful-architecture-in-general/)
***
## REST概念
REST并不是rest的意思，全称是Representational State Transfer(表现层状态转移)
REST的概念里面有几个特征，`Resources`，`Representational`，`State Transfer`
简单理解如下：

- 资源(Resource)
  把所有请求当成对资源的操作，每个url地址对应的就是一个资源的地址，对资源的操作不体现在请求的url上，体现在HTTP请求动词上，如`GET`，`POST`，`PUT`，`DELETE`，url地址本身只是代表资源的位置，故不能包含动词，引用参考中的例子,`:`后的为注释

        BAD
        /getProducts
        /listOrders
        /retrieveClientByOrder?orderId=1
        GOOD
        GET /products : will return the list of all products
        POST /products : will add a product to the collection
        GET /products/4 : will retrieve product 4PATCH/
        PUT /products/4 : will update product #4

- 表现层(Representation)

> 比如，文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现。

资源的表现形式，即这个资源的格式类型，这个资源是图片，网页，文本，或者是其他的表现形式指定也不想原来一样在url中用xxx.png这种方式呈现，而是使用请求中的HTTP协议的ACCEPT和Content-Type来描述表现层。

- 状态转化
  这个状态是资源的状态，使用`GET`，`POST`，`PUT`，`DELETE`等HTTP协议中的动词去改变资源的状态
## 使用SpringMVC实现RESTful风格
参考：<http://blog.csdn.net/w605283073/article/details/51338597>

