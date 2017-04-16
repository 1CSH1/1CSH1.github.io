---
title: Memcached简单介绍
date: 2016-03-21 19:49:54
categories: Memcached
tags: [Memcached, 技术人生]
---
**Memcached是什么**   
&emsp;&emsp;Memcached 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。它通过在内存中缓存数据和对象 来减少读取数据库的次数，从而提供动态、数据库驱动网站的速度。
&emsp;&emsp;相信很多人都用过缓存，在JavaWeb开发中有Ehcache缓存等等，还有很多第三方工具如apache，nginx等可以做静态资源的缓存，同时我们也可以制定自己的缓存机制，缓存数据库查询的数据以减少对数据库的频繁操作。但是很多时候我们总是感觉这些缓存总不尽人意， Memcached可以解决你不少的烦恼问题。 
&emsp;&emsp;Memcached基于一个存储键/值对的hashmap。其守护进程是用C写的，但是客户端可以用任何语言来编写(本文使用Java作为例子)，并通过memcached协议与守护进程通信。

**Memcached使用目的**
&emsp;&emsp;通过缓存数据库查询结果，减少数据库访问次数，以提高动态Web应用的速度、提高可扩展性。

**Memcached应用机制**
&emsp;&emsp;Memcached作为一个分布式内存对象缓存系统，它可以将缓存和应用分离开来，但是每个Memcached服务不会互相通信，而是客户端根据分布式算法将数据库数据保存到不同的Memcached服务器上。
![Memcached应用机制](https://1csh1.github.io/img/Memcached简单介绍/Memcached应用机制.jpg)

**Memcached特点**  
&emsp;&emsp;1.协议简单  
&emsp;&emsp;2.基于libevent的事件处理
&emsp;&emsp;3.内置内存存储方式
&emsp;&emsp;4.Memcached不互相通信的分布式，数据都是通过客户端的分布式算法存储到各个服务器中

**其他知识点**
&emsp;&emsp;Memcached的API使用32位元的循环冗余校验（CRC-32）计算键值后，将资料分散在不同的机器上。当表格满了以后，接下来新增的资料会以LRU机制替换掉。由于memcached通常只是当作快取系统使用，所以使用memcached的应用程式在写回较慢的系统时（像是后端的数据库）需要额外的程式码更新memcached内的资料
&emsp;&emsp;存储方式：为了提高性能，memcached中保存的数据都存储在memcached内置的内存存储空间中。由于数据仅存在于内存中，因此重启memcached、重启操作系统会导致全部数据消失。另外，内容容量达到指定值之后，就基于LRU(Least Recently Used)算法自动删除不使用的缓存。memcached本身是为缓存而设计的服务器，因此并没有过多考虑数据的永久性问题。

**Java代码实现Memcached基本操作**
&emsp;&emsp;需要导入jar包：spymemcached
下载链接：[spymemcached-2.12.0.jar](https://1csh1.github.io/file/Memcached简单介绍/spymemcached-2.12.0.jar)


&emsp;&emsp;首先需要启动Memcached服务，我是在Linux中测试的：启动命令如下
```
memcached -p 11211 -m 64m -d
```
&emsp;&emsp;Java客户端测试代码：
```
public static void main(String[] args){
    try {
        MemcachedClient mcc = new MemcachedClient(new InetSocketAddress("127.0.0.1", 11211));
        System.out.println("connection success");
        
        // set 向Memcached中设置缓存对象【如果缓存中已有该key的缓存对象，则替换原来的缓存对象】
        Future future = mcc.set("runoob", 900, "Free Education");
        System.out.println("status : " + future.get());
        System.out.println("runoob in cache : " + mcc.get("runoob"));
        
        // add 想Memcached中添加缓存对象【如果缓存中已有该key的缓存对象，则不添加】
        Future future1 = mcc.add("test2", 10, "test2-value");
        System.out.println("test2 in cache : " + mcc.get("test2"));
        Future future2 = mcc.add("test3", 1, "test3-value");
        System.out.println("test2 status : " + future1.get());
        System.out.println("test2 in cache : " + mcc.get("test2"));
        
        Thread.sleep(1000);
        //test3只缓存1秒，测试test3超时后还有没存在，结果是不存在了
        System.out.println("after sleep 1s test3 in cache : " + mcc.get("test3"));
        
        //replace 替换掉原来缓存的对象
        System.out.println("before replace : " + mcc.get("runoob"));
        mcc.replace("runoob", 90, "replace value");
        System.out.println("after replace : " + mcc.get("runoob"));
        
        //append 在相应缓存对象后面追加内容
        System.out.println("before append : " + mcc.get("runoob"));
        mcc.append("runoob", " abc ");
        System.out.println("after append : " + mcc.get("runoob"));
        
        //prepend 在相应缓存对象的前面添加内容
        System.out.println("before prepend : " + mcc.get("runoob"));
        mcc.prepend("runoob", " 123 ");
        System.out.println("after prepend : " + mcc.get("runoob"));
        
        //CAS 检查并设置操作
        System.out.println("before cas : " + mcc.get("runoob"));
        //获取token牌
        CASValue casValue = mcc.gets("runoob");
        System.out.println("CASValue : " + casValue);
        CASResponse casResponse = mcc.cas("runoob", casValue.getCas(), 90, "runoob cas");
        System.out.println("CASResponse : " + casResponse);
        
        // delete 删除掉缓存数据
        System.out.println("before delete : " + mcc.get("runoob"));
        mcc.delete("runoob");
        System.out.println("after delete : " + mcc.get("runoob"));
        
        // incr  将缓存数据自增
        mcc.set("num", 10, 100);
        System.out.println("before incr : " + mcc.get("num"));
        System.out.println(mcc.get("num") instanceof Integer);
        mcc.incr("num", 10);
        System.out.println("after incr : " + mcc.get("num"));
        
        //decr 将缓存数据自减
        System.out.println("before decr : " + mcc.get("num"));
        mcc.decr("num", 10);
        System.out.println("after decr : " + mcc.get("num"));
        mcc.shutdown();
        
        // test another memcachedclient can get the object
        MemcachedClient mcc1 = new MemcachedClient(new InetSocketAddress("node1", 11211));
        System.out.println(mcc1.get("runoob"));
        mcc1.shutdown();
    } catch (IOException e) {
        e.printStackTrace();
        System.out.println("connection fail");
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```

**参考文章链接**
[缓存应用--Memcached分布式缓存简介](http://www.cnblogs.com/qingyuan/archive/2011/01/17/1937855.html)
