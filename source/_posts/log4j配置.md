---
title: " log4j配置\t\t"
tags:
  - 配置相关
url: 144.html
id: 144
comments: false
categories:
  - 配置相关
date: 2018-10-19 10:23:08
---

引用：[log4j配置](http://www.cnblogs.com/ITtangtang/p/3926665.html)
参考：[玩转log4j](http://www.cnblogs.com/shenliang123/archive/2012/05/02/2479286.html)

---
## 组成部分
log4j由三个部分组成
    
    Loggers(记录器)，Appenders (输出源)和Layouts(布局)
log4j的日志被分为了几个级别，对应的级别及关系：
    
    DEBUG < INFO < WARN < ERROR < FATAL
注：log4j输出日志的时候，只会输出和设定等级相等和比设定等级高的Log
### 常用的Logger

    org.apache.log4j.ConsoleAppender（控制台）
    org.apache.log4j.FileAppender（文件）
    org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件）
    org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件）
    org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）
### 常用的Layout

    org.apache.log4j.HTMLLayout（以HTML表格形式布局）
    org.apache.log4j.PatternLayout（可以灵活地指定布局模式）
    org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串）
    org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等信息）
***
## 配置及语法
Log4j支持两种配置文件格式，一种是XML格式的文件，一种是properties属性文件，这里使用properites
### 配置根Logger

    log4j.rootLogger = [ level ] , appenderName1, appenderName2, …
    log4j.additivity.org.apache=false：表示Logger不会在父Logger的appender里输出，默认为true。
这里的appenderName可以自己定义，但是要在后面的配置中也使用
level表示输出日志的等级，log4j只建议使用ERROR、WARN、INFO、DEBUG（从高到低）
使用范例：

    lo4j.rootLoger = DEBUG,a,b,c

### 配置日志输出位置(appender)

    log4j.appender.appenderName = className
`className对应上面几种常用的Logger,appenderName对应根Logger的appenderName,以下不赘述`
范例：

    log4j.appender.a = org.apache.log4j.ConsoleAppender
几种Logger都有的属性

    Threshold=WARN：指定日志信息的最低输出级别，默认为DEBUG。
    ImmediateFlush=true：表示所有消息都会被立即输出，设为false则不输出，默认值是true。
FileAppender，DailyRollingFileAppender，RollingFileAppender都有的属性

    Append=false：true表示消息增加到指定文件中，false则将消息覆盖指定的文件内容，默认值是true。
    File=D:/logs/logging.log4j：指定消息输出到logging.log4j文件中。
特有属性：

 - `ConsoleAppender`
    Target=System.err：默认值是System.out
 - `DailyRollingFileAppender`
    DatePattern='.'yyyy-MM：每月滚动一次日志文件，即每月产生一个新的日志文件。当前月的日志文件名为logging.log4j，前一个月的日志文件名为logging.log4j.yyyy-MM。
    另外，也可以指定按周、天、时、分等来滚动日志文件，对应的格式如下：
    1)'.'yyyy-MM：每月
    2)'.'yyyy-ww：每周
    3)'.'yyyy-MM-dd：每天
    4)'.'yyyy-MM-dd-a：每天两次
    5)'.'yyyy-MM-dd-HH：每小时
    6)'.'yyyy-MM-dd-HH-mm：每分钟
 - `RollingFileAppender`
    MaxFileSize=100KB：后缀可以是KB,MB或者GB。在日志文件到达该大小时，将会自动滚动，即将原来的内容移到logging.log4j.1文件中。
    MaxBackupIndex=2：指定可以产生的滚动文件的最大数，例如，设为2则可以产生logging.log4j.1，logging.log4j.2两个滚动文件和一个logging.log4j文件。

范例：

    log4j.appender.a.Target = System.err
    log4j.appender.a.Threshold = DEBUG

### 配置输出的形式/格式（Layout）

    log4j.appender.appenderName.layout=className
这里的className指的是上面常用的Layout，以下不赘述
参数设置：

 - `HTMLLayout`
    LocationInfo=true：输出java文件名称和行号，默认值是false。
     Title=My Logging： 默认值是Log4J Log Messages。
 - `PatternLayout`
    ConversionPattern=%m%n：设定以怎样的格式显示消息。
