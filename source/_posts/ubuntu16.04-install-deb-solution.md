---
title: Ubuntu 16.04 无法安装.deb解决方案
date: 2017-06-21 23:00:00
categories: [Ubuntu]
tags: [Ubuntu, .deb]
---

> 摘要：本文讲述 Ubuntu 16.04 在刚安装好的时候，下载一些 .deb 软件后，通过 Ubuntu Softwrare Center 无法安装的解决方案。

### 问题
当刚安装好 Ubuntu 16.04 后，发现要安装 chrome 和 搜狗拼音都安装不了，下载的是 .deb 文件，出现图片中的问题。
![Waiting to install](https://1csh1.github.io/img/ubuntu16.04-install-deb-solution/problem.png)

### 解决方案
```
sudo apt-get install gdebi
```
接着在右击你要安装的 .deb 文件，选择 **Open With ---> GDebi Package Installer**这时会有如下窗口，点击 **Install Package**
![这里写图片描述](https://1csh1.github.io/img/ubuntu16.04-install-deb-solution/install-package.png)

有如下结果代表安装成功！！！
![这里写图片描述](https://1csh1.github.io/img/ubuntu16.04-install-deb-solution/installation-finished.png)

### 参考文章：
[16.04 Cannot install anything from Ubuntu Software center [duplicate]](https://askubuntu.com/questions/761210/16-04-cannot-install-anything-from-ubuntu-software-center)
