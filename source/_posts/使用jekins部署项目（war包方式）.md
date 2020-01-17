---
title: " 使用jekins部署项目（war包方式）\t\t"
tags:
  - 杂
url: 95.html
id: 95
comments: false
categories:
  - 未分类
date: 2018-10-19 08:49:04
---
[Jenkins官网](https://jenkins.io/)

---
## 启动Jekins
到Jenkins官网下载war包，可以直接通过jar包启动，默认端口是8080，war包中自带jetty，可以通过下面方式修改启动的端口

    java -jar jenkins.war --httpPort=8020
也可以将war包放到tomcat下启动
## 登录Jekins
第一次启动的时，Jenkins会自动生成一个密码，在日志文件中，这个密码作为默认账号admin的默认密码，参考
[Jekins官方文档中war包部分](https://jenkins.io/doc/book/installing/)，如果没有看到，可以在当前用户文件夹下的`.jenkins\secrets`下的`initialAdminPassword`这个文件中查找到（注windows和linux一样，linux中`.jenkins`可能是隐藏的）
## 使用Jenkins构建项目(maven)
### 构建准备
因为是maven项目，所以要在安装jekin的系统中安装git(如果是git)，maven，jdk并把他们的位置配置到

    系统管理---全局工具配置(Globel tools)--将对于工具的路径配置进去，自动配置暂时不知道什么用，点了没效果
![全局工具配置](http://imageu.oss-cn-shenzhen.aliyuncs.com/jenkins/%E5%85%A8%E5%B1%80%E5%B7%A5%E5%85%B7%E9%85%8D%E7%BD%AE.png)
工具配置好了之后还要准备一些插件
Maven Integration plugin 用于构建maven项目
Git plugin git的插件
通过

    系统管理---插件管理---可选插件
搜索安装，已安装忽略此步
### 构建配置
以上步骤准备好之后

    新建---输入项目名---构建一个maven项目
#### 源码管理
这里以git项目构建，svn类似
源码管理部分选择git,填入git的地址，填入之后，会自动测试，如果是报类似无法使用xxx命令，一般是git没装好（注：无法使用https也是，这种把git删了，重新完整的安装之后就可以了），如果提示没有权限
点击下面Credentials旁的add添加身份认证信息，如果有可以直接选，都正确情况下就不会存在红字。
#### build
Goals and options 填入maven构建的命令即可，如clean package
#### Post Steps
这里可以选择构建成功后执行的操作，上面有几个选项，表示执行的时间

    Run only if build succeeds 构建成功时执行
    Run only if build succeeds or is unstable 构建成功或者不稳定时执行
    Run regardless of build result 不管构建结果如何都执行
add选项中第一个选项是用在windows系统中执行命令的，第二个是用在linux中执行shell命令的，可以将执行的命令单独写在一个可执行文件中，如windos的bat，linux的sh文件（执行前记得先给文件授权(`chmod 777 文件`)），在这里填写执行命令
![s](http://imageu.oss-cn-shenzhen.aliyuncs.com/jenkins/postSteps.png)
#### 开始构建
配置好之后点击保存,可以在my views中查看新建的项目，点击立即构建，就可以开始构建项目了，点击左边
`构建执行状态`下的进度条，可以查看输出的控制台信息

## 设置构建邮件通知
用于构建过程中的邮件通知（成功失败都会通知）
### 设置发送邮箱信息
操作之前，要安装一些邮件的插件（一般在安装Jenkins的时候都会安装，如果勾选的话），具体百度

    系统管理---系统设置---邮件通知
填写stmp邮件服务器，如163邮箱的`smtp.163.com`每个邮件服务器的stmp不同，这个可以在网上查找，另外要实现用stmp服务，要先在对应的邮箱中开启stmp服务并设置密码，填写好之后点击高级，填写开启stmp服务的账户名和密码，最好勾选ssl协议，有些邮箱不用ssl协议不会通过，端口和用户默认后缀可以不填。
填写好之后，可以勾选`通过发送测试邮件测试配置`，填写发送的邮箱来测试填写的内容是否正确。
### 项目中设置要发送的邮箱
设置需要发送邮件的项目在`构建设置`中勾选`E-mail Notification`下面有几个错误的情况通知，根据情况自己选择，填入需要发送的邮箱就可以了
## 远程构建
### 构建前配置
可以参考：[使用shell脚本部署到远程服务器](http://blog.csdn.net/russ44/article/details/51694074)
在可选插件中安装`publish over ssh`插件，安装之后打开系统设置
在publish over ssh部分，展开高级选项配置ssh server

    Name 这里可以随便填写
    HostName 填写要连接的服务器ip或者名称
    UserName 要登录的用户
    Remote Directory 远程服务器文件目录
    勾选 Use password authentication, or use a different key
    Passphrase / Password 填写上方用户名的密码
其他部分可以不管，具体可以自行研究
### 项目设置
在项目设置的构建环境处勾选

    Send files or execute commands over SSH after the build runs
    //在构建之后发送文件或执行命令
点击add Transfers

    Source files 要拷贝的文件，这里是可以填相对路径(相对于项目的位置，如target/*.war)，也可以填绝对路径
    Remove prefix 移除前缀 可以不填
    Remote directory 远程目录，如果填相对的路径是相对于之前系统设置中的路径的
    Exec command 要执行的命令，高版本Jenkins会要求填，但是可以不管
配置好之后保存，重新构建下就可以了

## 遇到的问题
linux构建时tomcat启动随jenkin执行命令的结束而停止，导致tomcat启动不成功解决方法在执行的shell命令前加入

    export BUILD_ID=aaaa
    //BUILD_ID为任意值
构建最后报错
Maven JVM terminated unexpectedly with exit code 137
原因：http://blog.csdn.net/mzh1992/article/details/62432330