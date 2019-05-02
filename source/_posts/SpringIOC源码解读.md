---
title:  Spring IOC原理学习
date: 2018-4-3 21:18:40
categories:
	- JDK
tags:
	- JDK
	- Spring
---

## IOC容器的初始化
IOC容器的初始化包括BeanDefinition的Resource定位，载入，和注册三个基本的过程。
我们可以来看一下最基本的ApplicationContext系列容器，Web项目中经常遇到的XMLWebApplicationContext也属于这个继承体系，
### XMLBeanFactory的源码
```java
public class XmlBeanFactory extends DefaultListableBeanFactory{


     private final XmlBeanDefinitionReader reader; 
 

     public XmlBeanFactory(Resource resource)throws BeansException{
         this(resource, null);
     }
     

     public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory)
          throws BeansException{
         super(parentBeanFactory);
         this.reader = new XmlBeanDefinitionReader(this);
         this.reader.loadBeanDefinitions(resource);
    }
 }
 ```
 ### 创建过程
 ```java
 //根据Xml配置文件创建Resource资源对象，该对象中包含了BeanDefinition的信息
 ClassPathResource resource =new ClassPathResource("application-context.xml");
//创建DefaultListableBeanFactory
 DefaultListableBeanFactory factory =new DefaultListableBeanFactory();
//创建XmlBeanDefinitionReader读取器，用于载入BeanDefinition。之所以需要BeanFactory作为参数，是因为会将读取的信息回调配置给factory
 XmlBeanDefinitionReader reader =new XmlBeanDefinitionReader(factory);
//XmlBeanDefinitionReader执行载入BeanDefinition的方法，最后会完成Bean的载入和注册。完成后Bean就成功的放置到IOC容器当中，以后我们就可以从中取得Bean来使用
 reader.loadBeanDefinitions(resource);
 ```

 ## FileSystemApplicationContext的加载过程
 