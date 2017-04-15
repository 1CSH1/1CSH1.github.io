---
title: R-导入XML数据
date: 2016-05-25 10:23:33
categories: [R]
tags: [R, 导入XML, 技术人生]
---

### 导入XML ###

#### 下载安装XML包 ####

```
install.packages("XML")
```

#### 导入数据 ####

```
library(XML)
doc <- xmlRoot(xmlTreeParse("abc.xml"))
doc #显示xml数据
```

#### 显示导入数据 ####
![显示数据](https://1csh1.github.io/img/R-导入XML数据/显示数据.jpg)