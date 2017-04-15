---
title: Linux下MySQL导入导出数据
date: 2016-04-10 21:04:28
categories: [MySQL]
tags: [MySQL, 技术人生]
---

**导入数据**

```
create database test;
use test;
source /home/xxx/data/test.sql
```

**导出数据**

```
bin/mysqldump -u username -p [-d] database_name > xxx.sql
```

如果没有-d， 则导出表结构和数据；如果有-d，则只导出表结构

例如：
```
bin/mysqldump -u root -p test > test.sql
```

