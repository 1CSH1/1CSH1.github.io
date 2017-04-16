---
title: Thrift封装
date: 2017-03-16 08:00:00
categories: [Thrift]
tags: [Thrift, RPC, 封装]
---

> 摘要：前面几篇文章简单介绍了Thrift的基本知识和小案例。接下来这篇文章讲述在项目中使用Thrift框架作为RPC通信的时候，如何去封装Thrift，使我们在项目中使用的时候可以更简单易用。文中的代码不一定是真的项目应用中的代码，只是写了一个粗糙的例子，提供一个封装思想。

** 目标 **：从前面的文章的几个例子可以看出，我们在调用一个接口的一个方法的时候，需要写多几行代码来设置RPC通信的信息，而我们调用本地的某个类的一个方法的时候，我们只需要2行代码就可以了，比如：
```
HelloService helloService = new HelloService();
helloService.sayHello();
```
我们肯定是希望在使用Thrift作为RPC通信的时候，不管是Server端还是Client端，都可以像上面调用本地方法一样，只需要2行代码就搞定。带着这个目的，我们通过下面的代码来实现它。

** 使用手段 **：当我想到希望不管在Server端还是Client端，都用2行代码来实现Thrift调用的时候，我想到了Spring的AOP，我觉得跟它很相似，首先我们在调用一个方法的时候，需要设置一大堆东西，执行完方法的时候，又要把通讯关掉，这很符合AOP面向切面编程的思维。我们可以通过AOP把除了执行业务的代码给封装掉，让使用的时候只需要new一个对象然后就可以调用它的方法。AOP是用动态代理实现的，而动态代理跟反射息息相关。这个案例可以让我们重新回顾一下这些知识点。

# IDL接口定义
HelloService.thrift
```
namespace java com.thrift.demo.service 
service HelloService {
    i32 sayInt(1:i32 param)
    string sayString(1:string param)
    bool sayBoolean(1:bool param)
    void sayVoid()
}
```

TestService.thrift
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

# Client端
ThriftProxy.java
```
public class ThriftProxy implements InvocationHandler {

	//Client类
	private Class clientClass;
	//Service类
	private Class classs;
	private String[] hostPorts;
	
	public Object newInstance(Class classs, String[] hostPorts) {
		try {
			this.clientClass = Class.forName(classs.getName() + "$Client");
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}
		this.classs = classs;
		this.hostPorts = hostPorts;
		return Proxy.newProxyInstance(clientClass.getClassLoader(), clientClass.getInterfaces(), this);
	}
	
	@Override
	public Object invoke(Object obj, Method method, Object[] objs) throws Throwable {
		if (null != hostPorts && hostPorts.length > 0) {
			for (String hostPort : hostPorts) {
				if (null != hostPort && "" != hostPort.trim()) {
					TSocket socket = null;
					try {
						int index = hostPort.indexOf(":");
						String host = hostPort.substring(0, index);
						String port = hostPort.substring(index+1, hostPort.length());
					    socket = new TSocket(host, Integer.valueOf(port));
						
						TProtocol tProtocol = new TBinaryProtocol(socket);
						TMultiplexedProtocol multiplexedProtocol = new TMultiplexedProtocol(tProtocol, classs.getSimpleName());
						Class[] classes = new Class[]{TProtocol.class};
						socket.open();
						return method.invoke(clientClass.getConstructor(classes).newInstance(multiplexedProtocol), objs);
					} finally {
						if (null != socket) {
							socket.close();
						}
					}
				}
			}
		}
		
		return null;
	}

}
```

ThriftProxyFactory.java
```
public class ThriftProxyFactory {
	
	public static Object newInstance(Class classT, String[] hostPorts) {
		ThriftProxy thriftProxy = new ThriftProxy();
		return thriftProxy.newInstance(classT, hostPorts);
	}

}
```


# Server端
ThriftServer.java
```
public class ThriftServer {

	public static void start(Class[] classes, String localhost, int port) {
		//classes HelloServiceImpl.class
		try {
			Pattern pattern = Pattern.compile("^(.+)\\$Iface$");
			Matcher matcher = null;
			TServerSocket serverTransport = new TServerSocket(port);
			Factory protocolFactory = new TBinaryProtocol.Factory();
			TMultiplexedProcessor processors = new TMultiplexedProcessor();
			
			for (int i=0; i<classes.length; i++) {
				Class<?> classT = classes[i]; // classT HelloServiceImpl.class
				Class<?>[] interfaces = classT.getInterfaces();
				for (Class<?> interfacez : interfaces) {  // interfacez HelloService.Iface
					String interfacezName = interfacez.getName();
					matcher = pattern.matcher(interfacezName);
					if (matcher.find()) {
						String classTName = matcher.group(1);    //classTName HelloService
						Object classTObject = Class.forName(classTName).newInstance(); //classTObject HelloService
						Class iface = Class.forName(classTName + "$Iface");
						Object object = classT.newInstance();
						TProcessor processor = (TProcessor) Class.forName(classTName + "$Processor")
								.getDeclaredConstructor(iface)
								.newInstance(object);
						processors.registerProcessor(classTObject.getClass().getSimpleName(), processor);
					}
				}
			}
			
			TThreadPoolServer.Args args = new TThreadPoolServer.Args(serverTransport);
			args.protocolFactory(protocolFactory);
			args.processor(processors);
			args.minWorkerThreads(1000);
			args.maxWorkerThreads(1000);
			TServer server = new TThreadPoolServer(args);
System.out.println("开启thrift服务器，监听端口：9090");
			server.serve();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
}
```

# 运行 #
Server端启动
```
public static void startServer() {
	ThriftServer.start(new Class[]{HelloServiceImpl.class, TestServiceImpl.class}, "localhost", 9090);
}
```


Client端启动

```
public static void startClient() throws TException {
	String[] hostPorts = new String[]{"localhost:9090"};
	TestService.Iface client = (TestService.Iface) ThriftProxyFactory.newInstance(TestService.class, hostPorts);
	Test test = new Test();
	test.setI32_(100000);
	System.out.println(client.getI32(test));
	
	HelloService.Iface hello = (Iface) ThriftProxyFactory.newInstance(HelloService.class, hostPorts);
	hello.sayString("哈哈哈哈");
}
```

# 结果
我们可以看出，Server端只需要ThriftServer.start(xxx)就可以将Thrift Server启动起来，而Client端也像在调用本地方法一样可以通过2行代码就实现调用远程接口的方法。

参考文章
[thrift生产环境服务端使用的正确姿势](http://www.vccoo.com/v/7cea5b)
[Thrift 个人实战--Thrift 服务化 Client的改造 ](http://www.cnblogs.com/mumuxinfei/p/3876187.html)