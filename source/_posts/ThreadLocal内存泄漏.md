---
title: ThreadLocal内存泄露问题的分析
date: 2018-3-8 23:18:40
categories:
	- JDK
tags:
	- JUC
	- JDK 
---

##  深入分析 ThreadLocal 内存泄漏问题

原文出处 http://www.importnew.com/22039.html
ThreadLocal 的作用是提供线程内的局部变量，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度。但是如果滥用ThreadLocal，就可能会导致内存泄漏。下面，我们将围绕三个方面来分析ThreadLocal 内存泄漏的问题
* ThreadLocal实现原理
* ThreadLocal为什内存泄漏
* ThreadLocal最佳实践

其实我也只是在书上看过ThreadLocal在队Session存储，数据库连接等地方有较大作用，但是为什么会内存泄漏，还是看大神的分析吧。

# ThreadLocal实现原理

![JDP](http://incdn1.b0.upaiyun.com/2016/10/2fdbd552107780c5ae5f98126b38d5a4.png)

ThreadLocal的实现是这样的：每个Thread 维护一个 ThreadLocalMap 映射表，这个映射表的 key 是 ThreadLocal实例本身，value 是真正需要存储的 Object。  

也就是说 ThreadLocal 本身并不存储值，它只是作为一个 key 来让线程从 ThreadLocalMap 获取 value。值得注意的是图中的虚线，表示 ThreadLocalMap 是使用 ThreadLocal 的弱引用作为 Key 的，弱引用的对象在 GC 时会被回收。

## 那么 ThreadLocal为什么内存泄漏呢？
ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用来引用它，那么系统 GC 的时候，这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry的value，* 如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value * 永远无法回收，造成内存泄漏。
其实，ThreadLocalMap的设计中已经考虑到这种情况，也加上了一些防护措施：在ThreadLocal的get(),set(),remove()的时候都会清除线程ThreadLocalMap里所有key为null的value。

但是这并不能保证永远不会内存泄漏  
1. 使用static的ThreadLocal，延长了ThreadLocal的生命周期，可能导致的内存泄漏
2. 分配使用了ThreadLocal又不再调用get(),set(),remove()方法，那么就会导致内存泄漏。

## 弱引用 与 强引用

* 如果使用强引用的话
引用的ThreadLocal的对象被回收了，但是ThreadLocalMap还持有ThreadLocal的强引用，如果没有手动删除，ThreadLocal不会被回收，导致Entry内存泄漏。
* 弱引用的话
引用的ThreadLocal的对象被回收了，由于ThreadLocalMap持有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。value在下一次ThreadLocalMap调用set,get，remove的时候会被清除。

比较来看的话，还是弱引用较为安全一些，至少有GC的一层保障，
