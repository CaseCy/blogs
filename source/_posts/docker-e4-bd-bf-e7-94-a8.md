---
title: " docker使用\t\t"
tags:
  - docker
url: 121.html
id: 121
comments: false
categories:
  - docker
date: 2018-10-19 09:45:53
---

tags： docker

* * *

跳过安装教程

* * *

命令 [参考](https://blog.csdn.net/permike/article/details/51879578) [run命令详解](https://blog.csdn.net/one_clouder/article/details/39224767)

    docker version // docker的版本信息，包含server和client 
    docker info //docker的基本信息
    docker images //查看镜像
    docker ps //查看运行中的容器 -a包括停止的容器
    docker start [容器名称或id] //启动容器
    docker stop [容器名称或id] //停止容器
    docker exec -it [容器名称或id] /bin/bash 进入容器并运行bash，后台容器exit后可以不退出
    docker rm [容器名称或id] //删除容器 ，删除镜像是rmi
    docker commit [命令] [容器名称或id] repo:tag(可以不写)  //将容器变为镜像
      -a 作者
      -m 提交的说明文字
      -p 提交时暂停容器
    docker inspect [容器或者镜像的名称，id] //查看容器或镜像的详细信息
    docker run [命令及命令信息] [容器名称或id] [启动后要执行的命令(如：/bin/bash)]
      -i 以交互模式运行容器
      -v 挂载数据卷 [主机地址]:[容器中地址] 主机地址可以不用
      -d 在后台运行容器
      -p 将主机端口和容器端口做映射 [主机端口]:[容器端口]