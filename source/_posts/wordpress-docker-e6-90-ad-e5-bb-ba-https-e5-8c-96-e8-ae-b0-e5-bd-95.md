---
title: " wordpress docker搭建 https化记录\t\t"
tags:
  - docker
url: 404.html
id: 404
comments: false
categories:
  - docker
  - 杂
date: 2019-04-08 09:38:52
---

参考： [Centos7.4下用Docker-Compose部署WordPress](https://www.cnblogs.com/wushangjue/p/7795969.html)

使用docker-compose创建wordpress
---------------------------

安装略过 编写docker-compose.yml文件 写入如下内容：

    version: '3'
    
    services:
       db:
         image: mysql:5.7
         volumes:
           - db_data:/var/lib/mysql
         restart: always
         environment:
           MYSQL_ROOT_PASSWORD: "your password"
           MYSQL_DATABASE: wordpress
           MYSQL_USER: wordpress
           MYSQL_PASSWORD: wordpress
    
       wordpress:
         depends_on:
           - db
         image: wordpress:latest
         volumes:
            - wp_site:/var/www/html
         ports:
           - "9000:80"
         restart: always
         environment:
           WORDPRESS_DB_HOST: db:3306
           WORDPRESS_DB_USER: wordpress
           WORDPRESS_DB_PASSWORD: wordpress
    
    volumes:
        db_data:
        wp_site:
    
    networks:
      default:
        external:
          name: nginx-network
    

docker创建一个网络

    docker network create nginx-network
    

然后使用docker-compose构建

    docker-compose up -d
    

配置Nginx
-------

按照上面的内容 wordpress可以通过端口9000访问，但是现在先不忙访问 在对应的域名服务商上配置好域名映射，使域名能够正确访问到服务器的80端口（Nginx） 配置好了之后修改Nginx配置，将请求代理到9000端口 注：这时候还没有配置https

    upstream ssl{
            server 127.0.0.1:9000 weight=1;
    }
    
    server {
        listen 80;
        location / {
            proxy_pass http://ssl;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_set_header X-NginX-Proxy true;
        }
    }
    

重启nginx，就可以访问到网站了 https从现在开始配置 首先配置好wordpress，然后到wordpress中下载一个插件**Really Simple SSL** 这时候点击启用，会提示没有证书，下一步就是将https配置到Nginx上 这里略过 配置好了之后，通过https访问刚才的页面的时候，就会有启用https，点击启用，就可以使用https访问了

附录
--

### docker的一些命令

查看创建的数据卷

    docker volume ls
    

删除所有没有用到的数据卷

    docker volume prune
    

查看数据卷对应主机的位置

    docker volume inspect --format '{ { .Mountpoint }}' 数据卷名称
    

停掉docker-compose，和up一样需要在docker-compose.yml目录执行

    docker-copose down
    

列出现在的网络

    docker network ls
    

查看容器的日志

    docker logs 容器id/名称
    

### docker版本

查看docker版本

    docker version
    

本文使用的docker 版本为

    Client:
     Version:           18.09.3
     API version:       1.39
     Go version:        go1.10.8
     Git commit:        774a1f4
     Built:             Thu Feb 28 06:33:21 2019
     OS/Arch:           linux/amd64
     Experimental:      false
    
    Server: Docker Engine - Community
     Engine:
      Version:          18.09.3
      API version:      1.39 (minimum version 1.12)
      Go version:       go1.10.8
      Git commit:       774a1f4
      Built:            Thu Feb 28 06:02:24 2019
      OS/Arch:          linux/amd64
      Experimental:     false
    

### Nginx配置http端口跳转到https端口

方式一

    server {
            listen      80;
            server_name 域名;
            rewrite /(.*) https://$host/$1 permanent;
    }
    

方式二 在方式一基础上将rewrite修改为

    return      301 https://$server_name$request_uri;
    

还有些其他方式，自行百度