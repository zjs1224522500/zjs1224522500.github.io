---
title: 'Docker常用命令'
date: 2017-10-24 20:32:20
tags: 服务器
---
## 1、安装与启动、更新
- 安装`Docker` : `apt-get install -y docker.io`
- 启动`Docker` : `systemctl start docker`
- 运行系统引导时启用 `docker`: `systemctl enable docker`
- 核对`Docker`版本：`docker version`
- 更新`Docker`：`docker-machine upgrade default`

<!-- more -->

## 2、镜像、容器相关
### 镜像 images
- 查看安装的镜像**image**：`docker images`

- 搜索可用的**Docker**镜像：`docker search 镜像名字`

- 运行镜像：`docker run 镜像名称 要执行的命令`
    - 指定参数 `-d` 让容器在后台运行
    - `-P` 将容器内部使用的网络端口映射到我们的主机上  
    - `-p` 绑定指定端口 `-p 5000:5000`，`-p 127.0.0.1:5001:5002`
    - `-t`:在新容器内指定一个伪终端或终端。
    - `-i`:允许你对容器内的标准输入 (STDIN) 进行交互。
    - `--name`：为容器命名

> `> docker run -p 8080:80 --name nginx_web -it hub.c.163.com/library/nginx` 

- 从**dockerhub**上**pull**镜像：`docker pull 镜像名称`

- 构建镜像：`docker build 镜像名`
> `docker build -t runoob/centos:6.7 .`  
    - `-t` ：指定要创建的目标镜像名  
    - `.` ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径

- 设置镜像标签：`docker tag 镜像ID 镜像名：标签`

- 删除镜像：`docker rmi 镜像名称`



### 容器 containers
- 查看运行的容器**container**：`docker ps`
    - `-l`:查询最后一次创建的容器
    - `-a`:查看所有容器

- 查看容器的详细信息（JSON格式）：`docker inspect 容器编号`

- 提交容器修改：`docker commit id 容器`
> `docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2`  
> **-m** 提交的描述信息  
> **-a** 指定镜像作者

- 停止容器：`docker stop 容器名 或 容器ID`

- 查看指定容器的某个确定端口映射到宿主机的端口号：`docker port 容器ID或容器名`

- 查看指定容器的标准输出：`docker logs 容器ID或容器名`

- 查看容器内部运行的进程：`docker top 容器ID或容器名`

- 删除指定的容器：`docker rm 容器名或容器编号`

- 进入后台容器并提供`bash`: `docker exec -it 容器名 或 容器编号 bash`
    - `-it` 同`run`命令中的`-it`一致

- 退出容器: `Ctrl + D` 或 `exit`

