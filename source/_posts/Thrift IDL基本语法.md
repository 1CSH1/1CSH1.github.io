---
title: Thrift IDL基本语法
date: 2017-02-21 20:30:00
categories: [Thrift]
tags: [Thrift, RPC]
---

>摘要：本文介绍Thrift的IDL基本语法

# IDL
>Thrift 采用IDL（Interface Definition Language）来定义通用的服务接口，然后通过Thrift提供的编译器，可以将服务接口编译成不同语言编写的代码，通过这个方式来实现跨语言的功能。
 
# 基本类型

>bool: 布尔值          对应Java中的boolean
byte: 有符号字节      对应Java中的byte  
i16: 16位有符号整型   对应Java中的short
i32: 32位有符号整型   对应Java中的int
i64: 64位有符号整型   对应Java中的long
double: 64位浮点型    对应Java中的double
string: 字符串        对应Java中的String
binary: Blob 类型     对应Java中的byte[]

 
# struct 结构体

struct有以下一些约束：

>1.struct不能继承，但是可以嵌套，不能嵌套自己。
2.其成员都是有明确类型
3.成员是被正整数编号过的，其中的编号使不能重复的，这个是为了在传输过程中编码使用。
4.成员分割符可以是逗号（,）或是分号（;），而且可以混用
5.字段会有optional和required之分和protobuf一样，但是如果不指定则为无类型–可以不填充该值，但是在序列化传输的时候也会序列化进去，optional是不填充则部序列化，required是必须填充也必须序列化。
6.每个字段可以设置默认值
7.同一文件可以定义多个struct，也可以定义在不同的文件，进行include引入。


例子：
``` 
struct User{
  1: required string name, //改字段必须填写
  2: optional i32 age = 0; //默认值
  3: bool gender //默认字段类型为optional
}
``` 

规则：
>如果required标识的域没有赋值，Thrift将给予提示；
如果optional标识的域没有赋值，该域将不会被序列化传输；
如果某个optional标识域有缺省值而用户没有重新赋值，则该域的值一直为缺省值；
如果某个optional标识域有缺省值或者用户已经重新赋值，而不设置它的__isset为true，也不会被序列化传输。


# Container (容器)
有3种可用容器类型：

>list&lt;t>:  元素类型为t的有序表，容许元素重复。对应c++的vector，java的ArrayList或者其他语言的数组
set&lt;t>:   元素类型为t的无序表，不容许元素重复。对应c++中的set，java中的HashSet,python中的set，php中没有set，则转换为list类型了
map&lt;t, t>: 键类型为t，值类型为t的kv对，键不容许重复。对用c++中的map, Java的HashMap, PHP 对应 array, Python/Ruby 的dictionary


例子
``` 
struct Test {
  1: map<string, User> usermap,
  2: set<i32> intset,
  3: list<double> doublelist
}
``` 

 
# enum (枚举)

约束：
>1.编译器默认从0开始赋值
2.可以赋予某个常量某个整数
3.允许常量是十六进制整数
4.末尾没有分号
5.给常量赋缺省值时，使用常量的全称


规则：
>Thrift不支持枚举类嵌套，枚举常量必须是32位的正整数


例子：
``` 
enum HttpStatus {
  OK = 200,
  NOTFOUND=404
}
``` 

 
# 常量定义

使用方法：在变量前面加上const

例子：

``` 
const i32 const_int = 1;
``` 


# 类型定义
Thrift支持C/C++类型定义

例子：
``` 
typedef i32 myint
typedef i64 usernumber
``` 

规则：
> 末尾没有逗号
 
# Exception (异常)

异常在语法和功能上类似于结构体，差别是异常使用关键字exception，而且异常是继承每种语言的基础异常类。

例子：
``` 
exception MyException {
    1: i32 errorCode,
    2: string message
}
``` 
 
 
# Service (服务定义类型)

服务的定义方法在语义上等同于面向对象语言中的接口。
``` 
service HelloService {
    i32 sayInt(1:i32 param)
    string sayString(1:string param)
    bool sayBoolean(1:bool param)
    void sayVoid()
}
``` 


编译后的Java代码
``` 
public class HelloService {
  public interface Iface {
    public int sayInt(int param) throws org.apache.thrift.TException;
    public java.lang.String sayString(java.lang.String param) throws org.apache.thrift.TException;
    public boolean sayBoolean(boolean param) throws org.apache.thrift.TException;
    public void sayVoid() throws org.apache.thrift.TException;
  }
  // ... 省略很多代码
}
``` 
 
# Namespace (名字空间)

>Thrift中的命名空间类似于C++中的namespace和java中的package，它们提供了一种组织（隔离）代码的简便方式。名字空间也可以用于解决类型定义中的名字冲突。
由于每种语言均有自己的命名空间定义方式（如python中有module）, thrift允许开发者针对特定语言定义namespace：

例子：
``` 
namespace java com.example.test
``` 
 
转化成
``` 
package com.example.test
``` 

 
# Comment (注释)

Thrift支持C多行风格和Java/C++单行风格。
例子：
``` 
/** 
 * This is a multi-line comment. 
 * Just like in C. 
 */
 // C++/Java style single-line comments work just as well.
``` 
 
# Include

>便于管理、重用和提高模块性/组织性，我们常常分割Thrift定义在不同的文件中。包含文件搜索方式与c++一样。Thrift允许文件包含其它thrift文件，用户需要使用thrift文件名作为前缀访问被包含的对象，

如：
``` 
include "test.thrift"   
...
struct StSearchResult {
    1: in32 uid;
     ...
}
 ``` 

thrift文件名要用双引号包含，末尾没有逗号或者分号


# 参考文章
[Apache Thrift - 可伸缩的跨语言服务开发框架](https://www.ibm.com/developerworks/cn/java/j-lo-apachethrift/)
