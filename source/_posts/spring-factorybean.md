---
title: " Spring FactoryBean\t\t"
tags:
  - spring
url: 32.html
id: 32
comments: false
categories:
  - spring
date: 2018-10-18 19:48:35
---
>首先要分辨BeanFactory 与 FactoryBean的区别， 两个名字很像，所以容易搞混
>BeanFactory： 以Factory结尾，表示它是一个工厂类，是用于管理Bean的一个工厂
>FactoryBean：以Bean结尾，表示它是一个Bean，不同于普通Bean的是：它是实现了`FactoryBean<T>`接口的Bean，根据该Bean的Id从BeanFactory中获取的实际上是FactoryBean的getObject()返回的对象，而不是FactoryBean本身， 如果要获取FactoryBean对象，可以在id前面加一个&符号来获取。

和代理模式很像
FactoryBean对象
```java
public class AppResultFactoryBean implements FactoryBean<AppResult> {
    private String info ;

    public AppResult getObject() throws Exception {
        String[] split = info.split(",");
        AppResult appResult = new AppResult(split[0],split[1],split[2]);
        return appResult;
    }

    public Class<AppResult> getObjectType() {
        return AppResult.class;
    }

    public boolean isSingleton() {
        return true;
    }

    public void setInfo(String info) {
        this.info = info;
    }
}
```
xml配置
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:task="http://www.springframework.org/schema/task"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
        http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
        
    <bean id="appResultFactoryBean" class="cn.ws.test.AppResultFactoryBean" autowire="default">
        <property name="info" value="1,2,1"/>
    </bean>
</beans>
```
获取对象

---
## 通过applicationContext获取对象

```java
//通过applicationContext注入可以正确拿到对象,但是通@AutoWired存在问题，拿到的是FactoryBean本身
ApplicationContext context  = new ClassPathXmlApplicationContext("classpath:spring/spring-beans.xml");
        Object tryBean = context.getBean("tryBean");
```

---
## 通过@Resource根据bean的id获取对象

```java
//通过bean的id拿到的对象，这里的对象是通过FacatoryBean拿到的
@Resource
AppResultInterface appResultFactoryBean;
```

---
## 通过@Autowired注入
注：如果在xml中配置了对应的FactoryBean就不需要在加`@Component`注解，要不然会报
`springframework.beans.factory.NoUniqueBeanDefinitionException`这个错
```java
    @Autowired
    AppResult appResult;
```
这里因为使用了FactoryBean所以注入的都是经过了FactoryBean处理的AppResult对象

---
## 通过纯注解的方式使用
这种方式不需要在xml里面配置，但是要注意注解的使用方式，要将`@Component`打在实现了FactoryBean的类上，然后在`@Autowired`的时候使用`getObject`返回的对象也可以实现
注：这种方式在getObject的对象上不用打`@Component`标签，要不然注入的时候是真实对象。

---
## 使用`@Resouce`+`@Component`实现
通过在实现FactoryBean的类上加上标签`@Component("")`指定这个加载到spring容器中的名字，然后通过`@Resouce(name = "")`到spring的容器中取到对应的Bean
```java
@Component("appResultFactoryBean")
public class AppResultFactoryBean implements FactoryBean<AppResult>,InitializingBean,ApplicationListener{
    private String info ;
    @Override
    public AppResult getObject() throws Exception {
        AppResult appResult = new AppResult("调用成功");
        return appResult;
    }
    @Override
    public Class<AppResult> getObjectType() {
        return AppResult.class;
    }
    @Override
    public boolean isSingleton() {
        return true;
    }

    public void setInfo(String info) {
        this.info = info;
    }
    @Override
    public void afterPropertiesSet() throws Exception {

    }

    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent) {

    }
}
```
获取对象：
```java
    @Resource(name = "appResultFactoryBean")
    AppResult appResult;
```
使用这种方式也可以注入成功，而且比较方便，在原生的AppResult上面也可以使用`@Component("")`注解，注入的时候用`@Autowired`或者不用name属性的`@Resouce`注入就可以拿到

---
## 调用顺序

    getObjectType 调用！
    getObjectType 调用！
    getObjectType 调用！
    isSingleton 调用！
    getObjectType 调用！
    isSingleton 调用！
    getObject 调用！
    {returnCode=1, returnMsg=2, details=1}
    isSingleton 调用！
    getObjectType 调用！