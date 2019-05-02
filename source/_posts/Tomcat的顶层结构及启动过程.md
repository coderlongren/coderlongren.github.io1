---
title: Tomcat的顶层结构及启动过程
date: 2017-10-21 21:44:44
categories:
tags:
	- java
	- Tomcat
---
摘要：Tomcat的启动过程
<!-- More -->
正文：
![](/images/tomcat.jpg)
# Tomcat的顶层结构
---

Tomcat的最顶层的容器叫做servlet，代表整个服务器，Servlet中包含至少一个Service，用于具体提供务服务
。Service主要包含Connection Container 。 
从Tomcat启动调用栈可知，Bootstrap类的main方法为整个Tomcat的入口，在init初始化Bootstrap类的时候为设置Catalina的工作路径也就是Catalina_HOME信息、Catalina.base信息，在initClassLoaders方法中初始化类加载器，然后通过反射初始化org.apache.catalina.startup.Catalina作为catalina守护进程 
## 附上启动过程图： 
![](/images/tomcat1.png)
Tomcat中使用了观察者模式对Tomcat的生命周期进行了管理
---

## Lifecycle接口
Tomcat通过catalina.Lifecycle接口统一管理生命周期所有的生命周期组件 都要实现这个接口 这个接口一共做了四件事情
	1 设置了13个string类型常量 用于区分足间发出的lifecycle事件时候的状态
	2 定义了三个管理 监听器的方法 add find remove 来增加 查找 移除lifecycle类型的监听器
	3 定义了四个生命周期的方法 Init start stop destroy 用于执行生命周期的各个阶段的操作 
	4 获取状态的两个方法 getstate getstatename

## LifecycleBase 


	
