---
title: docker-dev
date: 2017-03-17 00:15:01
tags: [docker 开发环境]
categories: 工具 
---

# 基于Docker的Windows/Mac开发环境搭建

[toc]

## 目的

本文目的:

- 介绍基本Docker使用方法
- 基于当前最火热容器技术(docker)简化日常工作
- 在Windows/Mac搭建可移植的开发环境，摒弃掉使用计算云虚拟机为开发环境
- 相关项目可做成Docker镜像，方便部署，同时方便新人快速构建开发环境
- 在Windows/Mac使用超低的资源(~50M)，运行完整的Linux环境

<!--more-->

## 准备工作

操作系统：Windows/Mac OS

准备软件:

- Xshell: http://sys.qiyi.domain/download/devel-tools/Xshell4.exe
- DockerToolsBox: http://sys.qiyi.domain/download/devel-tools/DockerToolbox.exe


## 开发环境搭建

### 安装Docker客户端

Windows: https://docs.docker.com/installation/windows

Mac OS X: https://docs.docker.com/installation/mac


根据自己的系统安装完Docker Tools，执行 ``Docker Quickstart Terminal`` 会自动在Virtual Box创建基于内存的boot2docker虚拟机，


### 使用Xshell

官方文档介绍使用Windows Shell和putty链接boot2docker虚拟机或者Docker实例，本部分介绍使用Xshell链接，因为Xshell相对更方便：

- 更方便管理ssh-key
- 更多快捷键

使用Virtual Box查看boot2docker虚拟机的ip地址，使用 ``%USERPROFILE%\.docker\machine\machines\<name_of_your_machine>\id_rsa`` 私钥在xshell用docker用户登陆，IP地址是192.168.99.100 。

进入虚拟机添加qiyi内部镜像源为可信任源：


修改 ``/var/lib/boot2docker/profile``  添加下面信息


`EXTRA_ARGS='--insecure-registry docker-registry.qiyi.virtual'`

然后可执行docker命令:

```
    $ docker ps
    $ docker login 
    $ docker search 
    $ vi Dockerfile && docker build -t <your image name> .
    # docker run ...

```

若登录失败，重启docker2boot，重试即可

### 创建开发环境Dockerfile

### 拉取现有镜像


`docker pull /https://hub.docker.com/[your account]/centos-devenv:3.2`

### 创建容器


```
$ docker run --name dev -i -t -v /c/Users/Administrator/workspace:/root/workspace -p 222:22 -p 8080:80 https://hub.docker.com/[your account]/centos-devenv:3.2  /bin/bash

```



此命令可创建并进入容器，显示shell终端，此时可在Windows系统copy key 到 /c/Users/Administrator/workspace，然后执行

```
$ cp /root/workspace/id_rsa* /root/.ssh
$ cat /root/.ssh/id_rsa.pub >>/root/.ssh/authorized_keys
$ chmod 600 /root/.ssh/id_rsa

```

此时可以通过Xshell等客户端登陆Docker 容器了。

左键左键通过如下命令直接进入容器

`$ docker exec -i -t <container-name> /bin/bash`

note:

`docker 开发容器是在Virtual Box虚拟机启动的，上面的命令将22端口映射到外面为222所以登陆时使用222端口.`


创建启动脚本(/c/Users/Administrator/workspace/go.sh):

```
#!/bin/sh

container_id=dev
docker start $container_id
docker exec $container_id supervisord >/dev/null 2>&1 &

```

每次启动VM时，链接boot2docker虚拟机，即可执行下面命令启动之前的容器:

` sh /c/Users/Administrator/workspace/go.sh`

## centos-devenv 开发环境说明

目前组内 centos-devenv 环境是3.0版本有如下特性：

[3.2] Updates

- C/C++ 开发环境
- clang-format based 自动代码格式化配置
- 增加vim-snippets：各类语言的自动补全代码片段

完善:

- 使用tab完成代码自动补全
- 修复之前编辑模式下左键不可用的问题

[2.1] Updates

- Add vim-go Plugin

[2.0] Updates

- 完整的 python、go、java开发环境
- vim插件: python go java 自动补全功能
- python: 日常常用的开发库已经安装，可用pip freeze 查看
- man 手册，相比1.0版本 增加了man 手册
- 用户需要用自己的账号配置~/.ssh/config ~/.gitconfig ，即可快速搭建完成开发环境
- 内置nginx、supervisord
- nginx /sshd 服务使用supervisord默认启动可使用ssh客户端链接终端开发


## Tips

### 时区问题

修正时区

```
$ cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

```



清理

```

$ docker ps --all|grep Exited|awk '{print $1}'|xargs  docker rm
$ docker images | grep "^<none>" | awk '{print $3}'|xargs docker rmi

```


## 参考资料

### 官方资料

Docker docs: https://docs.docker.com/

Dockerfile Reference: https://docs.docker.com/engine/reference/builder/

Best practises for writing Docker file: https://docs.docker.com/engine/articles/dockerfile_best-practices/

Using Supervisor with Docker: https://docs.docker.com/engine/articles/using_supervisord/

