---
title: " Tomcat优化\t\t"
tags:
  - 杂
url: 114.html
id: 114
comments: false
categories:
  - 杂
date: 2018-10-19 09:40:27
---

tags： Tomcat 优化 引用：[Tomcat优化](http://blog.csdn.net/u012562943/article/details/51441157)

* * *

内存优化
----

### 优化

出现下面情况的时候，说明tomcat使用的内存不能支持访问了，需要进行内存优化

    严重: Exception invoking periodic operation: java.lang.OutOfMemoryError: Java heap space  
    严重: Error processing request java.lang.OutOfMemoryError: GC overhead limit exceeded  
    

修改tomcat目录下`/bin/catalina.sh(Linux)`或者`/bin/catalina.bat(Windows)`文件 Linux下修改，可以放在CLASSPATH=下面：

    JAVA_OPTS="-server -XX:PermSize=512M -XX:MaxPermSize=1024m -Xms2048m -Xmx2048m"
    

windows下修改，可以放在set CLASSPATH=下面：

    set JAVA_OPTS=-server -XX:PermSize=512M -XX:MaxPermSize=1024m -Xms2048m -Xmx2048m
    

参数配置如下：

    -server：启用 JDK的 server 版本；
    -Xms：Java虚拟机初始化时堆的最小内存，一般与Xmx配置为相同值，这样的好处是GC不必再为扩展内存空间而消耗性能；
    -Xmx：Java虚拟机可使用堆的最大内存；
    -XX:PermSize：Java虚拟机永久代大小；
    -XX:MaxPermSize：Java虚拟机永久代大小最大值；
    

除了这些参数外还可以根据具体需要配置其他参数，参数的配置可以参考JVM参数的配置

### 验证

使用JAVA_HOME/bin目录下的两个工具

    jps：用来显示本地的java进程，以及进程号，进程启动的路径等。
    jmap：观察运行中的JVM 物理内存的占用情况，包括Heap size , Perm size等。
    

配置了环境变量可以直接打开cmd输入jps(没有先跳转到JAVA_HOME/bin)，会出现下面的情况

    6112 BootStrap
    6640 Jre
    856 RemoteMavenServer
    

这里的BootStrap对应的就是tomcat,6112是进程号 输入`jmap -heap 6112`查看 `PermSize //永久代大小`

    Attaching to process ID 6112, please wait...
    Debugger attached successfully.
    Server compiler detected.
    JVM version is 25.131-b11
    
    using thread-local object allocation.
    Parallel GC with 4 thread(s)
    
    Heap Configuration:
       MinHeapFreeRatio         = 0
       MaxHeapFreeRatio         = 100
       MaxHeapSize              = 2147483648 (2048.0MB) //最大堆内存
       NewSize                  = 715653120 (682.5MB)
       MaxNewSize               = 715653120 (682.5MB)
       OldSize                  = 1431830528 (1365.5MB)
       NewRatio                 = 2
       SurvivorRatio            = 8
       MetaspaceSize            = 21807104 (20.796875MB)
       CompressedClassSpaceSize = 1073741824 (1024.0MB)
       MaxMetaspaceSize         = 17592186044415 MB
       G1HeapRegionSize         = 0 (0.0MB)
    
    Heap Usage:
    PS Young Generation
    Eden Space:
       capacity = 537395200 (512.5MB)
       used     = 247226456 (235.77352142333984MB)
       free     = 290168744 (276.72647857666016MB)
       46.00458954601753% used
    From Space:
       capacity = 89128960 (85.0MB)
       used     = 0 (0.0MB)
       free     = 89128960 (85.0MB)
       0.0% used
    To Space:
       capacity = 89128960 (85.0MB)
       used     = 0 (0.0MB)
       free     = 89128960 (85.0MB)
       0.0% used
    PS Old Generation
       capacity = 1431830528 (1365.5MB)
       used     = 0 (0.0MB)
       free     = 1431830528 (1365.5MB)
       0.0% used
    
    12256 interned Strings occupying 1742568 bytes.
    
    

* * *

配置优化
----

通过`server.xml`来优化配置

### Connector连接器优化

    <Connector port="80" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443" />
    

设置官方参考[Tomcat8.5配置](http://tomcat.apache.org/tomcat-8.5-doc/config/http.html) 常用的一些属性（以下引用自[Tomcat优化](http://blog.csdn.net/u012562943/article/details/51441157)）：

> `port`：代表Tomcat监听端口，也就是网站的访问端口，默认为8080，可以根据需要改成其他 `protocol`：协议类型，可选类型有四种，分别为BIO（阻塞型IO），NIO，NIO2和APR，Tomcat在默认情况下，是以bio模式运行的。遗憾的是，就一般而言，bio模式是三种运行模式中性能最低的一种。BIO配置采用默认即可 `maxThreads`：由该连接器创建的处理请求线程的最大数目，也就是可以处理的同时请求的最大数目。如果未配置默认值为200。如果一个执行器与此连接器关联，则忽略此属性，因为该属性将被忽略，所以该连接器将使用执行器而不是一个内部线程池来执行任务。 `minSpareThreads`：线程的最小运行数目，这些始终保持运行。如果未指定，默认值为10。 `acceptCount`：当所有可能的请求处理线程都在使用时传入连接请求的最大队列长度。如果未指定，默认值为100。一般是设置的跟maxThreads一样或一半，此值设置的过大会导致排队的请求超时而未被处理。所以这个值应该是主要根据应用的访问峰值与平均值来权衡配置。 `maxConnections`：在任何给定的时间内，服务器将接受和处理的最大连接数。当这个数字已经达到时，服务器将接受但不处理，等待进一步连接。NIO与NIO2的默认值为10000，APR默认值为8192。 `connectionTimeout`：当请求已经被接受，但未被处理，也就是等待中的超时时间。单位为毫秒，默认值为60000。通常情况下设置为30000。 `maxHttpHeaderSize`：请求和响应的HTTP头的最大大小，以字节为单位指定。如果没有指定，这个属性被设置为8192（8 KB）。 `tcpNoDelay`：如果为true，服务器socket会设置TCP\_NO\_DELAY选项，在大多数情况下可以提高性能。缺省情况下设为true。 `compression`：是否启用gzip压缩，默认为关闭状态。这个参数的可接受值为“off”（不使用压缩），“on”（压缩文本数据），“force”（在所有的情况下强制压缩）。 `compressionMinSize`：如果compression="on"，则启用此项。被压缩前数据的最小值，也就是超过这个值后才被压缩。如果没有指定，这个属性默认为“2048”（2K），单位为byte。 `disableUploadTimeout`：这个标志允许servletContainer在一个servlet执行的时候，使用一个不同的，更长的连接超时。最终的结果是给servlet更长的时间以便完成其执行，或者在数据上载的时候更长的超时时间。如果没有指定，设为false。 `enableLookups`：关闭DNS反向查询。 `URIEncoding`：URL编码字符集。

    <Connector port="8080"   
              protocol="HTTP/1.1"   
              maxThreads="1000"   
              minSpareThreads="100"   
              acceptCount="1000"  
              maxConnections="1000"  
              connectionTimeout="20000"   
              maxHttpHeaderSize="8192"  
              tcpNoDelay="true"  
              compression="on"  
              compressionMinSize="2048"  
              disableUploadTimeout="true"  
              redirectPort="8443"  
          enableLookups="false"  
              URIEncoding="UTF-8" />
    

    compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain"
    

* * *

### 线程池

    <Executor name="tomcatThreadPool"   
             namePrefix="catalina-exec-"   
             maxThreads="1000"   
             minSpareThreads="100"  
             maxIdleTime="60000"  
             maxQueueSize="Integer.MAX_VALUE"  
             prestartminSpareThreads="false"  
             threadPriority="5"  
             className="org.apache.catalina.core.StandardThreadExecutor"/>
    

> `name`：线程池名称，用于 Connector中指定。 namePrefix：所创建的每个线程的名称前缀，一个单独的线程名称为 namePrefix+threadNumber。 `maxThreads`：池中最大线程数。 `minSpareThreads`：活跃线程数，也就是核心池线程数，这些线程不会被销毁，会一直存在。 `maxIdleTime`：线程空闲时间，超过该时间后，空闲线程会被销毁，默认值为6000（1分钟），单位毫秒。 `maxQueueSize`：在被执行前最大线程排队数目，默认为Int的最大值，也就是广义的无限。除非特殊情况，这个值不需要更改，否则会有请求不会被处理的情况发生。 `prestartminSpareThreads`：启动线程池时是否启动 minSpareThreads部分线程。默认值为false，即不启动。 `threadPriority`：线程池中线程优先级，默认值为5，值从1到10。 `className`：线程池实现类，未指定情况下，默认实现类为org.apache.catalina.core.StandardThreadExecutor。如果想使用自定义线程池首先需要实现`org.apache.catalina.Executor`接口。

线程池配置完成后需要在 Connector中指定：

    <Connector executor="tomcatThreadPool"
    ...
    

* * *

组件优化
----

### APR

> 概念:APR(Apache Portable Runtime)是一个高可移植库，它是Apache HTTP Server 2.x的核心。APR有很多用途，包括访问高级 IO功能(例如sendfile,epoll和OpenSSL)，OS级别功能(随机数生成，系统状态等等)，本地进程管理(共享内存，NT管道和UNIX sockets)。这些功能可以使Tomcat作为一个通常的前台WEB服务器，能更好地和其它本地web技术集成，总体上让Java更有效率作为一个高性能web服务器平台而不是简单作为后台容器。

下载安装参考引用内容

性能测试
----

使用压力测试工具做性能测试。如：Jmeter