```
格式化符号说明：

%p：输出日志信息的优先级，即DEBUG，INFO，WARN，ERROR，FATAL。
%d：输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，如：%d{yyyy/MM/dd HH:mm:ss,SSS}。
%r：输出自应用程序启动到输出该log信息耗费的毫秒数。
%t：输出产生该日志事件的线程名。
%l：输出日志事件的发生位置，相当于%c.%M(%F:%L)的组合，包括类全名、方法、文件名以及在代码中的行数。例如：test.TestLog4j.main(TestLog4j.java:10)。
%c：输出日志信息所属的类目，通常就是所在类的全名。
%M：输出产生日志信息的方法名。
%F：输出日志消息产生时所在的文件名称。
%L:：输出代码中的行号。
%m:：输出代码中指定的具体日志信息。
%n：输出一个回车换行符，Windows平台为"rn"，Unix平台为"n"。
%x：输出和当前线程相关联的NDC(嵌套诊断环境)，尤其用到像java servlets这样的多客户多线程的应用中。
%%：输出一个"%"字符。
另外，还可以在%与格式字符之间加上修饰符来控制其最小长度、最大长度、和文本的对齐方式。如：
1) c：指定输出category的名称，最小的长度是20，如果category的名称长度小于20的话，默认的情况下右对齐。
2)%-20c："-"号表示左对齐。
3)%.30c：指定输出category的名称，最大的长度是30，如果category的名称长度大于30的话，就会将左边多出的字符截掉，但小于30的话也不会补空格。
```
范例:

    log4j.appender.c.layout=org.apache.log4j.PatternLayout
    log4j.appender.c.layout.ConversionPattern=[%-5p] [%d{yyyy-MM-dd HH:mm:ss}][%t] %c - %x %m%n
## 其他
### 特定类输出不同个log
在Java类中说明

    private static Log logger = LogFactory.getLog(Test.class);
说明：这里的LogFactory是`commons-logging`包中的，这个包不是Log4j的，是一个日志通用包，Log4j实现了其中的接口，这里通过LogFactory来获得日志对象，以后更改的时候会比较方便
然后在log4j.properties中加入:

    log4j.logger.cn.com.Test= DEBUG, test
    log4j.appender.test=org.apache.log4j.FileAppender
    log4j.appender.test.File=${myweb.root}/WEB-INF/log/test.log
    log4j.appender.test.layout=org.apache.log4j.PatternLayout
    log4j.appender.test.layout.ConversionPattern=%d %p [%c] - %m%n
### 同一个类输出多个log
Java类中：

    private static Log logger = LogFactory.getLog("test");
    private static Log logger = LogFactory.getLog("test1");
log4j.properties中加入:

    log4j.logger.test= DEBUG, test
    ...配置信息略去
    
    log4j.logger.test1= DEBUG, test1
    ...配置信息略去

> 这些自定义的日志默认是同时输出到log4j.rootLogger所配置的日志中的，通过设置下面的属性可以更改
> log4j.additivity.test1 = false
> 　　它用来设置是否同时输出到log4j.rootLogger所配置的日志中，设为false就不会输出到其它地方啦注意这里的"test1"是在程序中给logger起的那个自定义的名字！
> 如果你说，我只是不想同时输出这个日志到log4j.rootLogger所配置的logfile中，stdout里我还想同时输出呢！那也好办，把你的log4j.logger.myTest1 = DEBUG, test1改为下式就OK啦！ log4j.logger.test1=DEBUG, test1

## 使用
在`src`等级的目录下新建文件`log4j.properties`将配置写入就可以使用了
***
```
#设置根级别
#a为控制台输出。b为日志输出。c为错误日志
#DEBUG < INFO < WARN < ERROR < FATAL
log4j.rootLogger = DEBUG,a,c
################################
#设置每个日志输出的等级
log4j.appender.a.Threshold = DEBUG
#log4j.appender.b.Threshold = INFO
log4j.appender.c.Threshold = ERROR
################################
#设置spring的log级别
log4j.logger.org.springframework=ERROR
################################
#输出
log4j.appender.a = org.apache.log4j.ConsoleAppender
#Target输出的格式。可以写System.err和System.out(默认)
log4j.appender.a.Target = System.out
#指定输出的格式
log4j.appender.a.layout =org.apache.log4j.PatternLayout
log4j.appender.a.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
###############################
#输出到日志（文件）
log4j.appender.c = org.apache.log4j.DailyRollingFileAppender
log4j.appender.c.File = ${log.dir}/error.log
#指定输出的格式
log4j.appender.c.layout =org.apache.log4j.PatternLayout
log4j.appender.c.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
##################################
```