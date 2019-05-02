---
title: Mybatis源码分析1
date: 2018-7-9 14:18:40
categories:
	- Mybatis
tags:
	- Mybatis
---

摘要：Mybatis源码学习1
<!-- more -->

暑假看完了《深入Mybatis技术原理》，在这里写了一点对Mybatis运行原理的一点总结。
## Mybatis的基本构成
主要包含了下面几个重要的组件：
* SqlSessionFactoryBuilder 这是一个构造器，他会根据我们配置单XML文件来生成SqlSessionFactory（工程类）
* SqlSessionFactory 来生成回话(SqlSession)
* SqlSession 可以发送SQL执行并获得结果，可以获取Maper的接口
* SQLMapper Mybatis最重要的构成，它是由Java Interface 和一个对应的XML文件构成的
### 构造SqlSessionfactory
原生可以使用这样的方式
```
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```
XML 配置文件（configuration XML）中包含了对 MyBatis 系统的核心设置，包含获取数据库连接实例的数据源（DataSource）和决定事务作用域和控制方式的事务管理器（TransactionManager）。
### 从SqlSessionFactory 中获取Sqlsession
```
SqlSession session = sqlSessionFactory.openSession();
try {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
} finally {
  session.close();
}
```
## 作用域（Scope）和生命周期
### SqlSessionFactoryBuilder
单例，一经创建，就不再需要了。
### SqlSessionFactory

单例
### SqlSessionFactory 
一旦被创建就应该在应用的运行期间一直存在，没有任何理由对它进行清除或重建
### SqlSession

每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。
```
SqlSession session = sqlSessionFactory.openSession();
try {
  // do work
} finally {
  session.close();
}
```
映射器实例（Mapper Instances）

映射器是一个你创建来绑定你映射的语句的接口。映射器接口的实例是从 SqlSession 中获得的。
```
SqlSession session = sqlSessionFactory.openSession();
try {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  // do work
} finally {
  session.close();
}
```
## Mybatis重点
技术重点就是我们经常使用的Mapper了， 其中Mapper.xml的编写，是重中之重，能写出好的xml文件，需要动态SQl， Insert，delete,update,select 还有重要的resultMap。
## 动态SQL 
if 元素  
choose，when, otherwise  
trim, where, set元素
foreach元素
## 运行原理

前面说了那么多的废话，熟悉Java设计语言的程序员，看着Mybatis的官方文档，还是比较容易的。但是只会使用这个持久层的框架还远远不够，我们还需要去对她的内部源码一探究竟。
Mybatis的插件使用几乎都是构建在Mybatis的原理之上的，不懂原理基本无法使用Mybatis的插件。
Mybatis的运行分为两大部分， 第一分部是读取配置文件，构建Configration对象，用以创建SqlSessionFactory。第二部分是Sqlsession的执行过程。  
### 动态代理的两种方式
JDK原生的代理，和CGlib的动态代理是Mybatis的实现技术。
```java
public class MyProxy implements InvocationHandler {

	Object obj;
	public Object bind(Object obj) {
		this.obj = obj;
		return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), this);
	}
	
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println(" l am a proxy");
		Object res = method.invoke(obj, args);
//		Arrays.copyOf(original, newLength)
		return res;
	}

}
```
里面的重点自然是Proxy类，来产生一个代理对象。用着代理对象执行真实对象的方法时，invoke方法会进行拦截，方法中的三个参数含义为：
1. proxy 被代理对象
2. method 被拦截的方法
3. args 方法参数

要想使用CGlib，在poem.xml里面加入如下依赖就行了。
```
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>2.2.2</version>
</dependency>
```
```java
public class Intercepter implements MethodInterceptor{

	public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
		// TODO Auto-generated method stub
		System.out.println("before.........");
		
		Object res = proxy.invokeSuper(obj, args);
		System.out.println("after........");
		return res;
	}

}

Hello h = new Hello();
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(h.getClass());
enhancer.setCallback(new Intercepter());
Hello proxy = (Hello)enhancer.create();
proxy.hello(); // 就可以生成真实对象的子类来代理真实对象了
```
### 构建Configration
在SqlSessionFactory构建中，Configration是很重要的。
* 读入配置文件，包括基础配置的XML文件和映射器的XML文件
* 初始化基础配置，例如别名
* 提供单例，为后面创建SessionFactory服务提供配置参数
* 执行一些重要的对象方法，初始化配置信息

