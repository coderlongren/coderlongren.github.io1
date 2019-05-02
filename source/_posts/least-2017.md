---
title: least-2017
date: 2017-12-14 11:24:49
categories:
	- 生活
tags:
	- 代码
	- 个人感想
---
摘要： 六级考试之前的坑？ 不，是java反射

<!-- more -->
正文：
# 武汉的天气真是冷啊！毫无小桥流水的风度。
![](/images/leng.jpg)

---
## 什么事反射啊？ 电面试题
反射是一种间接操作目标对象的机制，在程序程序运行时获取或者设置对象自身的信息。   
只要给定类的名字，就可以通过反射获取类的所有信息，接着便能调用它的任何一个方法和属性。
## java的反射过程，
1. 获取类加载器：`ClassLoader loader=Thread.currentThread().getContextClassLoader();`//获取当前线程的上下文类加载器
2. 通过类加载器获取类  `Class clazz=loader.loadClass("com.taobao.reflect.car")`//通过对象的全称限定来获取对象。
3. 通过clazz获得构造函数：`Constructors cons=clazz.getDeclaraedConstructors(Class[]null);`//调用默认的构造函数
4. 然后通过构造函数构造对象：`Car car=(Car)cons.newInstance();`//获取类的默认构造函数对象并实例化对象。
5. 得到car对象，然后调用car的方法：`Method methd =car.getMethod("setName","String.class");`//method声明，并指向car的setName
得到setName方法。
6. `method.invoke(car,"红旗CA72");`//通过invoke方法，让car的method（就是setName方法）方法执行，参数为“红旗CA72”.  
### 优点：可以动态的创建对象和编译，最大限度发挥了java的灵活性。
### 缺点：对性能有影响。使用反射基本上一种解释操作，告诉JVM我们要做什么并且满足我们的要求，这类操作总是慢于直接执行java代码。
## 类的加载过程
JVM将类加载过程分为三个步骤：装载（Load），链接（Link）和初始化(Initialize)链接又分为三个步骤，如下图所示：
![](/images/load1.png)
1. 装载：查找并加载类的二进制数据；

2. 链接：

    验证：确保被加载类的正确性  

    准备：为类的静态变量分配内存，并将其初始化为默认值  

    解析：把类中的符号引用转换为直接引用  
3. 初始化：为类的静态变量赋予正确的初始值
## 类的初始化
类在什么时候才被初始化
1. 创建类的实例，也就是new一个对象
2. 反射（Class.forName("类的权限命名")）
3. 启动一个类的子类
4. 

## java的类加载器
Java虚拟机中系统默认的类加载器有三个：BootStrap,ExtClassLoader,AppClassLoader



