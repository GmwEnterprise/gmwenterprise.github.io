---
title: docker常用命令
date: 2019-12-13 11:28:07
tags:
  - docker
---

### 常用用法

Docker 常用命令
启动docker
sudo service docker start

停止docker
sudo service docker stop

重启docker
sudo service docker restart

列出Docker CLI命令
docker
docker container --help

显示Docker版本和信息
docker --version
docker version
docker info

Execute Docker image
docker run hello-world

列出镜像列表
docker image ls

列出docker容器 (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq

作者：凌雲木
链接：[https://www.jianshu.com/p/cef32b054968](https://www.jianshu.com/p/cef32b054968)
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 容器

- `docker image ls` 查看所有已安装镜像
- `docker container run [IMAGE]` 运行镜像
- `docker container ls` 查看所有运行中容器信息
- `docker container ls -all` 查看所有容器信息，包括已经停止的
- `docker container stop [CONTAINER_ID]` 停止容器，会执行清理工作
- `docker container kill [CONTAINER_ID]` 杀死容器，强行停止
- `docker container rm [CONTAINER_ID]` 删除容器
- `docker container start [CONTAINER_ID]` 启动现有容器
- `docker container logs [CONTAINER_ID]` 查看容器里面Shell的输出
- `docker container exec -it [CONTAINER_ID]` 进入一个正在运行的容器
- `docker container cp [containID]:[/path/to/file] .` 将容器中指定目录/文件拷贝到当前目录
- `docker ps` 查看容器运行情况

### 镜像

- `docker run [IMAGE]` 运行镜像
- `docker search [IMAGE]` 在docker hub中搜索镜像
- `docker pull [IMAGE]` 从docker hub中下载指定镜像
