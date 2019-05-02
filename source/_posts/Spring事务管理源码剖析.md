---
title: Spring事务管理
date: 2018-3-28 21:18:40
categories:
	- Spring
tags:
	- Spring
---
## 事务是神马？？
事务就是用来解决类似问题的。事务是一系列的动作，它们综合在一起才是一个完整的工作单元，这些动作必须全部完成，如果有一个失败的话，那么事务就会回滚到最开始的状态，仿佛什么都没发生过一样。   
他的ACID特性就不说了
## Spring对事务支持的接口大概
![事务](http://img.blog.csdn.net/20160324011156424)
### PlatformTransactionManager事务管理器
Spring并不直接管理事务，而提供了多种事务管理器，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。 
Spring事务管理器的接口是org.springframework.transaction.PlatformTransactionManager，通过这个接口，Spring为各个平台如JDBC、Mybatis, Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。
```java
Public interface PlatformTransactionManager()...{  
    // 由TransactionDefinition得到TransactionStatus对象
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException; 
    // 提交
    Void commit(TransactionStatus status) throws TransactionException;  
    // 回滚
    Void rollback(TransactionStatus status) throws TransactionException;  
} 
```
我们只需要使用TransactionDefinition.getTransaction()
进而使用TransactionManager.getTransaction()来获取特性的持久化框架的API即可。Spring是不是很强大

### JDBC的话

	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	        <property name="dataSource" ref="dataSource" />
	</bean>

### Hibernate也是一样的
	<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
	        <property name="sessionFactory" ref="sessionFactory" />
	    </bean>


### 最基本的事务属性的定义
事务管理器接口PlatformTransactionManager通过getTransaction(TransactionDefinition definition)方法来得到事务，这个方法里面的参数是TransactionDefinition类，这个类就定义了一些基本的事务属性。  

`TransactionDefinition接口定义如下`
```java
public interface TransactionDefinition {
    int getPropagationBehavior(); // 返回事务的传播行为
    int getIsolationLevel(); // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getTimeout();  // 返回事务必须在多少秒内完成
    boolean isReadOnly(); // 事务是否只读，事务管理器能够根据这个返回值进行优化，确保事务是只读的
} 
```
