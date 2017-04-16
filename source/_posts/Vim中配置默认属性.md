---
title: Vim中配置默认属性
date: 2017-01-19 21:53:14
categories: [Vim, Linux]
tags: [Linux, Vim, 技术人生]

---
### 编辑设置文件vimrc ###

```
sudo vim /etc/vim/vimrc
```

### 配置行号 ### 
在vimrc文件末尾添加
```
set number
```

### 设置缩进 ###
在vimrc文件末尾添加
```
set cindent shiftwidth=4
```

### 设置Tab键的空格数量 ###
在vimrc文件末尾添加
```
set tabstop=4
```