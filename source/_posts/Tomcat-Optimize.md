---
title: Tomcat Optimize
date: 2017-11-23 17:30:32
categories:
	- java
tags:
	- java
	- Tomcat
---
摘要：Tomcat的几种优化方法
<!-- More -->
正文：
tomcat默认参数是为开发环境制定，而非适合生产环境，尤其是内存和线程的配置，默认都很低，容易成为性能瓶颈。

 
tomcat内存优化

linux修改TOMCAT_HOME/bin/catalina.sh，在前面加入
``` bash
	JAVA_OPTS="-XX:PermSize=64M -XX:MaxPermSize=128m -Xms512m -Xmx1024m -Duser.timezone=Asia/Shanghai"
```

windows修改TOMCAT_HOME/bin/catalina.bat，在前面加入
``` bash
set JAVA_OPTS=-XX:PermSize=64M -XX:MaxPermSize=128m -Xms512m -Xmx1024m
```
最大堆内存是1024m，对于现在的硬件还是偏低，实施时，还是按照机器具体硬件配置优化。

 
tomcat 线程优化
``` bash
<Connector port="80" protocol="HTTP/1.1" maxThreads="600" minSpareThreads="100" maxSpareThreads="500" acceptCount="700"
connectionTimeout="20000" redirectPort="8443" />
```
maxThreads="600"       ///最大线程数
minSpareThreads="100"///初始化时创建的线程数
maxSpareThreads="500"///一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程。
acceptCount="700"//指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理

 

这里是http connector的优化，如果使用apache和tomcat做集群的负载均衡，并且使用ajp协议做apache和tomcat的协议转发，那么还需要优化ajp connector。
``` bash
<Connector port="8009" protocol="AJP/1.3" maxThreads="600" minSpareThreads="100" maxSpareThreads="500" acceptCount="700"
connectionTimeout="20000" redirectPort="8443" />
```
 

由于tomcat有多个connector，所以tomcat线程的配置，又支持多个connector共享一个线程池。

首先。打开/conf/server.xml，增加

<Executor name="tomcatThreadPool" namePrefix="catalina-exec-" maxThreads="500" minSpareThreads="20" maxIdleTime="60000" />

最大线程500（一般服务器足以），最小空闲线程数20，线程最大空闲时间60秒。

 

然后，修改<Connector ...>节点，增加executor属性，executor设置为线程池的名字：
``` bash
	<Connector executor="tomcatThreadPool" port="80" protocol="HTTP/1.1"  connectionTimeout="60000" keepAliveTimeout="15000" maxKeepAliveRequests="1"  redirectPort="443" />
```
可以多个connector公用1个线程池，所以ajp connector也同样可以设置使用tomcatThreadPool线程池。

 
禁用DNS查询


当web应用程序向要记录客户端的信息时，它也会记录客户端的IP地址或者通过域名服务器查找机器名 转换为IP地址。

DNS查询需要占用网络，并且包括可能从很多很远的服务器或者不起作用的服务器上去获取对应的IP的过程，这样会消耗一定的时间。

修改server.xml文件中的Connector元素，修改属性enableLookups参数值: enableLookups="false"

如果为true，则可以通过调用request.getRemoteHost()进行DNS查询来得到远程客户端的实际主机名，若为false则不进行DNS查询，而是返回其ip地址

 

 
设置session过期时间

conf\web.xml中通过参数指定：
``` bash
	<session-config>   
	<session-timeout>180</session-timeout>     
	</session-config> 
 ```
单位为分钟。


