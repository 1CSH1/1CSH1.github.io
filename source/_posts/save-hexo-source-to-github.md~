---
title: 保存 Hexo 博客源码到 GitHub
date: 2017-04-15 09:00:00
categories: [Hexo]
tags: [Hexo, Github]
---

个人博客原文：[保存 Hexo 博客源码到 GitHub](https://1csh1.github.io/2017/04/15/save-hexo-source-to-github/)

> 摘要：本文介绍在使用 Hexo 博客的时候，怎么保存博客原文。

我们在使用 Hexo 来写博客的时候，我们将博客搭建在 GitHub 上后，当我们使用 hexo d 部署到 GitHub 上，其实没有把我们编写的 xxx.md 原博客文档保存上去，而只是生成一些博客需要的静态的页面。假设我们硬盘坏了，数据无法恢复，那么我们的博客源文档也会不见，所以最好我们是把源代码也托管到 GitHub 上， 这个也有一个好处就是你在其他电脑上也可以同步编写博客。接下来就讲述如何实现。

# 建立分支

因为 master 分支是存放 Hexo 博客生成的页面，所以只能创建一个分支来保存博客的源代码在你的 GitHub 。例如，我创建了一个分支叫 save

![创建分支 save](https://1csh1.github.io/img/save-hexo-source-to-github/hexo01.png)

# 更改默认分支

更改我们的项目的默认分支为 save

![更改默认分支 save](https://1csh1.github.io/img/save-hexo-source-to-github/hexo02.png)

# clone 项目

clone 你的项目到你的 Hexo 目录下。例如我的是执行下面的命令：

```
git clone https://github.com/1CSH1/1CSH1.github.io.git
```

# 执行命令

执行下面的命令将你的博客的原文配置等等保存到分支 save 上

```
git remote add origin https://github.com/1CSH1/1CSH1.github.io.git
git add .
git commit -m "your description"
git push origin save
```

# 配置

确认你的 _config.yml 配置是提交到 master 分支

```
deploy:
  type: git
  repository: git@github.com:1CSH1/1CSH1.github.io.git
  branch: master
```

通过以上的配置，就可以把 Hexo 博客的源文件配置等到保存到 GitHub 上。
