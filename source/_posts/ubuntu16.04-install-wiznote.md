---
title: 在 Ubuntu 16.04 中 安装为知笔记
date: 2017-06-21 22:00:00
categories: [Ubuntu]
tags: [Ubuntu, WizNote]
---

> 摘要：本文讲述如何在 Ubuntu 16.04 中编译安装为知笔记。


### 安装依赖的软件

#### git
```
sudo apt-get install git
```

#### 编译工具
```
sudo apt-get install build-essential
```

#### CMake
```
sudo apt-get install cmake
```

#### zlib
```
sudo apt-get install zlib1g-dev
```

#### Qt
安装 5.7.0 for Linux 64-bit (715 MB) 或者更高版本
[Qt5.7.0](http://download.qt.io/official_releases/qt/5.7/5.7.0/qt-opensource-linux-x64-5.7.0.run)
下载下来的安装文件，更改权限为可执行，然后执行安装程序。不要用管理员权限安装，直接安装到home目录即可，例如~/Software/Qt5.7.0

### 安装 Wiz

#### 下载 Wiz 源代码
```
cd ~
mkdir WizTeam
cd WizTeam
git clone https://github.com/WizTeam/WizQTClient.git
cd WizQTClient
git checkout v2.5.1
```

#### 在 Qt 打开 Wiz
在 **Qt5.7.0/Tools/QtCreator/bin/** 目录下运行 qtcreator 程序，并通过 Qt 打开 WizQTClient 目录下的 ** CMakeLists.txt ** 文件

右键点击项目 选择 ** Run CMake **
右键点击项目 选择 ** Run **

##### 问题
如果遇到下面的问题
```
Failed to find "GL/gl.h" in "/usr/include/libdrm".
```
处理方法：
执行下面命令
```
sudo apt-get install libgl1-mesa-dev libglu1-mesa-dev
```
一切顺利的话，为知笔记就运行起来啦。。。。。。

#### 找 WizNote
那么为知笔记编译后的文件在哪呢？
我们可以找到 **WizTeam** 这个文件夹里面多了个 ** build-WizQTClient-Desktop_Qt_5_7_0_GCC_64bit-Debug ** 文件夹，进入文件夹里面找到 **src/WizNote**，没错。。。就是这个了，双击就可以打开为知笔记了。
![找 WizNote](http://1csh1.github.io/img/ubuntu16.04-install-wiznote/find-wiznote.png)


参考文章：
[编译为知笔记客户端](http://www.wiz.cn/compile-client.html)
[Linux编译安装为知笔记](http://jingyan.baidu.com/article/fd8044fa3797ec5031137a9d.html)
[Ubuntu 16.04编译WizNote填坑小记](http://jinbitou.net/2017/01/05/2319.html)

