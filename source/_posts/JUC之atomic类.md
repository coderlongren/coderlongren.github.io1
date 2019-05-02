---
title:  java.util.concurrent.atomic
date: 2018-2-5 20:18:40
categories:
	- JUC
tags:
	- JUC
	- 并发处理
	- 源码剖析
---
Aomic数据类型有四种类型：AomicBoolean, AomicInteger, AomicLong, 和AomicReferrence(针对Object的)以及它们的数组类型，
还有一个特殊的AomicStampedReferrence,它不是AomicReferrence的子类，而是利用AomicReferrence实现的一个储存引用和Integer组的扩展类  
所有原子操作都是依赖于sun.misc.Unsafe这个类，这个类底层是由C++实现的，利用指针来实现数据操作
## 关于CAS
一种无锁机制，比较并交换, 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。无 论哪种情况，它都会在 CAS 指令之前返回该位置的值。CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可”
好处：操作系统级别的支持，效率更高，无锁机制，降低线程的等待，实际上是把这个任务丢给了操作系统来做。
 
这个理论是整个java.util.concurrent包的基础。

### ATtomicXX的四个数值类型
1. value成员都是volatile
2. 基本方法get/set
3. compareAndSet
    weakCompareAndSet,
    lazySet: 使用Unsafe按照顺序更新参考Unsafe的C++实现）
    getAndSet：取当前值，使用当前值和准备更新的值做CAS
4. 对于Long和Integer
getAndIncrement/incrementAndGet
getAndDecrement/decrementAndGet
getAndAdd/addAndGet
三组方法都和getAndSet，取当前值，加减之得到准备更新的值，再做CAS，/左右的区别在于返回的是当前值还是更新值。