---
title: Ubuntu 16.04 设置菜单栏位置
date: 2017-06-21 22:10:00
categories: [Ubuntu]
tags: [Ubuntu]
---

> 摘要：本文讲述在 Ubuntu 16.04 中如何设置菜单栏在桌面的下边或者左边。

设置在下边
```
gsettings set com.canonical.Unity.Launcher launcher-position Bottom
```
效果如图：
![下边](https://1csh1.github.io/img/ubuntu16.04-set-menubar/bottom.png)



设置在左边
```
gsettings set com.canonical.Unity.Launcher launcher-position Left
```
![左边](https://1csh1.github.io/img/ubuntu16.04-set-menubar/left.png)

目前不支持上边和右边。。。。
