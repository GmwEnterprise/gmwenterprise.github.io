---
title: 如何使用docker部署jar应用以及前端应用
date: 2019-12-18 21:00:22
tags:
  - docker
---

## 前言

主要是为了学习一下docker，提前熟悉容器化相关概念。

## 主要步骤

### 安装docker

安装的方式主要参照官方文档。由于我的服务器用的是ubuntu18.04LTS，所以我简单整理一下ubuntu的安装方式：
``` shell
# 卸载旧版本（安装前执行以下，防止安装出什么差错）
sudo apt-get remove docker docker-engine docker.io containerd runc
# 更新包索引
sudo apt-get update
# 安装以下包，以允许通过HTTPS来使用apt存储库
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
# 添加docker官方的GPG密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 使用以下命令来建立稳​​定的存储库，以便docker的安装
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
# 再次更新存储库索引
sudo apt-get update
# 安装docker
sudo apt-get install docker-ce docker-ce-cli containerd.io
# 查看安装好的docker版本
docker version
# 运行一个hello world镜像
sudo docker run hello-world
```
如此便安装好了最新版的docker。以上安装过程全部出自[官方文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/)。

### 使用过程简介

使用的过程很简单。

docker的镜像是只读的，可以从一个镜像中创建出一个容器，然后运行这个容器。容器有着满足程序基本需求的最小系统，占用资源也极低（相对于虚拟机）。镜像可以从官方镜像仓库中下载，也可以自己制作然后发布到官方仓库中让别人使用。docker官方提供了各种各样的官方镜像（offical images），比如openjdk、nginx、centos、ubuntu、mysql、mongodb等等。为了方便我将使用openjdk、mysql，以及nginx来部署我的应用。

### 部署

spring-boot打包后的程序就是一个jar文件，可以轻松地通过命令行来执行：`java -jar app.jar`。

为此，只需要一个可以执行java的环境即可。[openjdk](https://hub.docker.com/_/openjdk)官网有着详细的使用说明，这里我简单总结一下我的使用方式：
1. 创建一个Dockerfile文件用于基于openjdk镜像来生成我的应用镜像：
``` docker
FROM openjdk:11
ADD ./application.jar /usr
CMD ["java", "-jar", "/usr/application.jar"]
# 非常简单，几乎没有额外的设置，就创建好了可以执行的镜像
```
2. 拉取官方mysql镜像，设置好初始化必要设置后直接启动，作为应用的数据库

### 未完待续【2019年12月18日21点45分】