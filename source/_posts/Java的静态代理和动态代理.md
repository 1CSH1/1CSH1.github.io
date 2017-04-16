---
title: Java的静态代理和动态代理
date: 2017-02-20 22::24:20
categories: [Java]
tags: [Java, 静态代理, 动态代理, Cglib]
---

> 摘要：本文讲述Java中的静态代理和动态代理，动态代理的2种实现：Java自带的动态代理实现以及Cglib的动态代理实现。

# 静态代理

> 由程序员创建或工具生成代理类的源码，再编译代理类。所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。

## 例子
```
//User实体类
public class User {
	
	private int id;
	private String name;
	private int age;
	
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	@Override
	public String toString() {
		return "User [id=" + id + ", name=" + name + ", age=" + age + "]";
	}
}
```

```
//用户操作接口
public interface UserService {

	void add(User user);
	// 。。。
	
}
```

```
//接口真实实现
public class UserServiceImpl implements UserService {

	@Override
	public void add(User user) {
		System.out.println("UserServiceImpl : add user" + user.toString() );
	}

}
```

```
//代理类
public class ProxyUserService implements UserService {
	
	private UserService userService;

	public ProxyUserService(UserService userService) {
		super();
		this.userService = userService;
	}
	
	public void add(User user) {
		System.out.println("Static Before add");
		userService.add(user);
		System.out.println("Static After add");
	}

}
```

```
//代理工厂类
public class ProxyStaticFactory {

	public static UserService getInstance() {
		return new ProxyUserService(new UserServiceImpl());
	}
	
}
```

```
//测试
public class TestStaticProxy {

	public static void main(String[] args) {
		UserService userService = ProxyStaticFactory.getInstance();
		User user = new User();
		user.setId(12);
		user.setName("哈哈");
		user.setAge(23);
		userService.add(user);
	}
	
}
```

> 结果：
Static Before add
UserServiceImpl : add userUser [id=12, name=哈哈, age=23]
Static After add


## 静态代理类优缺点 
### 优点：
业务类只需要关注业务逻辑本身，保证了业务类的重用性。这是代理的共有优点。 
### 缺点： 
1）代理对象的一个接口只服务于一种类型的对象，如果要代理的方法很多，势必要为每一种方法都进行代理，静态代理在程序规模稍大时就无法胜任了。 
2）如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。 


# 动态代理
>动态代理类的源码是在程序运行期间由JVM根据反射等机制动态的生成，所以不存在代理类的字节码文件。代理类和委托类的关系是在程序运行时确定。 

## 例子

```
//代理类
public class UserServiceInvocationHandler implements InvocationHandler {

	private Object delegate;
	
	public UserServiceInvocationHandler(Object delegate) {
		super();
		this.delegate = delegate;
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("Dynamic Before add");
		Object object = method.invoke(delegate, args);
		System.out.println("Dynamic After add");
		return object;
	}

}
```

```
//动态代理工厂类
public class ProxyDynamicFactory {

	public static UserService getInstance() {
		UserService userService = new UserServiceImpl();
		UserServiceInvocationHandler handler = new UserServiceInvocationHandler(userService);
		return (UserService) Proxy.newProxyInstance(userService.getClass().getClassLoader(), userService.getClass().getInterfaces(), handler);
	}
	
}
```

```
//测试
public class TestDynamicProxy {
	
	public static void main(String[] args) {
		UserService userService = ProxyDynamicFactory.getInstance();
		User user = new User();
		user.setId(12);
		user.setName("哈哈");
		user.setAge(23);
		userService.add(user);
	}
	
}
```

## 创建动态代理过程：
一个典型的动态代理创建对象过程可分为以下四个步骤：
> 1、通过实现InvocationHandler接口创建自己的调用处理器 IvocationHandler handler = new InvocationHandlerImpl(...);
2、通过为Proxy类指定ClassLoader对象和一组interface创建动态代理类
Class clazz = Proxy.getProxyClass(classLoader,new Class[]{...});
3、通过反射机制获取动态代理类的构造函数，其参数类型是调用处理器接口类型
Constructor constructor = clazz.getConstructor(new Class[]{InvocationHandler.class});
4、通过构造函数创建代理类实例，此时需将调用处理器对象作为参数被传入
Interface Proxy = (Interface)constructor.newInstance(new Object[] (handler));
为了简化对象创建过程，Proxy类中的newInstance方法封装了2~4，只需两步即可完成代理对象的创建。
生成的ProxySubject继承Proxy类实现Subject接口，实现的Subject的方法实际调用处理器的invoke方法，而invoke方法利用反射调用的是被代理对象的的方法（Object result=method.invoke(proxied,args)）

