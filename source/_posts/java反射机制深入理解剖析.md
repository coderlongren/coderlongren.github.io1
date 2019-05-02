---
title: java反射机制深入理解分析
date: 2018-3-3 14:18:40
categories:
	- JDK
tags:
	- JDK
	- 反射
---

## 反射的简单介绍
主要是指程序可以访问，检测和修改它本身状态或行为的一种能力，并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义。

一个类有多个组成部分，例如：成员变量、方法、构造方法等，反射就是加载类,并解剖出类的各个组成部分。
主要功能:  
1. 在运行时判断一个对象所属于的类
2. 运行时构造任意一个类的对象
3. 在运行时判断任意一个类所具有的成员变量和方法；
4. 在运行时调用任意一个对象的方法；
5. 生成动态代理

## 反射以及作用
![](https://img.w3cschool.cn/attachments/day_161205/201612051945255425.png)
java反射机制中有哪些类：主要有五个主要的类 
```java
java.lang.Class;                
java.lang.reflect.Constructor;
java.lang.reflect.Field;        
java.lang.reflect.Method;
java.lang.reflect.Modifier;
```
*通过一个对象获得完整的包名和类名*
```java
public class TestReflect {
	public static void main(String[] args) {
		TestReflect testReflect = new TestReflect();
        System.out.println(testReflect.getClass().getName());
        // 反射.TestReflect
	}
}
```
实例化Class类对象
```java
public static void main(String[] args) throws ClassNotFoundException {
		// TODO Auto-generated method stub
		Class<?> class1 = null;
        Class<?> class2 = null;
        Class<?> class3 = null;
        // 一般采用这种形式
        class1 = Class.forName("反射.TestReflect");
        class2 = new TestReflect().getClass();
        class3 = TestReflect.class;
        System.out.println("类名称   " + class1.getName());
        System.out.println("类名称   " + class2.getName());
        Systsem.out.println("类名称   " + class3.getName());
        /** 
         *  类名称   反射.TestReflect
			类名称   反射.TestReflect
			类名称   反射.TestReflect
         */
	
	}
```
获取一个对象的父类与实现的接口
```java

public class TestReflect1 implements Cloneable, Serializable{
	private static final long seriaVersionUID = -2862585049955236662L;
	public static void main(String[] args) throws ClassNotFoundException {
		// TODO Auto-generated method stub
		Class<?> clazz = Class.forName("反射.TestReflect1");
		
		// 获取父类
		Class<?> parentClass = clazz.getSuperclass();
		
		System.out.println("class对象的父类是:" + parentClass.getName());
		// clazz的父类为： java.lang.Object
        // 获取所有的接口
		
		Class<?> intes[] = clazz.getInterfaces();
		System.out.println("clazz实现的接口有");
		for (int i = 0; i <intes.length; i++) {
			System.out.println(i++ + ":" + intes[i].getName());
		}
		/**
		 * class对象的父类是:java.lang.Object
			clazz实现的接口有 0:java.io.Serializable
		 */

	}

}
```

通过反射机制实例化一个类的对象
```java
package 反射;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

/**
* @author 作者 : coderlong
* @version 创建时间：2018年3月3日 下午9:27:11
* 类说明: 
*/
public class TestReflect {

	public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException {
		// TODO Auto-generated method stub
		Class<?> clazz1 = null;
		clazz1 = Class.forName("反射.User");
		User user = (User)clazz1.newInstance();
		user.setAge(20);
        user.setName("林黛玉");
        System.out.println(user);
        // result :  User [age=20, name=林黛玉]
        
        // 第二种方法 获取构造方法
        Constructor<?> cons[] = clazz1.getConstructors();
        // 看一下 每个构造方法需要的方法
        for (int i = 0; i < cons.length; i++) {
        	Class<?> clazzs[] = cons[i].getParameterTypes();
        	System.out.print("cons[" + i + "] (");
            for (int j = 0; j < clazzs.length; j++) {
                if (j == clazzs.length - 1)
                    System.out.print(clazzs[j].getName());
                else
                    System.out.print(clazzs[j].getName() + ",");
            }
            System.out.println(")");
        }
        // 结果
        //cons[0] (int,java.lang.String)
//        cons[1] (java.lang.String)
//        cons[2] ()
        
        User user2 = (User)cons[0].newInstance(20,"林黛玉");
        
        System.out.println(user2);
	}

}

```
## *获取某个类的全部属性* 可以使用clazz.getDeclaredFields()  
## *获得一个类的所有public属性，和父类所有方法 * clazz.getFields()

```java
public class TestReflect2 extends TestReflect implements Serializable,Cloneable{
	private static final long serialVersionUID = -2862585049955236662L;
	public static void main(String[] args) throws ClassNotFoundException{
		// TODO Auto-generated method stub
		Class<?> clazz = Class.forName("反射.TestReflect2");
		System.out.println("本类属性");
		// 取得自己累得全部属性
		Field[] fields = clazz.getDeclaredFields();
		
		for (int i = 0; i < fields.length; i++) {
			// 获得属性的修饰符
			int mo = fields[i].getModifiers();
			String priv = Modifier.toString(mo);
			System.out.println(priv);
			// 获得属性的类型
			Class<?> type = fields[i].getType();
			System.out.println(priv + " " + type.getName() + " " + fields[i].getName());
		}
		
		// 取得接口的实现 或者父类的属性
		System.out.println("接口的实现 或者父类的属性");
		Field[] fields2 = clazz.getFields();
		
		for (int i = 0; i < fields2.length; i++) {
			// 权限修饰符
			int mo = fields2[i].getModifiers();
			String priv = Modifier.toString(mo);
			// 属性的类型
			Class<?> type = fields2[i].getType();
			System.out.println(priv + " " + type.getName() + " " + fields2[i].getName());
		}
	}

}

```

* 获取某个类的全部方法 *

```java
public class testReflect3 {

	public static void main(String[] args) throws Exception {
		// TODO Auto-generated method stub
		Class<?> clazz = Class.forName("反射.testReflect3");
		
		Field[] fields = clazz.getDeclaredFields();
		Method[] methods = clazz.getDeclaredMethods();
		for (int i = 0; i < methods.length; i++) {
			// 返回值
			Class<?> returnType = methods[i].getReturnType();
			// 获得方法参数
			Class<?>[] para = methods[i].getParameterTypes();
			int mo = methods[i].getModifiers();
			// 修饰符
			String priv = Modifier.toString(mo);
			System.out.print(priv + " ");
			System.out.print(returnType.getName() + " ");
			System.out.print(methods[i].getName() + " ");
			System.out.print("(");
			for (int j = 0; j < para.length; j++) {
				System.out.print(para[i].getName() + " arg " + j);
				if (j < methods.length - 1) {
					System.out.print(",");
				}
			}
			Class<?>[] excel = methods[i].getExceptionTypes();
			if (excel.length > 0) {
				for (int k = 0; k < excel.length; k++) {
					System.out.print(excel[k].getName() + " ");
					if (k < excel.length - 1) {
						System.out.print(",");
					}
				}
				
			}
			else {
				System.out.print(")");
			}
			System.out.print(")");
		}
	}
	
	// result:
	// public static void main ([Ljava.lang.String; arg 0java.lang.Exception )

}

```
* 通过反射机制调用某个类的方法 *
```java
public class TestReflect4 {

	public static void main(String[] args) throws Exception {
		Class<?> clazz = Class.forName("反射.TestReflect4");
		Method method = clazz.getMethod("reflect1");
		method.invoke(clazz.newInstance());
		
		// 带参数的方法如何调用
		method = clazz.getMethod("reflect2", int.class, String.class);
		method.invoke(clazz. newInstance(),20,"wangyake");
		
	}
	public void reflect1() {
        System.out.println("Java 反射机制 - 调用某个类的方法shibushi1.");
    }
    public void reflect2(int age, String name) {
        System.out.println("Java 反射机制 - 调用某个类的方法2.");
        System.out.println("age -> " + age + ". name -> " + name);
    }
}
```

* 通过反射机制操作某个类的属性 *
```java
public class TestReflect5 {
	private String name  = "coderlong";
	public static void main(String[] args) throws Exception {
		// TODO Auto-generated method stub
		Class<?> clazz = Class.forName("反射.TestReflect5");
		// 反射对对象属性赋值
		Field field = clazz.getDeclaredField("name");
		Object object = clazz.newInstance();
		System.out.println(field.get(object));
		field.setAccessible(true);
		field.set(object,"yake");
		System.out.println(field.get(object));
	}

}
```
* 反射机制的动态代理 *

```
// 获取类加载器的方法
TestReflect testReflect = new TestReflect();
        System.out.println("类加载器  " + testReflect.getClass().getClassLoader().getClass().getName());
```

在java中有三种类类加载器。

*. Bootstrap ClassLoader 此加载器采用c++编写，一般开发中很少见。
*. Extension ClassLoaderyong来进行扩展类的加载，一般对应的是jrelibext目录中的类
 
*. AppClassLoader 加载classpath指定的类，是最常用的加载器。同时也是java中默认的加载器。
 
如果想要完成动态代理，首先需要定义一个InvocationHandler接口的子类，已完成代理的具体操作

## 反射的应用
在泛型为Integer的ArrayList中存放一个String类型的对象

```java
ArrayList<Integer> list = new ArrayList<>();
		Method method = list.getClass().getMethod("add", Object.class);
		method.invoke(list, "可以添加String？");
		System.out.println(!list.isEmpty());
```

反射修改数组元素
```java
public static void main(String[] args) throws NoSuchMethodException, SecurityException, IllegalAccessException, IllegalArgumentException, InvocationTargetException {
		// TODO Auto-generated method stub
		ArrayList<Integer> list = new ArrayList<>();
		Method method = list.getClass().getMethod("add", Object.class);
		method.invoke(list, "可以添加String？");
		System.out.println(!list.isEmpty());
		
		int[] temp = {1,2,3,4,5};
		Class<?> clazz = temp.getClass().getComponentType();
		System.out.println("数组类型" + clazz.getName());
		System.out.println("长度"  + Array.getLength(temp));
		System.out.println("数组第一个元素" + Array.get(temp,0));
		Array.set(temp, 0, 100);
		System.out.println("修改后第一个元素" + Array.get(temp, 0));
	}
```


优点： 
1. 能够运行时动态获取类的实例，大大提高系统的灵活性和扩展性。 
2. 与Java动态编译相结合，可以实现无比强大的功能 

缺点： 
1. 使用反射的性能较低 
2. 使用反射相对来说不安全 
3. 破坏了类的封装性，可以通过反射获取这个类的私有方法和属性 