---
title: Thrift多接口支持例子
date: 2017-02-27 23:00:00
categories: [Thrift]
tags: [Thrift, Server, RPC]
---

>摘要：本文简单写一个Thrift支持多接口的代码

Thrift通过TMultiplexedProcessor来设置多个服务接口
下面是代码：
# IDL接口
## HelloService.thrift
```
namespace java com.thrift.demo.service 
service HelloService {
    i32 sayInt(1:i32 param)
    string sayString(1:string param)
    bool sayBoolean(1:bool param)
    void sayVoid()
}
```

## TestService.thrift
```
struct Test {
  1: required bool bool_; //required 必须填充也必须序列化
  2: optional i8 i8_; //optional 不填充则不序列化
  3: i16 i16_;
  4: i32 i32_;
  5: i64 i64_;
  6: double double_;
  7: string string_;
  8: binary binary_;
}

service TestService {
  void getVoid(1:Test test);
  bool getBool(1:Test test);
  i8 getI8(1:Test test);
  i16 getI16(1:Test test);
  i32 getI32(1:Test test);
  i64 getI64(1:Test test);
  double getDouble(1:Test test);
  string getString(1:Test test);
  binary getBinary(1:Test test);
}
```

# 接口实现类
## HelloServiceImpl.java
```
public class HelloServiceImpl implements HelloService.Iface {

    public int sayInt(int param) throws TException {
        System.out.println("say int :" + param);
        return param;
    }

    public String sayString(String param) throws TException {
        System.out.println("say string :" + param);
        return param;
    }
    
    public boolean sayBoolean(boolean param) throws TException {
        System.out.println("say boolean :" + param);
        return param;
    }
    public void sayVoid() throws TException {
        System.out.println("say void ...");
    }
}
```

## TestServiceImpl.java
```
public class TestServiceImpl implements TestService.Iface {

	@Override
	public void getVoid(Test test) throws TException {
		System.out.println("TestServiceImpl + " + test.toString());
	}

	@Override
	public boolean getBool(Test test) throws TException {
		System.out.println("TestServiceImpl.getBool() + " + test.bool_);
		return test.bool_;
	}

	@Override
	public byte getI8(Test test) throws TException {
		System.out.println("TestServiceImpl.getI8() + " + test.i8_);
		return test.i8_;
	}

	@Override
	public short getI16(Test test) throws TException {
		System.out.println("TestServiceImpl.getI16() + " + test.i16_);
		return test.i16_;
	}

	@Override
	public int getI32(Test test) throws TException {
		System.out.println("TestServiceImpl.getI32() + " + test.i32_);
		return test.i32_;
	}

	@Override
	public long getI64(Test test) throws TException {
		System.out.println("TestServiceImpl.getI64() + " + test.i64_);
		return test.i64_;
	}

	@Override
	public double getDouble(Test test) throws TException {
		System.out.println("TestServiceImpl.getDouble() + " + test.double_);
		return test.double_;
	}

	@Override
	public String getString(Test test) throws TException {
		System.out.println("TestServiceImpl.getString() + " + test.string_);
		return test.string_;
	}

	@Override
	public ByteBuffer getBinary(Test test) throws TException {
		System.out.println("TestServiceImpl.getBinary() + " + test.binary_);
		return test.binary_;
	}

}
```

# Server端
```
public class ThriftServer {
	public static void main(String[] args) {
		try {
			TServerSocket serverTransport = new TServerSocket(9090);
			Factory protocolFactory = new TBinaryProtocol.Factory();
			//创建多个服务Processor
			Processor<TestService.Iface> processorTest = new TestService.Processor<TestService.Iface>(new TestServiceImpl());
			com.thrift.demo01.service.HelloService.Processor<Iface> processorHello = new HelloService.Processor<HelloService.Iface>(new HelloServiceImpl());
			
			//将服务注册到TMultiplexedProcessor中
			TMultiplexedProcessor processors = new TMultiplexedProcessor();
			processors.registerProcessor("testService", processorTest);
			processors.registerProcessor("helloService", processorHello);
			
			TThreadPoolServer.Args args = new TThreadPoolServer.Args(serverTransport)
								.protocolFactory(protocolFactory)
								.processor(processors)
								.minWorkerThreads(1000)
								.maxWorkerThreads(1000);
			TServer server = new TThreadPoolServer(args);
			System.out.println("开启thrift服务器，监听端口：9090");
			server.serve();
		} catch (TTransportException e) {
			e.printStackTrace();
		}
	}
}
```

# Client端
```
public class ThriftClient {
	public static void testTMultiplexedProcessor() {
		try {
			TTransport transport = new TSocket("localhost", 9090);
			TProtocol protocol = new TBinaryProtocol(transport);
			//通过TMultiplexedProtocol获取对应的服务
			TMultiplexedProtocol protocolTest = new TMultiplexedProtocol(protocol, "testService");
			TMultiplexedProtocol protocolHello = new TMultiplexedProtocol(protocol, "helloService");
			
			//创建Client调用服务接口的方法
			TestService.Client client = new TestService.Client(protocolTest);
			HelloService.Client clientHello = new HelloService.Client(protocolHello);
			transport.open();
			
			//调用接口方法
			Test test = new Test();
			test.setBool_(true);
			test.setDouble_(5.0);
			System.out.println("Client getBool()" + client.getBool(test));
			System.out.println("Client getDouble()" + client.getDouble(test));
		
			System.out.println("HelloService sayString :" + clientHello.sayString("哈哈哈"));
			transport.close();
		} catch (TTransportException e) {
			e.printStackTrace();
		} catch (TException te) {
			te.printStackTrace();
		}
	}
}
```

# 运行结果:
Server端:
>开启thrift服务器，监听端口：9090
TestServiceImpl.getBool() + true
TestServiceImpl.getDouble() + 5.0
say string :哈哈哈

Client端:
>Received 1
Client getBool()true
Received 2
Client getDouble()5.0
Received 1
HelloService sayString :哈哈哈

通过这个方法,就可以在启动一个服务中同时提供多个接口给客户端调用，这种方式在项目开发中可能会用的更多。

参考文章:
[Thrift对多接口服务的支持](http://www.myexception.cn/software-architecture-design/1404363.html)
