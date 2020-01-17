---
title: "WebService"
tags:
  - 杂
url: 116.html
id: 116
comments: false
categories:
  - spring
date: 2018-10-19 09:42:07
---

tags： WebService

* * *

概念： Web service是一个平台独立的，低耦合的，自包含的、基于可编程的web的应用程序，可使用开放的XML（标准通用标记语言下的一个子集）标准来描述、发布、发现、协调和配置这些应用程序，用于开发分布式的互操作的应用程序。 ![webservice原理](http://imageu.oss-cn-shenzhen.aliyuncs.com/noteimage/webservice%E5%8E%9F%E7%90%86.png) webservice用到的技术主要有XML+XSD，SOAP和WSDL，XML Schema(XSD)定义了一套标准的数据类型，并给出了一种语言来扩展这套数据类型 完整的流程如下 客户端——> 阅读WSDL文档 (根据文档生成SOAP请求) ——>发送到Web服务器——>交给WebService请求处理器 （ISAPI Extension） ——>处理SOAP请求——> 调用WebService——>生成SOAP应答 ——> Web服务器通过http的方式交给客户端

* * *

了解
--

### WSDL

WSDL(Web Servie Description Language)是一个基于xml的web service描述语言

> WSDL 是一种使用 XML 编写的文档。这种文档可描述某个 Web service。它可规定服务的位置，以及此服务提供的操作（或方法）。

WSDL通过下面几个参数描述webservice | 元素 | 定义 | | --------- | -------------------------- | | | web service 使用的消息 | | | web service 使用的数据类型 | | | web service 使用的通信协议 | ![wsdl](http://imageu.oss-cn-shenzhen.aliyuncs.com/noteimage/wsdl.png)

### SOAP协议

SOAP(Simple Object Access Protocol)基于xml和HTTP的简单对象交换协议 webService三要素

> SOAP、WSDL(WebServicesDescriptionLanguage)、UDDI(UniversalDescriptionDiscovery andIntegration)之一， soap用来描述传递信息的格式， WSDL 用来描述如何访问具体的接口， uddi用来管理，分发，查询webService 。具体实现可以搜索 Web Services简单实例 ; SOAP 可以和现存的许多因特网协议和格式结合使用，包括超文本传输协议（HTTP），简单邮件传输协议（SMTP），多用途网际邮件扩充协议（MIME）。它还支持从消息系统到远程过程调用（RPC）等大量的应用程序。SOAP使用基于XML的数据结构和超文本传输协议(HTTP)的组合定义了一个标准的方法来使用Internet上各种不同操作环境中的分布式对象。

webservice的开发主要是服务端和客户端，服务端发布服务，客户端调用服务。

### UDDI注册中心

> UDDI 是一种目录服务，企业可以使用它对 Web services 进行注册和搜索。 UDDI，英文为 "Universal Description, Discovery and Integration"，可译为“通用描述、发现与集成服务”。 UDDI 是一个独立于平台的框架，用于通过使用 Internet 来描述服务，发现企业，并对企业服务进行集成

简而言之就是一个webservice的注册中心，当需要一个webservice的服务的时候可以到UDDI去寻找，并调用对应的服务。也可以直接告诉webservice的地址就可以调用了

* * *

发布前准备
-----

一个用于WebService的类

    import javax.jws.WebMethod;
    import javax.jws.WebParam;
    import javax.jws.WebService;
    import javax.jws.soap.SOAPBinding;
    import javax.xml.ws.BindingType;
    
    @WebService(targetNamespace = Constants.NSPACE)
    @BindingType()
    public interface HelloCxfInterface {
        //@WebService是必须的；@WebParam不是必须的。
        //如果没有@WebParam的描述，在wsdl文件内描述的方法中，参数名将变为arg0,arg1…以此类推.
        @SOAPBinding(use = SOAPBinding.Use.ENCODED,style = SOAPBinding.Style.RPC,parameterStyle = SOAPBinding.ParameterStyle.BARE)
        @WebMethod(action = "www.baidu.com",operationName = "say")
        public String sayHello(@WebParam(name = "name") String people);
    
        @SOAPBinding(use = SOAPBinding.Use.ENCODED,style = SOAPBinding.Style.RPC)
        @WebMethod(action = "www.baidu.com",operationName = "getUser")
        public UserDTO getUser(String userId);
    }


    @WebService(endpointInterface = "cn.cases.webservice.beans.HelloCxfInterface",//<wsdl:portType name="HelloCxfInterface">
            serviceName="HelloService",//<wsdl:portType name="HelloService">
            targetNamespace = Constants.NSPACE,
            name = "Hello")//<wsdl:port binding="tns:HelloServiceSoapBinding" name="HelloPort">
    public class HelloCxf implements HelloCxfInterface{
    
        @Override
        public String sayHello(String people){
            return "Hello "+people;
        }
    
        @Override
        public UserDTO getUser(String userId) {
            UserDTO dto = new UserDTO();
            dto.setId(Long.parseLong("1001"));
            dto.setLoginName("safdas");
            dto.setName("张三");
            dto.setEmail("sdasf@qq.com");
            return dto;
        }
    
        public String getOne(){
            return "One";
        }
    }

* * *

发布WebService
------------

关于wsdl中的soap消息绑定机制，可以参考：[SOAP消息绑定机制](http://blog.csdn.net/seasky323/article/details/6746174)，[浅谈SOAP](https://www.ibm.com/developerworks/cn/xml/x-sisoap/)

### 使用Java原生的WebService

参考：[Web Service开篇](http://www.cnblogs.com/decarl/archive/2012/05/15/2502074.html)

    public class JaxWsServerListener {
        private static final Log log = LogFactory.getLog(JaxWsServerListener.class);
    
        public static void main(String[] args) {
            CxfService cxfService = new CxfServiceImpl();
            String address = "http://localhost/hello";
    
            Endpoint.publish(address,cxfService);
            log.info("暴露成功！");
        }
    }


### 使用Axis1.x发布

参考：[Axis1.x WebService开发指南](http://www.cnblogs.com/hoojo/archive/2010/12/20/1911363.html) Axis有两种方式发布，一种是`jws`方式，一种是`wsdd`方式 第一种方式操作简单，但是一般不怎么用，可以参考：[使用jws方式发布](http://www.cnblogs.com/hoojo/archive/2010/12/20/1911357.html) 这里介绍wsdd方式 导入pom.xml

            <dependency>
                <groupId>axis</groupId>
                <artifactId>axis</artifactId>
                <version>1.4</version>
            </dependency>


在web.xml中配置

        <servlet>
            <display-name>Apache-Axis Servlet</display-name>
            <servlet-name>AxisServlet</servlet-name>
            <servlet-class>org.apache.axis.transport.http.AxisServlet</servlet-class>
        </servlet>
        <servlet>
            <display-name>Axis Admin Servlet</display-name>
            <servlet-name>AdminServlet</servlet-name>
            <servlet-class>org.apache.axis.transport.http.AdminServlet</servlet-class>
            <load-on-startup>100</load-on-startup>
        </servlet>
        <servlet>
            <display-name>SOAPMonitorService</display-name>
            <servlet-name>SOAPMonitorService</servlet-name>
            <servlet-class>org.apache.axis.monitor.SOAPMonitorService</servlet-class>
            <init-param>
                <param-name>SOAPMonitorPort</param-name>
                <param-value>5101</param-value>
            </init-param>
            <load-on-startup>100</load-on-startup>
        </servlet>
        <servlet-mapping>
            <servlet-name>AxisServlet</servlet-name>
            <url-pattern>/servlet/AxisServlet</url-pattern>
        </servlet-mapping>
        <servlet-mapping>
            <servlet-name>AxisServlet</servlet-name>
            <url-pattern>*.jws</url-pattern>
        </servlet-mapping>
        <servlet-mapping>
            <servlet-name>AxisServlet</servlet-name>
            <url-pattern>/services/*</url-pattern>
        </servlet-mapping>
        <servlet-mapping>
            <servlet-name>SOAPMonitorService</servlet-name>
            <url-pattern>/SOAPMonitor</url-pattern>
        </servlet-mapping>
        <servlet-mapping>
            <servlet-name>AdminServlet</servlet-name>
            <url-pattern>/servlet/AdminServlet</url-pattern>
        </servlet-mapping>
        <mime-mapping>
            <extension>wsdl</extension>
            <mime-type>text/xml</mime-type>
        </mime-mapping>


在`WEB-INF`目录下创建一个`server-config.wsdd`文件，里面添加如下内容。

    <?xml version="1.0" encoding="UTF-8"?>
    <deployment xmlns="http://xml.apache.org/axis/wsdd/" xmlns:java="http://xml.apache.org/axis/wsdd/providers/java">
      <globalConfiguration>
        <parameter name="sendMultiRefs" value="true"/>
        <parameter name="disablePrettyXML" value="true"/>
        <parameter name="adminPassword" value="admin"/>
    
        <parameter name="dotNetSoapEncFix" value="true"/>
        <parameter name="enableNamespacePrefixOptimization" value="false"/>
        <parameter name="sendXMLDeclaration" value="true"/>
        <parameter name="sendXsiTypes" value="true"/>
        <parameter name="attachments.implementation" value="org.apache.axis.attachments.AttachmentsImpl"/>
        <requestFlow>
          <handler type="java:org.apache.axis.handlers.JWSHandler">
            <parameter name="scope" value="session"/>
          </handler>
          <handler type="java:org.apache.axis.handlers.JWSHandler">
            <parameter name="scope" value="request"/>
            <parameter name="extension" value=".jwr"/>
          </handler>
        </requestFlow>
      </globalConfiguration>
    
      <handler name="LocalResponder" type="java:org.apache.axis.transport.local.LocalResponder"/>
      <handler name="URLMapper" type="java:org.apache.axis.handlers.http.URLMapper"/>
      <handler name="Authenticate" type="java:org.apache.axis.handlers.SimpleAuthenticationHandler"/>
      <service name="AdminService" provider="java:MSG">
        <parameter name="allowedMethods" value="AdminService"/>
        <parameter name="enableRemoteAdmin" value="false"/>
        <parameter name="className" value="org.apache.axis.utils.Admin"/>
        <namespace>http://xml.apache.org/axis/wsdd/</namespace>
      </service>
    
      <service name="Version" provider="java:RPC">
        <parameter name="allowedMethods" value="getVersion"/>
        <parameter name="className" value="org.apache.axis.Version"/>
      </service>
      <service name="SOAPMonitorService" provider="java:RPC">
        <parameter name="allowedMethods" value="publishMessage"/>
        <parameter name="scope" value="Application"/>
        <parameter name="className" value="org.apache.axis.monitor.SOAPMonitorService"/>
      </service>
      <!--自己的webservice-->
      <service name="MyCService" provider="java:RPC" style="RPC" use="encoded">
        <parameter name="className" value="cn.ws.webservice.InfoService"/>
        <!--设置发布的方法，以空格分隔-->
        <parameter name="allowedMethods" value="Method1 Method2"/>
        <!--设置-->
        <parameter name="scope" value="request"/>
        <namespace>http://case.cn</namespace>
      </service>
    
      <handler name="soapmonitor" type="java:org.apache.axis.handlers.SOAPMonitorHandler">
        <parameter name="wsdlURL" value="/axis/SOAPMonitorService-impl.wsdl"/>
        <parameter name="serviceName" value="SOAPMonitorService"/>
        <parameter name="namespace" value="http://tempuri.org/wsdl/2001/12/SOAPMonitorService-impl.wsdl"/>
        <parameter name="portName" value="Demo"/>
     </handler>
      <transport name="http">
        <requestFlow>
          <handler type="URLMapper"/>
          <handler type="java:org.apache.axis.handlers.http.HTTPAuthHandler"/>
    
          <!--comment following line for REMOVING wsdl spying via SOAPMonitor-->
          <handler type="soapmonitor"/>
        </requestFlow>
        <responseFlow>
          <!--comment following line for REMOVING wsdl spying via SOAPMonitor-->
          <handler type="soapmonitor"/>
        </responseFlow>
        <parameter name="qs:list" value="org.apache.axis.transport.http.QSListHandler"/>
        <parameter name="qs:wsdl" value="org.apache.axis.transport.http.QSWSDLHandler"/>
        <parameter name="qs.list" value="org.apache.axis.transport.http.QSListHandler"/>
        <parameter name="qs.method" value="org.apache.axis.transport.http.QSMethodHandler"/>
        <parameter name="qs:method" value="org.apache.axis.transport.http.QSMethodHandler"/>
        <parameter name="qs.wsdl" value="org.apache.axis.transport.http.QSWSDLHandler"/>
      </transport>
      <transport name="local">
        <responseFlow>
          <handler type="LocalResponder"/>
        </responseFlow>
      </transport>
    </deployment>


上面这一步也可以使用axis提供的工具生成，具体操作可参考:[定制发布](http://blog.csdn.net/qq_14852397/article/details/46385713) 需要将项目编译之后在WEB-INF下面打开命令行窗口，执行下面的命令

    java -Djava.ext.dirs=lib org.apache.axis.client.AdminClient deploy.wsdd


如果执行上面的命令出现404的错误，就使用下面的命令，这里的端口号根据tomcat启动的端口号行，如果有上下文路径要加入上下文路径

    java -Djava.ext.dirs=lib org.apache.axis.client.AdminClient -lhttp://localhost:8080/services/AdminService deploy.wsdd


执行了之后会在目录中生成一个叫`server-config.wsdd`的文件，将这个文件复制项目中的`WEB-INF`下面就可以了。 deploy.wsdd需要自己编写放在`WEB-INF`下内容如下

    <deployment name="test" xmlns="http://xml.apache.org/axis/wsdd/"
                xmlns:java="http://xml.apache.org/axis/wsdd/providers/java">
        <!-- provider=”java:RPC”是服务类型，除了RPC方式以外，还有Document、Wrapped和Message方式 -->
        <service name="MyService" provider="java:RPC">
            <!-- 告诉服务应该调用的类 -->
            <parameter name="className" value="cn.ws.webservice.InfoService" />
            <!-- * 代表所有的方法都暴露 -->
            <parameter name="allowedMethods" value="Method1 Method2" />
            <!-- 当前WebService的作用域，分别是：request、session、application -->
            <parameter name="scope" value="application" />
        </service>
    </deployment>


wsdd的一些详细配置可以参考:[最详细的WSDD配置文件注释](http://blog.csdn.net/u011063151/article/details/52590282) 使用idea可以快速的创建`Axis`的项目文件,其他IDE工具应该都有这个功能，这里没有试过。 发布好的webservice访问界面如下 ![axis发布成功](http://imageu.oss-cn-shenzhen.aliyuncs.com/noteimage/axis%E5%8F%91%E5%B8%83%E6%88%90%E5%8A%9F.png) 这里的MyService就是自己部署Service下面对应发布的方法，其他的几个Service是axis自带的一些Service。 注：发布的时候如果出现异常`ClassNotFound`+发布类的权限定名，是因为要发布的类没有编译导致的，重新编译项目即可。

### 使用CXF发布(和Spring集成)

导入pom.xml

            <dependency>
                <groupId>org.apache.cxf</groupId>
                <artifactId>cxf-rt-frontend-jaxws</artifactId>
                <version>3.1.8</version>
            </dependency>
            <dependency>
                <groupId>org.apache.cxf</groupId>
                <artifactId>cxf-rt-transports-http</artifactId>
                <version>3.1.8</version>
            </dependency>


注：Spring版本使用4.2.0以上的，CXF的版本必须在3.0以上，要不然会报下面这个错

    java.lang.NoSuchMethodError:org.springframework.aop.support.AopUtils.isCglibProxyClass(Ljava/lang/Class;)


在web.xml中加入Servlet

        <servlet>
            <servlet-name>cxf</servlet-name>
            <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
            <load-on-startup>2</load-on-startup>
        </servlet>
        <servlet-mapping>
            <servlet-name>cxf</servlet-name>
            <url-pattern>/services/*</url-pattern>
        </servlet-mapping>


在Spring的配置文件中加入下面的内容

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:jaxws="http://cxf.apache.org/jaxws"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://cxf.apache.org/jaxws
            http://cxf.apache.org/schemas/jaxws.xsd">
    
        <!--  Cxf WebService 服务端示例,这里的implementor是Service的实现类，address是发布的地址的上下文路径 -->
        <jaxws:endpoint id="DemoWebservice" implementor="cn.cases.webservice.beans.HelloCxf" address="/demo"/>
    
    </beans>


注：使用的时候要加入`xmlns:jaxws="http://cxf.apache.org/jaxws"`等内容

    import java.io.Serializable;
    
    @XmlRootElement
    @XmlType(name = "User", namespace = Constants.NSPACE)
    public class UserDTO implements Serializable {
        private Long id;
        private String loginName;
        private String name;
        private String email;
    
        //略去getter/setter
    }


暴露在外的接口

    import javax.jws.WebParam;
    import javax.jws.WebService;
    
    @WebService(targetNamespace = Constants.NSPACE)
    @SOAPBinding(use = SOAPBinding.Use.ENCODED,style = SOAPBinding.Style.RPC,parameterStyle = SOAPBinding.ParameterStyle.WRAPPED)
    public interface HelloCxfInterface {
        //@WebService是必须的；@WebParam不是必须的。
        //如果没有@WebParam的描述，在wsdl文件内描述的方法中，参数名将变为arg0,arg1…以此类推.
        @WebMethod(action = "www.baidu.com",operationName = "say")
        public String sayHello(@WebParam(name = "name") String people);
        @WebMethod(action = "www.baidu.com",operationName = "getUser")
        public UserDTO getUser(String userId);
    }


这里的action对应的是生成的wsdl中的soapAction，soapAction是用来定义请求的地址的，可以为空，但是，SOAPBinding对应的是发送的SOAP消息中包含的内容，有几种模式，可以参考：[WSDL样式详解](http://blog.csdn.net/etttttss/article/details/17412081) [SOAP的消息绑定机制](http://blog.csdn.net/seasky323/article/details/6746174) 接口实现类

    @WebService(endpointInterface = "cn.cases.webservice.beans.HelloCxfInterface",//<wsdl:portType name="HelloCxfInterface">
            serviceName="HelloService",//<wsdl:portType name="HelloService">
            targetNamespace = Constants.NSPACE,
            name = "Hello")//<wsdl:port binding="tns:HelloServiceSoapBinding" name="HelloPort">
    public class HelloCxf implements HelloCxfInterface{
    
        @Override
        public String sayHello(String people){
            return "Hello "+people;
        }
    
        @Override
        public UserDTO getUser(String userId) {
            UserDTO dto = new UserDTO();
            dto.setId(Long.parseLong("1001"));
            dto.setLoginName("safdas");
            dto.setName("张三");
            dto.setEmail("sdasf@qq.com");
            return dto;
        }
    
        public String getOne(){
            return "One";
        }
    }


这里的接口和实现类都要加`@WebService`的标签，暴露在外的方法是以接口的方法来定的。如这里暴露在外的方法只有`sayHello()`和`getUser()`而没有`getOne()`

* * *

客户端
---

    import org.apache.axis.client.Call;
    import org.apache.axis.client.Service;
    import javax.xml.namespace.QName;
    import java.net.MalformedURLException;
    -------------------------------------------------------------------------
          Call call = (Call) service.createCall();
          //设置地址
          call.setTargetEndpointAddress(new java.net.URL("http://localhost:8080/services/hello?wsdl"));
          //设置要执行的方法
          call.setOperationName(new QName("http://case.cn","getUser"));
          //设置要传入参数,如果没有要传入的参数，则不要写这个
          call.addParameter("arg0", org.apache.axis.Constants.XSD_STRING,javax.xml.rpc.ParameterMode.IN);
          //设置返回的类型
          call.setReturnType(new javax.xml.namespace.QName("http://case.cn", "User"));
          call.setReturnClass(User.class);
          String name = "获得对象";
          //执行，调用webservice
          User result = (User) call.invoke(new Object[]{name});
          System.out.println("返回结果"+result.getEmail());


请求的SOAP消息

    <?xml version="1.0" encoding="utf-8"?>
    
    <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <soapenv:Body>
        <ns1:getUser xmlns:ns1="http://case.cn" soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
          <arg0 xsi:type="xsd:string">获得对象</arg0>
        </ns1:getUser>
      </soapenv:Body>
    </soapenv:Envelope>


响应的SOAP消息

    <?xml version="1.0" encoding="utf-8"?>
    
    <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
      <soap:Body>
        <ns2:getUserResponse xmlns:ns2="http://case.cn">
          <return>
            <email>sdasf@qq.com</email>
            <id>1001</id>
            <loginName>safdas</loginName>
            <name>张三</name>
          </return>
        </ns2:getUserResponse>
      </soap:Body>
    </soap:Envelope>


注：这个客户端是axis的客户端，类都是axis包中的

* * *

Idea生成webservice（以axis1.x为例）
----------------------------

IDE工具(如eclipse)应该都有这个功能，这里以idea为例

* * *

### 服务端

#### 新建的工程

![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/servernew1.png) 这里会下载一些jar包，需要有网络才可以 生成项目如下： ![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/servernew2.png) 这里的server-config.wsdd就是axis1.x的配置文件，web.xml中加入了一些axis的servlet，如果要发布自己的webservice，需要在wsdd中加入标签，如： ![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/servernew3.png) 配置一下tomcat就可以发布了访问地址为 项目访问地址+/services/+webservice名称?wsdl 可以用 项目访问地址+/services 查看axis发布的所有service ####已有的工程 ![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/serverlocation1.png) ![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/serverlocation2.png) idea会自动在项目的WEB-INF下生成server-confg.wsdd，发布的服务也是配置service web.xml也会自动加入axix的内容

* * *

### 客户端

#### 新建的工程

![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/clientnew1.png) ![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/clientnew2.png) 生成的main方法会有错误，这里需要手都更改一下第一行不用改导入包就可以了,locator.get+调用的类名 ![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/clientnew3.png) ![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/clientnew4.png) ![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/clientnew5.png) 就可以通过service调用对应的方法了。 这里还需要导入junit的包

#### 已有的工程

![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/clientlocation1.png) ![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/clientlocation2.png) 点击OK，这个时候在一个包上点击右键New的时候就会出现webclient选项 ![](http://imageu.oss-cn-shenzhen.aliyuncs.com/webservice-idea/clientlocation3.png) 点击按照上面介绍的操作即可 注：使用axis1.x生成的客户端也可以调用cxf生成的webservice