Configration是通过XMLConfigBuilder来创建的，
### 映射器的内部组成
* MapperStatement 保存映射器的一个节点(slect, insert, delete, update),包含了我们配置的SQL，Sql的ID，缓存信息，resultMap，parameterType,resultType等内容
* SqlSource 是MapperStatement的一个属性，提供BoundSQL的地方
* BoundSQL，建立SQL和参数，
### SqlSession的运行过程
他是通过动态代理实现的，用`MapperProxy`类生成代理对象，
```java
public class MapperProxyFactory<T> {

  @SuppressWarnings("unchecked")
  protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }
	// MapperProxy
  public T newInstance(SqlSession sqlSession) {
    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
  }

}
```
MapperProxy的源码如下
```java
public class MapperProxy<T> implements InvocationHandler, Serializable {

  private static final long serialVersionUID = -6424540398559729838L;
  private final SqlSession sqlSession;
  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethod> methodCache;
	/// 构造函数
  public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, MapperMethod> methodCache) {
    this.sqlSession = sqlSession;
    this.mapperInterface = mapperInterface;
    this.methodCache = methodCache;
  }

// 需要先判断是 class 还是 接口 
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (Object.class.equals(method.getDeclaringClass())) {
      try {
        return method.invoke(this, args);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    }
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    return mapperMethod.execute(sqlSession, args); //开始执行mappermethod
  }
`// --> 上面的if如果不成立的话，就会执行下面的方法来创建mappernethod
  private MapperMethod cachedMapperMethod(Method method) {
    MapperMethod mapperMethod = methodCache.get(method);
    if (mapperMethod == null) {
      mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
      methodCache.put(method, mapperMethod);
    }
    return mapperMethod;
  }

  ```
上面的代码大概就是，执行invoke方法，如果mapper是一个代理对象，就会运行invoke方法，接着判定Mapper是一个接口，不是类，失败通过CacheMapperMethod来生成MAPPMethod对象，执行execute方法，把当前的SqlSession和运行参数传进去，
继续追踪这个Mappermethod，我们终于看到了SQL语句执行的代码
```java
public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    if (SqlCommandType.INSERT == command.getType()) {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.insert(command.getName(), param));
    } else if (SqlCommandType.UPDATE == command.getType()) {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.update(command.getName(), param));
    } else if (SqlCommandType.DELETE == command.getType()) {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.delete(command.getName(), param));
    } else if (SqlCommandType.SELECT == command.getType()) {
      if (method.returnsVoid() && method.hasResultHandler()) {
        executeWithResultHandler(sqlSession, args);
        result = null;
      } else if (method.returnsMany()) {
      // 我们可以看这一行代码
        result = executeForMany(sqlSession, args);
      } else if (method.returnsMap()) {
        result = executeForMap(sqlSession, args);
      } else if (method.returnsCursor()) {
        result = executeForCursor(sqlSession, args);
      } else {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = sqlSession.selectOne(command.getName(), param);
      }
    } else if (SqlCommandType.FLUSH == command.getType()) {
        result = sqlSession.flushStatements();
    } else {
      throw new BindingException("Unknown execution method for: " + command.getName());
    }
    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
      throw new BindingException("Mapper method '" + command.getName() 
          + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
    }
    return result;
  }
  ```

  上面都是样板代码，我们看其中一个方法
```java
  private <E> Object executeForMany(SqlSession sqlSession, Object[] args) {
    List<E> result;
    Object param = method.convertArgsToSqlCommandParam(args);
    if (method.hasRowBounds()) {
      RowBounds rowBounds = method.extractRowBounds(args);
      result = sqlSession.<E>selectList(command.getName(), param, rowBounds);
    } else {
    // SqlSession已经开始真正的执行了
      result = sqlSession.<E>selectList(command.getName(), param);
    }
    // issue #510 Collections & arrays support
    if (!method.getReturnType().isAssignableFrom(result.getClass())) {
      if (method.getReturnType().isArray()) {
        return convertToArray(result);
      } else {
        return convertToDeclaredCollection(sqlSession.getConfiguration(), result);
      }
    }
    return result;
  }
  ```
到这里，我们知道了映射器mapper就是一个JDK动态代理对象，进入到了MapperMethod的eexecute方法，他经过简单的判定就进入到了SqlSession的insert, delete,update, select等方法，这些方法如何执行、是我们需要搞清楚的，
### mapper的四大组件
* Executor 执行器，来调度StatementHandle， ParameterHandle，resultHandle，来执行相应的SQL
* StatementHandle是核心，使用数据库的Statement来操作
* parameterHandle SQL参数处理
* ResultHandle 对结果集进行封装处理

。。。。。   
这里的四大组件是很重要的，Mybatis的插件基本都是从这里起步的。
关于statementHandle的执行，留着下一篇博客在讲解把。马上开始实习了，也没时间学搞这些东西了。