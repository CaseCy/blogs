---
title: " 获取properties配置文件\t\t"
tags:
  - 配置相关
url: 129.html
id: 129
comments: false
categories:
  - 配置相关
date: 2018-10-19 09:51:00
---
## 通过FileInputStream获取
注：FileInputStream是InputStream的子类

```java
File file = new File("src/main/resources/spring/test.properties");
Properties p = new Properties();
p.load(new FileInputStream(file));
String word = p.getProperty("word");
```

上面第三步也可以直接
    
```java
p.load(new FileInputStream("src/main/resources/spring/test.properties"));
```
路径问题：
这里的`src/main/resources/spring/test.properties`是相对路径，相对于工程的路径，也可以写绝对路径（加上盘符等等）。
如果写`/src/main/resources/spring/test.properties`表示工程所在盘符的下的这个路径，而不是工程路径下的
如下:

```java
    File file = new File("src/main/resources/spring/test.properties");
    System.out.println(file.getAbsolutePath());
    File file1 = new File("/src/main/resources/spring/test.properties");
    System.out.println(file1.getAbsoluteFile());
```
结果为：

    C:\Users\Administrator\IdeaProjects\project\src\main\resources\spring\test.properties
    C:\src\main\resources\spring\test.properties

注:

    p.load()可以传入的参数是Reader和InputStream
***
## 通过ClassLoader类加载器获取
```java
InputStream resourceAsStream = a.getClass().getClassLoader().getResourceAsStream("spring/test.properties");
Properties p = new Properties();
p.load(resourceAsStream);
String word = p.getProperty("word");
```
这个方法是从classpath目录下加载的文件，比较方便

***

## 通过currentThread当前线程获取
```java
InputStream resourceAsStream1 = Thread.currentThread().getContextClassLoader().getResourceAsStream("spring/test.properties");
    Properties p = new Properties();
    p.load(resourceAsStream1);
    String word = p.getProperty("word");
```

这个也是通过classpath拿到的文件

***

注：获取之后一定要`try catch finally` 关闭流资源