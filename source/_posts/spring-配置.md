---
title: " Spring 配置\t\t"
tags:
  - spring
url: 112.html
id: 112
comments: false
categories:
  - spring
date: 2018-10-19 09:38:49
---
引用：[property-placeholder/override](http://blog.csdn.net/chyohn/article/details/54945777)
　　　[读取property文件的方式](http://www.cnblogs.com/hafiz/p/5876243.html)

---

## Context
```xml
    <!--注解支持-->
    <context:annotation-config />
    <!--扫描的包-->
    <context:component-scan base-package="cn.ws.test"/>
    <context:component-scan base-package="cn.ws.handle">
        <context:exclude-filter type="regex" expression=".."/>
        <context:include-filter type="annotation" expression="cn.ws.annotation.AccessCheck"/>
    </context:component-scan>
```
这里`component-scan`中包含两个属性，一个是`exclude-filter`不包含
`include-filter`包含
里面的`type`属性对应有四种

 - `annotation`
    表示过滤方式为注解，含有后面expression注解的都会被过滤掉
     expression示例：`cn.ws.annotation.AccessCheck`
 - `regex`
    以正则表达式的方式过滤
     expression示例：com.\ws\\..*
 - `assignable`
    以继承和实现的方式过滤，所有继承或者实现了后面expression的都会被过滤
    expression示例：`java.util.HashMap`
 - `aspectj`
    以Aspectj表达式的方式过滤
     expression示例：
 - `custom`
    自定义TypeFilter过滤规则，写一个类，这个类必须实现`org.springframework.core.type.TypeFilter`接口
     expression示例：`cn.ws.MyTypeFilter`
```xml
    <context:property-override file-encoding="utf-8" ignore-unresolvable="true" location="classpath:spring/test.properties"/>
    <context:property-placeholder location="classpath:spring/test.properties" file-encoding="utf-8" ignore-unresolvable="true"/>
```
`property-placeholder`可以读取properties文件
另外通过下面的方式也可以读取
```xml
<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
   <property name="ignoreUnresolvablePlaceholders" value="true"/>
   <property name="locations">
      <list>
         <value>classpath:test.properties</value>
         <value>classpath:test1.properties</value>
      </list>
    </property>
</bean>
```

> 注意：这种方式下，如果你在spring-mvc.xml文件中有如下配置，则一定不能缺少下面的红色部分，[原因](http://www.cnblogs.com/hafiz/p/5875770.html)

详细参考：[spring获取property](http://www.cnblogs.com/hafiz/p/5876243.html)