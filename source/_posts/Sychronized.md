---
title: Sychronized关键字解析
date: 2018-3-30 14:18:40
categories:
	- JDK
tags:
	- JDK
---

synchronized可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时它还可以保证共享变量的内存可见性 .
Java中每一个对象都可以作为锁，这是synchronized实现同步的基础：
1. 普通同步方法，锁是当前实例对象
2. 静态同步方法，锁是当前类的class对象
3. 同步方法块，锁是括号里面的对象
同步代码块是使用monitorenter和monitorexit指令实现的，依靠的是方法修饰符上的ACC_SYNCHRONIZED实现。
同步代码块：monitorenter指令插入到同步代码块的开始位置，monitorexit指令插入到同步代码块的结束位置，JVM需要保证每一个monitorenter都有一个monitorexit与之相对应。任何对象都有一个monitor与之相关联，当且一个monitor被持有之后，他将处于锁定状态。线程执行到monitorenter指令时，将会尝试获取对象所对应的monitor所有权，即尝试获取对象的锁；
同步方法：synchronized方法则会被翻译成普通的方法调用和返回指令如:invokevirtual、areturn指令，在JVM字节码层面并没有任何特别的指令来实现被synchronized修饰的方法，而是在Class文件的方法表中将该方法的access_flags字段中的synchronized标志位置1，表示该方法是同步方法并使用调用该方法的对象或该方法所属的Class在JVM的内部对象表示Klass做为锁对象。

## Java对象头，monitor
Hotspot虚拟机的对象头主要包括两部分数据：MarkWord（标记字段）、Klass Pointer（类型指针）。其中Klass Point是是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，MarkWord用于存储对象自身的运行时数据，它是实现轻量级锁和偏向锁的关键。
### Mark Word
Mark Word用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等等
!(Mark Word图片)[https://images0.cnblogs.com/blog/571766/201312/25200026-f206ec76fb2c4fc0823e5c8de705a8af.png]
### Monitor
我们可以把它理解为一个同步工具，所有的Java对象是天生的Monitor，每一个Java对象都有成为Monitor的潜质，每一个Java对象出来就带了一把看不见的锁，它叫做内部锁或者Monitor锁。
Monitor 是线程私有的数据结构，每一个线程都有一个可用monitor record列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个monitor关联（对象头的MarkWord中的LockWord指向monitor的起始地址）