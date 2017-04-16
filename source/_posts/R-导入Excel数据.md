---
title: R-导入Excel数据
date: 2016-05-24 16:23:30
categories: [R]
tags: [R, 导入Excel, 技术人生]
---

### 导入2007前的Excel ###

#### 下载安装RODBC包 ####

```
install.packages("RODBC")
```

#### Window 32-bit 导入数据 ####

```
library(RODBC)
channel <- odbcConnectExcel("myfile.xls")
mydataframe <- sqlFetch(channel, "mysheet")
odbcClose(channel)
```

#### Window 64-bit 导入数据 ####

```
library(RODBC)
channel <- odbcConnectExcel2007("myfile.xls")
mydataframe <- sqlFetch(channel, "mysheet")
odbcClose(channel)
```

### 导入2007 Excel ###

#### 下载安装xlsx包 ####

```
install.packages("xlsx")
```

#### 导入数据 ####

```
library(xlsx)
workbook <- "abc.xlsx"
mydataframe <- read.xlsx(workbook, 1)
mydataframe #显示Excel数据
```

#### 显示导入数据 ####
![显示数据](https://1csh1.github.io/img/R-导入Excel数据/显示数据.jpg)