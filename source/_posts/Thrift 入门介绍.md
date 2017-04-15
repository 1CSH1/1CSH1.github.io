---
title: Thrift 入门技术
date: 2017-02-21 19:50:00
categories: [Thrift]
tags: [Thrift, RPC]
---

> 摘要：本文讲述Thrift的一些基本知识，以及概念。

# 介绍
&emsp;&emsp;Thrift作为Facebook开源的RPC框架, 通过IDL中间语言, 并借助代码生成引擎生成各种主流语言的rpc框架服务端/客户端代码，主要特点：

## 开发速度快：
&emsp;&emsp;通过编写RPC接口IDL文件，利用编译生成器自动生成Server端骨架(Skeletons)和客户端Stubs，省去开发者自定义和维护接口编解码、消息传输、服务器多线程模型等基础工作；Server端开发者只需按照服务骨架，写好自己的业务处理程序(Handlers)即可，Client端程序只需创建IDL中定义的服务对象，然后就像调用本地对象的方法一样调用远端服务。

## 接口维护简单高效：
&emsp;&emsp;通过维护Thrift格式的IDL(Interface Description Language)文件（注意写好注释），即可作为给Clients使用的接口文档使用，也自动生成接口代码，始终保持代码和文档的一致性。且Thrift协议可灵活支持接口的可扩展性。

## 学习成本低：
&emsp;&emsp;因为其来自Google Protocol Buffers开发团队，所以其IDL文件风格类似Google Protocol Buffers，且更加易读易懂；特别是RPC服务接口的风格就像写一个一般的面向对象的Class一样简单。
&emsp;&emsp;初学者只需参照http://thrift.apache.org/几个小时即可理解和使用Thrift。

## 多语言/跨语言支持：
&emsp;&emsp;Thrift支持C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, JavaScript, Node.js, Smalltalk, OCaml and Delphi等多种语言，即可生成上述语言的服务器端和客户端程序。
对于我们经常使用的Java、PHP、Python、C++支持良好，虽然对iOS环境的Objective-C(Cocoa)支持稍逊，但也完全满足我们的使用要求。

## 已验证成熟稳定：
&emsp;&emsp;Thrift在很多开源项目中已经被验证是稳定和高效的，例如Cassandra、Evernode等；在Facebook、Baidu等后台产品中也有使用。

# 基础知识
## Thrift 软件栈
&emsp;&emsp; Thrift对软件栈的定义非常的清晰, 使得各个组件能够松散的耦合, 针对不同的应用场景, 选择不同是方式去搭建服务.
&emsp;&emsp; Transport: 传输层，定义数据传输方式，可以为TCP/IP传输，内存共享或者文件共享等
&emsp;&emsp; Protocol: 协议层, 定义数据传输格式，可以为二进制或者XML等
&emsp;&emsp; Processor: 处理层, 这部分由定义的idl来生成, 封装了协议输入输出流, 并委托给用户实现的handler进行处理.
&emsp;&emsp; Server: 服务层, 整合上述组件, 提供网络模型(单线程/多线程/事件驱动), 最终形成真正的服务.

## Thrift 对语言的支持
&emsp;&emsp;Thrift和Google Protobuf相比, 都提供了可扩展序列化机制, 不但兼容性好而且压缩率高. 两者在这块各有长短, 性能上PB稍有优势. 但在语言的支持度上, Protobuf只支持c++/java/python这三种主流的语言, Thrift则几乎覆盖了大部分的语言, 从这点上来说, Thrift的优势非常的明显.

## 协议
&emsp;&emsp;Thrift可以让你选择客户端与服务端之间传输通信协议的类别，在传输协议上总体上划分为文本(text)和二进制(binary)传输协议, 为节约带宽，提供传输效率，一般情况下使用二进制类型的传输协议为多数，但有时会还是会使用基于文本类型的协议，这需要根据项目/产品中的实际需求：
&emsp;&emsp; TBinaryProtocol – 二进制编码格式进行数据传输。
&emsp;&emsp; TCompactProtocol – 这种协议非常有效的，使用Variable-Length Quantity (VLQ) 编码对数据进行压缩。
&emsp;&emsp; TJSONProtocol – 使用JSON的数据编码协议进行数据传输。
&emsp;&emsp; TSimpleJSONProtocol – 这种节约只提供JSON只写的协议，适用于通过脚本语言解析
&emsp;&emsp; TDebugProtocol – 在开发的过程中帮助开发人员调试用的，以文本的形式展现方便阅读。

## 传输层
&emsp;&emsp; TSocket- 使用堵塞式I/O进行传输，也是最常见的模式。
&emsp;&emsp; TFramedTransport- 使用非阻塞方式，按块的大小，进行传输，类似于Java中的NIO。
&emsp;&emsp; TFileTransport- 顾名思义按照文件的方式进程传输，虽然这种方式不提供Java的实现，但是实现起来非常简单。
&emsp;&emsp; TMemoryTransport- 使用内存I/O，就好比Java中的ByteArrayOutputStream实现。
&emsp;&emsp; TZlibTransport- 使用执行zlib压缩，不提供Java的实现。

## 服务端类型
&emsp;&emsp; TSimpleServer -  单线程服务器端使用标准的堵塞式I/O。
&emsp;&emsp; TThreadPoolServer -  多线程服务器端使用标准的堵塞式I/O。
&emsp;&emsp; TNonblockingServer – 多线程服务器端使用非堵塞式I/O，并且实现了Java中的NIO通道。

# Thrift 架构
<center>
![Thrift架构图](https://1csh1.github.io/img/Thrift入门介绍/Thrift架构图.jpg)
</center>

&emsp;&emsp;如图所示，图中黄色部分是用户实现的业务逻辑，褐色部分是根据 Thrift 定义的服务接口描述文件生成的客户端和服务器端代码框架，红色部分是根据 Thrift 文件生成代码实现数据的读写操作。红色部分以下是 Thrift 的传输体系、协议以及底层 I/O 通信，使用 Thrift 可以很方便的定义一个服务并且选择不同的传输协议和传输层而不用重新生成代码。
&emsp;&emsp;Thrift 服务器包含用于绑定协议和传输层的基础架构，它提供阻塞、非阻塞、单线程和多线程的模式运行在服务器上，可以配合服务器 / 容器一起运行，可以和现有的 J2EE 服务器 /Web 容器无缝的结合。

# 参考文章
[Thrift RPC实战(一) 初次体验Thrift](http://www.jianshu.com/p/bfd514fd0d6d)
[Apache Thrift - 可伸缩的跨语言服务开发框架](https://www.ibm.com/developerworks/cn/java/j-lo-apachethrift/)

