---
title: " Spring boot注解\t\t"
tags:
  - spring
url: 395.html
id: 395
comments: false
categories:
  - spring boot
date: 2019-04-04 10:25:22
---

记一些springboot中用的较少但可能会用到的注解

    //读取以te开头的配置文件
    @ConfigurationProperties("te")
    //加载自定义配置文件到spring上下文
    @PropertySource()
    //包扫描，默认是扫描当前包下的所有包
    @ComponentScan(excludeFilters = {@ComponentScan.Filter(pattern = "^[C]",type = FilterType.REGEX)})
    //一个接口有多个实现类时，优先使用有该注解的类
    @Primary
    //避免注入歧义
    @Qualifier
    
    /*
    条件装配Bean
    */
    //使用一个实现了Condition接口的类作为判断是否注入Bean
    @Conditional
    
    //根据配置active的配置文件来配置决定是否加载Bean，没配置active或default，这个Bean不会被加载
    //JAVA_OPTS="-Dspring.profiles.active=dev"
    @Profile
    //可以从xml中加载Bean
    @ImportResource
    //指定Bean的Class，将其加载到容器中
    @Import
    
    /*
    Spring EL表达式
    */
    //代表占位符，它会读取山下文属性装配到属性中
    ${...}
    //T（...）代表的是引入类，如果不是Java自身的类要写全限定名
    #{T(Sytem).currentTimeMillis()}
    //可以调用容器中的其他Bean的属性给当前属性赋值，beanName是容器中Bean的名称
    #{beanName.str}
    //这里问号是判断是否为空，如果不为空才会执行后面的方法
    #{beanName.str?.toUpperCase()}
    //还可以使用EL表达式进行算术运算，字符串比较，字符串连接，三位运算
    @Value("#{beanName.str eq 'Spring Boot'}")//字符串比较
    
    //给一个类增加新的方法（aop）
    /*
     eg:
     @DeclareParents(value = "com.springboot2.x.learn.lang.CatService",defaultImpl = CatValidatorImpl.class)
        public CatValidator catValidator;
     CatService如果是一个接口，可跟一个+号，表示所有实现类
    */
    @DeclareParents
    
    //mybatis的注解，指定别名
    @Alias