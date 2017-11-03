---
title: MyBatis 在 @Select 写 IN SQL
date: 2017-11-03 22:00:00
categories: [Mybatis]
tags: [MyBatis, SQL]
---


> 摘要：本文简单介绍在 MyBatis 的注解方式中，写包含 in 语法的 SQL

直接了断看下面的代码，SQL 是获取某几个 ID 的文章

```
@Select("<script>" +
            "select * from article where id in " +
            "<foreach item='item' index='index' collection='articleIds' open='(' separator=', ' close=')'>" +
                "#{item}" +
            "</foreach>" +
        "</script>")
List<Article> getArticlesByIds(@Param("articleIds") List<Long> articleIds);
```

