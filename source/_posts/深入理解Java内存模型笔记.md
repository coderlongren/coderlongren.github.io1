---
title: 深入理解Java内存模型
date: 2018-3-5 14:18:40
categories:
	- JMM
tags:
	- 代码重构
	- JDK
	- JMM
---

本文是我整理的程晓明《深入理解Java内存模型》读书笔记  
主要包括了三部分的知识。 
1. 基本概念
	包括“并发、同步、主内存、本地内存、重排序、内存屏障、happens before规则、as-if-serial规则、数据依赖性、顺序一致性模型、JMM的含义和意义”。
2. 同步机制

该部分中就介绍了“同步”的3种方式：volatile、锁、final。对于每一种方式，从该方式的“特性”、“建立的happens before关系”、“对应的内存语义”、“实现方式”等几个方面进行了分析说明。实际上，JMM保证“如果程序正确同步，则执行结果与顺序一致性内存模型的结果相同”的机制；而这部分这是确保程序正确同步的机制。
3. JMM 总结

## 1. 基本概念
并发编程模型的分类  
线程之间的通信机制有两种：共享内存和消息传递。
Java的并发采用的是共享内存模型，
## 指令重排序
在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。重排序分三种类型：
 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
 内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。
 ## 锁的语义分析
 ![](https://res.infoq.com/articles/java-memory-model-5/zh/resources/11.png)
 看这张图 ，还不明白吗？ 
 ReentrantLock中，调用lock()方法获取锁；调用unlock()方法释放锁。
 ReentrantLock的实现依赖于java同步器框架AbstractQueuedSynchronizer（本文简称之为AQS）。而这个所谓的AQS使用一个整型的volatile变量，来维护同步状态，JUC框架中，大多数同步容器均使用的AQS模型  
这是ReentrantLock 加锁的一段源代码
 ```java
 protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();   //获取锁的开始，首先读volatile变量state
    if (c == 0) {
        if (isFirst(current) &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)  
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
具体加锁的操作就是：   
ReentrantLock : lock()  
公平锁: lock()  
AQS: acquire(int arg)  
ReentrantLock: tryAcquire(int acquires)  
`tryAcquire(int acquires)  ` 才是真正加锁的步骤  

---
unlock()方法调用如下 ;
reentrantLock : unlock()  
AQS: release(int arg)  
Sync : tryRelease(int releases)  
```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);           //释放锁的最后，写volatile变量state
    return free;
}
```
在释放锁的最后写volatile变量state。
根据volatile的happens-before规则，释放锁的线程在写volatile变量之前可见的共享变量，在获取锁的线程读取同一个volatile变量后将立即变的对获取锁的线程可见。

## 公平锁如此，那么非公平的锁怎么样？ 
只是他们获取锁的过程不一样 

    ReentrantLock : lock()
    NonfairSync : lock()
    AbstractQueuedSynchronizer : compareAndSetState(int expect, int update)

以原子变量的方式，来更新states  ,也就是还是使用CAS算法。
	public final native boolean compareAndSwapInt(Object o, long offset,
                                              int expected,
                                              int x);

