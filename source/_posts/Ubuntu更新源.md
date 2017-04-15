---
title: Ubuntu更新源
date: 2016-11-22 22:30:08
categories: [Linux, Ubuntu]
tags: [Linux, Ubuntu, 更新源]
---

### 1.备份原来的源 ###

```
cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

### 2.编辑源文件 ###

```
vi /etc/apt/sources.list
```

### 复制以下代码进文件/etc/apt/sources.list ###

```
#163
deb http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse
#中科大
deb http://debian.ustc.edu.cn/ubuntu/ trusty main multiverse restricted universe
deb http://debian.ustc.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe
deb http://debian.ustc.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe
deb http://debian.ustc.edu.cn/ubuntu/ trusty-security main multiverse restricted universe
deb http://debian.ustc.edu.cn/ubuntu/ trusty-updates main multiverse restricted universe
deb-src http://debian.ustc.edu.cn/ubuntu/ trusty main multiverse restricted universe
deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe
deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe
deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-security main multiverse restricted universe
deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-updates main multiverse restricted universe
#sohu
deb http://mirrors.sohu.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.sohu.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.sohu.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.sohu.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.sohu.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.sohu.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.sohu.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.sohu.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.sohu.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.sohu.com/ubuntu/ trusty-backports main restricted universe multiverse
#oschina
deb http://mirrors.oschina.net/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.oschina.net/ubuntu/ trusty-backports main restricted universe multiverse
deb http://mirrors.oschina.net/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.oschina.net/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.oschina.net/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.oschina.net/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.oschina.net/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.oschina.net/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.oschina.net/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.oschina.net/ubuntu/ trusty-updates main restricted universe multiverse
#北京交通大学
deb http://mirror.bjtu.edu.cn/ubuntu/ trusty main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ trusty-security main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ trusty-updates main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ trusty main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ trusty-security main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ trusty-updates main multiverse
# 北京理工
deb http://mirror.bjtu.edu.cn/ubuntu/ precise main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ precise-backports main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ precise-proposed main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ precise-security main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ precise-updates main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ precise main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ precise-backports main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ precise-proposed main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ precise-security main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ precise-updates main multiverse restricted universe

#上海交大
deb http://ftp.sjtu.edu.cn/ubuntu/ trusty universe restricted multiverse main
deb http://ftp.sjtu.edu.cn/ubuntu/ trusty-security universe restricted multiverse main
deb http://ftp.sjtu.edu.cn/ubuntu/ trusty-updates universe restricted multiverse main
deb http://ftp.sjtu.edu.cn/ubuntu/ trusty-proposed universe restricted multiverse main
deb http://ftp.sjtu.edu.cn/ubuntu/ trusty-backports universe restricted multiverse main

#阿里云服务器
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
```

### 3.运行更新命令 ###

```
sudo apt-get update
```