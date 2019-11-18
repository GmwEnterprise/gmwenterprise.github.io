---
title: 从阿里云上安装ubuntu server作为一个运行javaweb程序的服务器
date: 2019-11-14 10:25:27
tags: java
---

写这篇文章的目的在于详细记录一下初始化一台服务器的一些基本操作，本来是可以用的，结果最近笔者的服务器遭到了黑客攻击，丢失了好几条**宝贵**的数据不说，还被威胁说不给0.03个btc就删了我的备份。。。

因为我图方便，希望能够在家里或者公司都能自由连接上我的服务器以便开发我的个人小博客网站，所以开放了端口。。。

**血的教训！**

以下是正文

---

使用的镜像为ubuntu18.04，惊喜的是系统自带SSH，不需要像老版本一样需要安装额外的ssh包才能使用xshell进行远程登录。

#### 安装mysql

我先前使用的数据库是mysql8.0，所以此时我要给服务器安装mysql8.0，但是apt search能找到的包只有mysql5.7，所以得自己另寻办法：

1. 从这个[链接](https://dev.mysql.com/downloads/repo/apt/)去下载一个mysql-apt-config-xxxxxx.deb包，然后通过xftp上传到服务器。我将它上传到了根目录；
2. 安装：`dpkg -i /mysql-apt-config_0.8.14-1_all.deb`，会弹出粉色背景的安装界面，根据需求选择后选择OK；
3. `apt update`更新下载列表，然后`apt upgrade`更新。至此，已可以通过`apt install mysql-server`来安装mysql8；
4. `apt install mysql-server`即可安装，选择密码加密方式时选择了mysql5.7的mysql_native_password加密方式，因为目前默认的客户端管理工具都不支持新版加密方式；
5. 安装好后，修改mysql.user表root用户的host字段为'%'，然后重启mysql服务`service mysql restart`便开启了mysql8的远程登录。
6. 为了安全性的考虑，只好设置Mysql数据库的3306端口IP限制，每次换个地方，换了网络，就要把自己的外网IP给替换为当前的限制IP，这样也算是比较有效降低了被黑客入侵的风险。。。

#### 安装jdk

#### 安装redis

#### 尝试使用docker

未完待续...
