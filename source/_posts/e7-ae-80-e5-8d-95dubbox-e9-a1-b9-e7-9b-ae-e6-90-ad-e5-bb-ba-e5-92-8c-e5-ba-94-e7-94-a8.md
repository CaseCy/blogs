---
title: " 简单dubbox项目搭建和应用\t\t"
tags:
  - 杂
url: 123.html
id: 123
comments: false
categories:
  - 杂
date: 2018-10-19 09:47:21
---

tags： dubbo

* * *

了解
--

认识dubbo和dubbox可以参考： [Dubbo架构设计详解](http://shiyanjun.cn/archives/325.html) [Dubbo详细介绍与安装使用过程](http://blog.csdn.net/xlgen157387/article/details/51865289) [Dubbo入门](http://blog.csdn.net/noaman_wgs/article/details/70214612) dubbo服务管理中心 用来管理，查询发布的服务，和dubbo本身的业务逻辑没有关系，是用来管理消费者和服务方的 dubbo

* * *

下载安装dubbox
----------

到[githug](https://github.com/dangdangdotcom/dubbox)下载dubbox的源码，

> 在checkout出来的dubbox目录执行mvn install -Dmaven.test.skip=true来尝试编译一下dubbo（并将dubbo的jar安装到本地maven库） 在checkout出来的dubbox根目录执行mvn idea:idea或者mvn eclipse:eclipse，来创建IDE工程文件

具体可以参考：[demo](https://dangdangdotcom.github.io/dubbox/demo.html) 上面引用至dubbox的github，除此之外可以将项目同步到本地，用ide工具的maven项目执行Install根项目（parent项目）来实现第二步，idea和eclipse都有这个功能 将jar包安装到本地之后就可以在项目中引用了

* * *

使用dubbo搭建RPC项目
--------------

RPC的目的就是为了远程调用服务，所以dubbo在使用的时候有两个角色，一个是服务的提供方，一个是服务的消费方，提供方提供接口和接口的实现，消费方通过接口消费服务。

### 准备

一个web server作为服务的提供者（也可以不是web项目，通过main方法也可以）, 一个项目作为服务的消费者 一个定义接口的项目打成jar包供消费者和提供者调用 下载[zookper](http://zookeeper.apache.org/releases.html)作为服务的注册中心

### 启动zookeeper

#### window

下载下来之后可以在`/conf`下将`zoo_sample.cfg`复制一份，然后更改名字为`zoo.cfg`（也可以根据需要去配置zookeeper，配置参照[百度](https://www.baidu.com)）然后在`/bin`目录下点击`zkServer.cmd`启动zookeeper。 注：`conf`目录下没有`zoo.cfg`启动的时候会出现闪退。

#### Linux

### 在项目中使用

下载完install后导入

            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>dubbo</artifactId>
                <version>2.8.4</version>
            </dependency>
            <!--这个是zookeeper的东西，如果不引入会报错：
            java.lang.NoClassDefFoundError: org/I0Itec/zkclient/IZkStateListener
            -->
            <dependency>
                <groupId>com.101tec</groupId>
                <artifactId>zkclient</artifactId>
                <version>0.4</version>
            </dependency>
    

如果需要RESTFul风格的可以参照[githug](https://github.com/dangdangdotcom/dubbox)查看需要导入的包 除此之外还要导入spring的包，这里不赘述 公共接口类(这里取名remote-api)中的接口

    public interface RemoteService {
    
        String sayHello(String word);
    }
    

提供方的实现类

    public class RemoteServiceImpl implements RemoteService{
    
        @Override
        public String sayHello(String word){
            return "Helllo"+word;
        }
    }
    

消费方只需要引入接口就行了

#### spring配置文件

配置文件可以参考：[Dubbo配置方式详解](http://www.cnblogs.com/chanshuyi/p/5144288.html) 　　　　　　　　　[Dubboo配置方式详解](http://www.cnblogs.com/linjiqin/p/5859153.html) 提供方

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
        <!--定义了提供方应用信息，用于计算依赖关系；在 dubbo-admin 或 dubbo-monitor 会显示这个名字，方便辨识-->
        <dubbo:application name="provider" owner="programmer" organization="dubbox"/>
        <!--使用 zookeeper 注册中心暴露服务，注意要先开启 zookeeper-->
        <dubbo:registry address="zookeeper://localhost:2181"/>
        <!-- 用dubbo协议在20880端口暴露服务 -->
        <dubbo:protocol name="dubbo" port="20880" />
        <!--使用 dubbo 协议实现定义好的 api.PermissionService 接口-->
        <dubbo:service interface="cn.cases.remote.RemoteService" ref="remoteService" protocol="dubbo" />
        <!--具体实现该接口的 bean-->
        <bean id="remoteService" class="RemoteServiceImpl"/>
    </beans>
    

消费方

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
           http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    
        <dubbo:application name="consumer" owner="programmer" organization="dubbox"/>
        <!--向 zookeeper 订阅 provider 的地址，由 zookeeper 定时推送-->
        <dubbo:registry address="zookeeper://localhost:2181"/>
        <!--使用 dubbo 协议调用定义好的 Service 接口-->
        <dubbo:reference id="remoteService" interface="cn.cases.remote.RemoteService"/>
    </beans>
    

#### 调用

消费方和服务方我都是起的web项目，在web.xml引入spring的Listener之后，分别在不同的端口启动，在消费方执行下面的测试用例

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration("classpath:spring/spring-dubbo.xml")
    public class RemoteTest {
        @Autowired
        RemoteService remoteService;
    
        @Test
        public void test1 (){
            String word = remoteService.sayHello("张三");
            System.out.println(word);
        }
    }
    --------------------------------------------------------------------
    //也可以
    public class RemoteTest {
    
        @Test
        public void test1 (){
            ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring/spring-dubbo.xml");
            RemoteService remoteService = (RemoteService)context.getBean("remoteService");
            String word = remoteService.sayHello("张三");
            System.out.println(word);
        }
    }
    

调用成功

    Helllo张三
    

用main方法调用，不用servlet容器，启动之后就可以在客户端调用

    public static void main(String[] args) throws IOException {
            ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring/spring-dubbo.xml");
            context.start();
            System.in.read();
        }
    

注：如果使用外网连接`zookeeper`出现一直超时的问题，可以更改`<dubbo:registry protocol="zookeeper" address="zookeeper://127.0.0.1:2181" timeout="10000"/>`将超时时间设置长点就可以了，如果依旧超时，参考[内网ip问题](http://blog.csdn.net/xlgen157387/article/details/52702659) 至此基本的dubbo使用就完了。

* * *

Dubbo client
------------

Dubbo client是用来管理和查看服务的消费和提供情况的，将下载下来的dubbo项目install后，在dubbo-admin下将war拷贝到tomcat的webapps下，启动tomcat，访问localhost+端口号+项目名就可以登录了（默认密码用户名都是root）。