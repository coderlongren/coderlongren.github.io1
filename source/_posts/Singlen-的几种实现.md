---
title: Singlen 的几种实现
date: 2017-12-18 23:30:58
categories:
	- java
tags:
	- java
	- 设计模式
---
摘要：没有摘要
![](/images/huda2.jpeg)
<!-- more -->
正文：
# java设计模式之单例模式，
这里主要介绍三种：懒汉式单例、饿汉式单例、登记式单例。  
## 懒汉式单例
```
//懒汉式单例类.在第一次调用的时候实例化自己   
public class Singleton {  
	private Singleton() {}  
	private static Singleton single=null;  
	//静态工厂方法   
	public static Singleton getInstance() {  
		 if (single == null) {    
			 single = new Singleton();  
		 }    
		return single;  
	}  
}  
```
很明显，以上懒汉式单例的实现没有考虑线程安全问题，它是线程不安全的，  
并发环境下很可能出现多个Singleton实例，要实现线程安全，有以下三种方式，  
都是对getInstance这个方法改造，保证了懒汉式单例的线程安全。
### 在getInstance()方法上加上同步
```
public static synchronized Singleton getInstance() {  
		 if (single == null) {    
			 single = new Singleton();  
		 }    
		return single;  
}  
```
### 加双重锁锁定 
```
public static Singleton getInstance() {  
		if (singleton == null) {    
			synchronized (Singleton.class) {    
			   if (singleton == null) {    
				  singleton = new Singleton();   
			   }    
			}    
		}    
		return singleton;   
	}  
```
### 静态内部类实现  我更倾向这种方法的单例实现
这种方式能达到双检锁方式一样的功效，但实现更简单。对静态域使用延迟初始化，应使用这种方式而不是双检锁方式。  
这种方式只适用于静态域的情况，双检锁方式可在实例域需要延迟初始化时使用。  
这种方式同样利用了 classloder 机制来保证初始化 instance 时只有一个线程，它跟饿汉方式不同的是：  
饿汉方式只要 Singleton 类被装载了，那么 instance 就会被实例化（没有达到 lazy loading 效果），  
而这种方式是 * Singleton 类被装载了，instance 不一定被初始化。因为 SingletonHolder 类没有被主动使用， * 
只有显示通过调用 getInstance 方法时，才会显示装载 SingletonHolder 类，从而实例化 instance。想象一下，  
如果实例化 instance 很消耗资源，所以想让它延迟加载，另外一方面，又不希望在 Singleton 类加载时就实例化，  
因为不能确保 Singleton 类还可能在其他的地方被主动使用从而被加载，那么这个时候实例化 instance 显然是不合适的。  
这个时候，这种方式相比饿汉方式就显得很合理。
```
public class Singleton {    
	private static class LazyHolder {    
	   private static final Singleton INSTANCE = new Singleton();    
	}    
	private Singleton (){}    
	public static final Singleton getInstance() {    
	   return LazyHolder.INSTANCE;    
	}    
}      
```

## 饿汉式单例
```
public class Singleton {  
    private static Singleton instance = new Singleton();  
    private Singleton (){}  
    public static Singleton getInstance() {  
    return instance;  
    }  
} 
```
这种单例模式，并没有解决Lazy加载，所以使用的频率也不高