## 动态代理的优点和缺点 
### 优点： 
动态代理与静态代理相比较，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。在本示例中看不出来，因为invoke方法体内嵌入了具体的外围业务（记录任务处理前后时间并计算时间差），实际中可以类似Spring AOP那样配置外围业务。 

### 缺点：
诚然，Proxy 已经设计得非常优美，但是还是有一点点小小的遗憾之处，那就是它始终无法摆脱仅支持 interface 代理的桎梏，因为它的设计注定了这个遗憾。回想一下那些动态生成的代理类的继承关系图，它们已经注定有一个共同的父类叫 Proxy。Java 的继承机制注定了这些动态代理类们无法实现对 class 的动态代理，原因是多继承在 Java 中本质上就行不通。 
有很多条理由，人们可以否定对 class 代理的必要性，但是同样有一些理由，相信支持 class 动态代理会更美好。接口和类的划分，本就不是很明显，只是到了 Java 中才变得如此的细化。如果只从方法的声明及是否被定义来考量，有一种两者的混合体，它的名字叫抽象类。实现对抽象类的动态代理，相信也有其内在的价值。此外，还有一些历史遗留的类，它们将因为没有实现任何接口而从此与动态代理永世无缘。如此种种，不得不说是一个小小的遗憾。 



# CGLib的动态代理     

>JDK实现动态代理需要实现类通过接口定义业务方法，对于没有接口的类，如何实现动态代理呢，这就需要CGLib了。CGLib采用了非常底层的字节码技术，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑

## 例子

```
//pom.xml
<dependency>
	<groupId>cglib</groupId>
	<artifactId>cglib</artifactId>
	<version>3.2.4</version>
</dependency>
```

```
//代理类
public class UserServiceCglibMethodInterceptor implements MethodInterceptor {

	@Override
	public Object intercept(Object object, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
		System.out.println("Cglib Before add");
		Object result = methodProxy.invokeSuper(object, objects);
		System.out.println("Cglib After add");
		return result;
	}

}
```


```
//代理工厂
public class ProxyCglibFactory {

	public static UserService getInstance() {
		Enhancer enhancer = new Enhancer();  
		enhancer.setSuperclass(UserServiceImpl.class);  
		enhancer.setCallback(new UserServiceCglibMethodInterceptor());  
		return (UserServiceImpl)enhancer.create();
	}
	
}
```

```
//测试
public class TestCglibProxy {

	public static void main(String[] args) {
		UserService userService = ProxyCglibFactory.getInstance();
		User user = new User();
		user.setId(12);
		user.setName("哈哈");
		user.setAge(23);
		userService.add(user);
	}
	
}
```

> 结果：
Cglib Before add
UserServiceImpl : add userUser [id=12, name=哈哈, age=23]
Cglib After add


## 代理对象的生成过程由Enhancer类实现，大概步骤如下：
> 1、生成代理类Class的二进制字节码；
2、通过Class.forName加载二进制字节码，生成Class对象；
3、通过反射机制获取实例构造，并初始化代理类对象



# Java自带的动态代理和CGlib的动态代理实现的区别

> 1、jdk动态代理生成的代理类和委托类实现了相同的接口；
2、cglib动态代理中生成的字节码更加复杂，生成的代理类是委托类的子类，且不能处理被final关键字修饰的方法；
3、jdk采用反射机制调用委托类的方法，cglib采用类似索引的方式直接调用委托类方法；


# 参考文章
[说说cglib动态代理 ](http://www.cnblogs.com/chinajava/p/5880887.html)
[Java 代理模式（一） 静态代理](http://www.cnblogs.com/mengdd/archive/2013/01/30/2883468.html)
[JAVA学习篇--静态代理VS动态代理 ](http://blog.csdn.net/hejingyuan6/article/details/36203505)
[java静态代理和动态代理 ](http://layznet.iteye.com/blog/1182924)
[Java 动态代理机制分析及扩展，第 1 部分](http://www.ibm.com/developerworks/cn/java/j-lo-proxy1/index.html)
[JDK动态代理实现原理](http://rejoy.iteye.com/blog/1627405)
[彻底理解JAVA动态代理](http://www.cnblogs.com/flyoung2008/archive/2013/08/11/3251148.html)