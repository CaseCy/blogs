---
title: " JSP\t\t"
tags:
  - front
url: 97.html
id: 97
comments: false
categories:
  - front
date: 2018-10-19 08:51:00
---

tags： Jsp 参考：[菜鸟教程](https://www.runoob.com/jsp/jsp-syntax.html)

* * *

脚本程序
----

    <% 代码片段 %>
    

示例代码

    <html>
    <head><title>Hello World</title></head>
    <body>
    Hello World!<br/>
    <%
    out.println("Your IP address is " + request.getRemoteAddr());
    %>
    </body>
    </html>
    

中文编码

    <%@ page language="java" contentType="text/html; charset=UTF-8"
        pageEncoding="UTF-8"%>
    

注：脚本程序中不能包含任何的文本，HTML标签，和JSP元素，如if写法

    <% if (day == 1 | day == 7) { %>
          <p>今天是周末</p>
    <% } else { %>
          <p>今天不是周末</p>
    <% } %>
    

JSP声明
-----

    <%! declaration; [ declaration; ]+ ... %>
    

示例：

    <%!
        private int initVar=0;
        private int serviceVar=0;
        private int destroyVar=0;
    %>
    
    <%!
        public void jspInit(){
            initVar++;
            System.out.println("jspInit(): JSP被初始化了"+initVar+"次");
        }
        public void jspDestroy(){
            destroyVar++;
            System.out.println("jspDestroy(): JSP被销毁了"+destroyVar+"次");
        }
    %>
    
    <%
        serviceVar++;
        System.out.println("_jspService(): JSP共响应了"+serviceVar+"次请求");
    
        String content1="初始化次数 : "+initVar;
        String content2="响应客户请求次数 : "+serviceVar;
        String content3="销毁次数 : "+destroyVar;
    %>
    

JSP注释方式及其含义
-----------

![注释](http://imageu.oss-cn-shenzhen.aliyuncs.com/noteimage/jsp%E6%B3%A8%E9%87%8A%E8%AF%AD%E6%B3%95.png) ##JSP表达式

> 一个JSP表达式中包含的脚本语言表达式，先被转化成String，然后插入到表达式出现的地方。 由于表达式的值会被转化成String，所以您可以在一个文本行中使用表达式而不用去管它是否是HTML标签。 表达式元素中可以包含任何符合Java语言规范的表达式，但是不能使用分号来结束表达式。

    <%= 表达式 %>
    

示例：

       今天的日期是: <%= (new java.util.Date()).toLocaleString()%>
    

JSP的内置对象
--------

![](http://imageu.oss-cn-shenzhen.aliyuncs.com/noteimage/jsp%E9%9A%90%E5%90%AB%E5%AF%B9%E8%B1%A1.png) 注：使用的时候要放在代码块中执行