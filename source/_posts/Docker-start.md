---
title: 'Docker入门笔记'
date: 2017-10-24 20:32:20
tags: 服务器
---
## 1、什么是Docker？
> `Docker`是一个开源的引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署，包括`VMs（虚拟机）`、`bare metal`、`OpenStack` 集群和其他的基础应用平台。

<!-- more -->
## 2、Docker的组成
> `Docker`系统有两个程序：`docker`服务端和`docker`客户端。其中`docker`服务端是一个服务进程，管理着所有的容器。`docker`客户端则扮演着`docker`服务端的远程控制器，可以用来控制`docker`的服务端进程。大部分情况下，`docker`服务端和客户端运行在一台机器上。

## 3、Docker在容器中安装新的程序
- 在使用`docker run 镜像名称 要执行的命令`时，对于要执行的命令，有的可能存在需要交互的情况，即需要用户输入命令来进行确认，例如`apt-get`，但在`docker`环境中是无法响应这种交互的，故常常需要加上些参数，例如`apt-get -y`，来省略交互过程。

## 4、保存对容器的修改
> docker commit id 容器 
- 当你对某一个容器做了修改之后（通过在容器中运行某一个命令），可以把对容器的修改保存下来，这样下次可以从保存后的最新状态运行该容器。`docker`中保存状态的过程称之为`committing`，它保存的新旧状态之间的区别，从而产生一个新的版本。
- 对于`commit`过程中`id`，需要先使用`docker ps -l`命令获得安装完`ping`命令之后的容器`id`。`
- 无需拷贝完整的`id`，通常来讲最开始的三至四个字母即可区分。（非常类似`git`里面的版本号)
- 执行完后，会返回新版本镜像的id号。通过使用`dockr images`命令可查看最近的镜像版本
- 提交前需要停止容器

## 5、创建镜像
- 当我们从docker镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改。
    -  1.从已经创建的容器中更新镜像，并且提交这个镜像
    -  2.使用 `Dockerfile` 指令来创建一个新的镜像

## 6、构建镜像
- 从零开始来创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。
```
runoob@runoob:~$ cat Dockerfile 
FROM    centos:6.7
MAINTAINER      Fisher "fisher@sudops.com"

RUN     /bin/echo 'root:123456' |chpasswd
RUN     useradd runoob
RUN     /bin/echo 'runoob:123456' |chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
EXPOSE  22
EXPOSE  80
CMD     /usr/sbin/sshd -D
```
- 每一个指令都会在镜像上创建一个新的层，每一个指令的前缀都必须是大写的。
- 第一条`FROM`，指定使用哪个镜像源
- `RUN` 指令告诉`docker`在镜像内执行命令，安装了什么。然后，我们使用 `Dockerfile` 文件，通过 `docker build` 命令来构建一个镜像。