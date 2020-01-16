---
title: " DockerFile\t\t"
tags:
  - docker
url: 203.html
id: 203
comments: false
categories:
  - docker
date: 2018-10-22 17:07:52
---

参考及引用：[Dockerfile指令详解](http://itmuch.com/docker/07-docker-docker-file/)

FROM 指定基础镜像
-----------

使用FROM指令指定基础镜像，FROM指令有点像Java里面的“extends”关键字。需要注意的是，FROM指令必须指定且需要写在其他指令之前。FROM指令后的所有指令都依赖于该指令所指定的镜像。 支持三种格式： - `FROM <image>` \- `FROM <image>:<tag>` \- `FROM <image>@<digest>`

ADD 复制文件
--------

ADD指令用于复制文件，格式为： - `ADD <src>... <dest>` \- `ADD ["<src>",... "<dest>"]` 从src目录复制文件到容器的dest。其中src可以是Dockerfile所在目录的相对路径，也可以是一个URL，还可以是一个压缩包 注： 1. 如果src是一个url，以/会被视为一个目录，不以/结尾视为文件 2. 如果src是一个目录，那么整个目录下的内容将会被拷贝，包括文件系统元数据。 3. 如果文件是可识别的压缩包格式，则docker会自动解压。 4. src必须在构建的上下文内，不能使用例如：ADD ../somethine /something 这样的命令，因为docker build 命令首先会将上下文路径和其子目录发送到docker daemon。

    ADD abc.jar def.jar
    

COPY 复制文件
---------

复制文件，格式为： \- `COPY <src>... <dest>` \- `COPY ["<src>",... "<dest>"]` 复制本地端的src到容器的dest。COPY指令和ADD指令类似，COPY不支持URL和压缩包。

ARG 设置构建参数
----------

ARG指令用于设置构建参数，类似于ENV。和ARG不同的是，ARG设置的是构建时的环境变量，在容器运行时是不会存在这些变量的。 格式为： - `ARG <name>[=<default value>]`

ENV 设置环境变量
----------

ENV指令用于设置环境变量，格式为： - `ENV <key> <value>` \- `ENV <key>=<value> ...` 示例：

    ENV JAVA_HOME /path/to/java
    

### ENV 和 ARG区别

参考：[http://scmroad.com/topics/39188](http://scmroad.com/topics/39188) \- `The ARG instruction defines a variable that users can pass at build-time to the builder with the docker build command using the --build-arg = flag.` ARG指令定义了用户可以在编译时或者运行时传递的变量，如使用如下命令： `--build-arg <varname>=<value>` \- `The ENV instruction sets the environment variable to the value .` The environment variables set using ENV will persist when a container is run from the resulting image. ENV指令是在dockerfile里面设置环境变量，不能在编译时或运行时传递。 有效结合：

    ARG var
    ENV var=${var}
    

You can then either build an image with a specific var value at build-time (docker build --build-arg var=xxx), or run a container with a specific runtime value (docker run -e var=yyy)

ENTRYPOINT 入口点
--------------

格式为： \- `ENTRYPOINT ["executable", "param1", "param2"]` \- `ENTRYPOINT command param1 param2` ENTRYPOINT和CMD指令的目的一样，都是指定Docker容器启动时执行的命令，可多次设置，但只有最后一个有效。ENTRYPOINT不可被重写覆盖。

CMD 容器启动命令
----------

CMD指令用于为执行容器提供默认值。每个Dockerfile只有一个CMD命令，如果指定了多个CMD命令，那么只有最后一条会被执行，如果启动容器的时候指定了运行的命令，则会覆盖掉CMD指定的命令。 支持三种格式： - `CMD ["executable","param1","param2"] (推荐使用)` \- `CMD ["param1","param2"] (为ENTRYPOINT指令提供预设参数)` \- `CMD command param1 param2 (在shell中执行)` 示例：

    CMD echo "This is a test." | wc -
    

RUN 执行命令
--------

该指令支持两种格式： \- `RUN <command>` \- `RUN ["executable", "param1", "param2"]` `RUN <command>`在shell终端中运行，在Linux中默认是/bin/sh -c ，在Windows中是 cmd /s /c ，使用这种格式，就像直接在命令行中输入命令一样。 `RUN ["executable", "param1", "param2"]` 使用exec执行，这种方式类似于函数调用。指定其他终端可以通过该方式操作，例如：RUN \["/bin/bash", "-c", "echo hello"\] ，该方式必须使用双引号\[“\]而不能使用单引号\[‘\]，因为该方式会被转换成一个JSON 数组。

### CMD/ENTRYPOINT/RUN区别

参考:[https://segmentfault.com/q/1010000000417103](https://segmentfault.com/q/1010000000417103) 个人理解： ENTRYPOINT 相当于设置初始值 CMD 相当于覆盖初始值

> RUN是在Build时运行的，先于CMD和ENTRYPOINT。Build完成了，RUN也运行完成后，再运行CMD或者ENTRYPOINT

EXPOSE 声明暴露的端口
--------------

EXPOSE指令用于声明在运行时容器提供服务的端口，格式为： - `EXPOSE <port> [<port>...]` 需要注意的是，这只是一个声明，运行时并不会因为该声明就打开相应端口。该指令的作用主要是帮助镜像使用者理解该镜像服务的守护端口；其次是当运行时使用随机映射时，会自动映射EXPOSE的端口。示例：

    # 声明暴露一个端口示例
    EXPOSE port1
    # 相应的运行容器使用的命令
    docker run -p port1 image
    # 也可使用-P选项启动
    docker run -P image
    
    # 声明暴露多个端口示例
    EXPOSE port1 port2 port3
    # 相应的运行容器使用的命令
    docker run -p port1 -p port2 -p port3 image
    # 也可指定需要映射到宿主机器上的端口号  
    docker run -p host_port1:port1 -p host_port2:port2 -p host_port3:port3 image
    

USER 设置用户
---------

该指令用于设置启动镜像时的用户或者UID，写在该指令后的RUN、CMD以及ENTRYPOINT指令都将使用该用户执行命令。 格式为： - `USER 用户名`

VOLUME 指定挂载点
------------

该指令使容器中的一个目录具有持久化存储的功能，该目录可被容器本身使用，也可共享给其他容器。当容器中的应用有持久化数据的需求时可以在Dockerfile中使用该指令。格式为 - `VOLUME ["/data"]` 使用示例：

    FROM nginx
    VOLUME /tmp
    

当该Dockerfile被构建成镜像后，/tmp目录中的数据即使容器关闭也依然存在。如果另一个容器也有持久化的需求，并且想使用以上容器/tmp目录中的内容，则可使用如下命令启动容器：

    docker run -volume-from 容器ID 镜像名称  # 容器ID是di一个容器的ID，镜像是第二个容器所使用的镜像。
    

WORKDIR 指定工作目录
--------------

格式为： \- `WORKDIR /path/to/workdir` 切换目录指令，类似于cd命令，写在该指令后的RUN，CMD以及ENTRYPOINT指令都将该目录作为当前目录，并执行相应的命令。 Dockerfile还有一些其他的指令，例如STOPSINGAL、HEALTHCHECK、SHELL等。由于并不是很常用，本书不作赘述。有兴趣的读者可前往[https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/) 扩展阅读。

LABEL 为镜像添加元数据
--------------

LABEL指令用于为镜像添加元数据。 格式为： - `LABEL <key>=<value> <key>=<value> <key>=<value> ...` 使用 ""和"\\"转换命令行，示例：

    LABEL "com.example.vendor"="ACME Incorporated"
    LABEL com.example.label-with-value="foo"
    LABEL version="1.0"
    LABEL description="This text illustrates \
    that label-values can span multiple lines."
    

MAINTAINER 指定维护者的信息（已过时）
------------------------

MAINTAINER指令用于指定维护者的信息，用于为Dockerfile署名 格式为： - `MAINTAINER <name>` 该指令已过时，建议使用如下形式： `LABEL maintainer="SvenDowideit@home.org.au"`

拓展阅读
----

*   Dockerfile官方文档：[https://docs.docker.com/engine/reference/builder/#dockerfile-reference](https://docs.docker.com/engine/reference/builder/#dockerfile-reference)
*   Dockerfile最佳实践：[https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#build-cache](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#build-cache)
*   例子参考 [http://blog.51cto.com/zero01/2069879](http://blog.51cto.com/zero01/2069879) [https://blog.csdn.net/qq_34409701/article/details/79949401](https://blog.csdn.net/qq_34409701/article/details/79949401)