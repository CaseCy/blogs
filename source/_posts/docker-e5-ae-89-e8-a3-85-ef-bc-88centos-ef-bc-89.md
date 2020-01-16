---
title: " docker安装（centos）\t\t"
tags:
  - docker
url: 314.html
id: 314
comments: false
categories:
  - docker
date: 2018-12-26 14:19:04
---

参考：[docker安装](http://itmuch.com/docker/02-docker-install/)

系统要求
----

CentOS 7或更高版本（内核3.1以上） centos-extras 仓库必须处于启用状态，该仓库默认启用，但如果您禁用了该仓库，请按照https://wiki.centos.org/AdditionalResources/Repositories 中的描述重新启用。 建议使用overlay2 存储驱动

yum安装
-----

### 卸载老版本

在CentOS中，老版本Docker名称是docker 或docker-engine ，而Docker CE的软件包名称是docker-ce 。因此，如已安装过老版本的Docker，需使用如下命令卸载。

    sudo yum remove docker \
                      docker-common \
                      docker-selinux \
                      docker-engine
    

需要注意的是，执行该命令只会卸载Docker本身，而不会删除Docker存储的文件，例如镜像、容器、卷以及网络文件等。这些文件保存在/var/lib/docker 目录中，需要手动删除。

### 安装仓库

执行以下命令，安装Docker所需的包。其中，yum-utils 提供了yum-config-manager 工具；device-mapper-persistent-data 及 lvm2 则是devicemapper 存储驱动所需的包。

    //在此之前最好yum clean all,yum update
    sudo yum install -y yum-utils device-mapper-persistent-data lvm2
    

执行如下命令，安装stable 仓库。必须安装stable 仓库，即使你想安装edge 或test 仓库中的Docker构建版本。

    sudo yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
    

### 安装Docker ce

执行以下命令，更新yum的包索引

    sudo yum makecache fast
    

安装最新版的docker-ce

    sudo yum install docker-ce
    

指定安装版本

    yum list docker-ce.x86_64  --showduplicates | sort -r
    sudo yum install docker-ce-<VERSION>
    

启动docker

    sudo systemctl start docker
    

ps:如果启动不起来可尝试更换docker版本，自测用最新版本可能会出现问题，可尝试降一个版本 验证安装是否正确

    sudo docker run hello-world
    

### 参考文档

CentOS 7安装Docker官方文档：[https://docs.docker.com/engine/installation/linux/docker-ce/centos/](https://docs.docker.com/engine/installation/linux/docker-ce/centos/) ，文档中还讲解了在CentOS 7中安装Docker CE的其他方式

### 升级Docker CE

如需升级Docker CE，只需执行如下命令：

    sudo yum makecache fast
    

按照安装Docker的步骤，即可升级Docker

### shell一键安装

curl -fsSL get.docker.com -o get-docker.sh sudo sh get-docker.sh