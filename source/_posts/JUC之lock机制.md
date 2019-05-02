---
title:  java.util.concurrent之锁机制
date: 2018-2-5 21:18:40
categories:
	- JUC
tags:
	- JUC
	- 并发处理
	- 源码剖析
---
`此博文过长，纯属自己写笔记用，慎入！`
### Condition借口
一个线程因为某个condition不满足被挂起，直到该Condition被满足了。
类似与Object的wait/
notify，因此Condition对象应该是被多线程共享的，需要使用锁保护其状态的一致性

```java
class BoundedBuffer {
     final Lock lock = new ReentrantLock();
     final Condition notFull     = lock.newCondition();
     final Condition notEmpty = lock.newCondition();

     final Object[] items = new Object[100];
     int putptr, takeptr, count;

     public void put(Object x) throws InterruptedException {
          lock.lock();
          try {
               while (count == items.length)
                    notFull.await();
               items[putptr] = x;
               if (++putptr == items.length) putptr = 0;
               ++count;
               notEmpty.signal();
          } finally {
               lock.unlock();
          }
     }         

     public Object take() throws InterruptedException {
          lock.lock();
          try {
               while (count == 0)
                    notEmpty.await();
               Object x = items[takeptr];
               if (++takeptr == items.length) takeptr = 0;
               --count;
               notFull.signal();
               return x;
          } finally {
               lock.unlock();
          }
     }
}
```

## Lock和ReadWriteLock
很多人一看到ReadWriteLock就觉得这是一个实现了Lock的实现类，然而这也是一个interface，不过实现了这个interface的类，是可以作为读写锁来使用的。的
两个接口，后者不是前者的子接口，通过以下ReadWriteLock的代码就可以看出两者的联系了：
```java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading.
     读锁
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *写锁
     * @return the lock used for writing.
     */
    Lock writeLock();
}
```
两个接口都有一个可重入(ReentrantLock, ReentrantReadWriteLock)的实现
## LockSupport工具类
这个类，我也不是很明白，是一个工具类，操作对象是线程，基于Unsafe类实现
基本操作park和unpark。park会把使得当前线程失效（没有提供操作其他线程的，其实是可以实现的），暂时挂起
## AbstractOwnableSynchronizer, AbstractQueuedSynchronizer, 
AbstractQueuedLongSynchronizer
并发包的JDK源码充满了大量的 包装类，和设计模式用到的类，所以就算看着中文的注释还是很难读懂，这三个类，就是典型的代表。
AbstractQueuedSynchronizer, AbstractQueuedLongSynchronizer是AbstractOwnableSynchronizer的子类，
AbstractOwnableSynchronizer只是实现了被线程独占这些功能的Synchronizer，并不包含如何管理实现多个线程的同步。包含了一个exclusiveOwnerThread，set/get方法。

AbstractQueuedSynchronizer利用Queue的方式来管理线程关于锁的使用和同步，相当于一个锁的管理者。
首先关注四个最核心的方法：
protected boolean tryAcquire(int arg)
protected boolean tryRelease(int arg)
protected int tryAcquireShared(int arg)
protected boolean tryReleaseShared(int arg)

前两个用于独占锁，后两者用于共享锁，这四个方法是由子类来实现的，即如何获取和释放锁AbstractQueuedSynchronizer是不参与的，默认实现是不支持，即抛出UnsupportedOperationException。
AbstractQueuedSynchronizer做什么呢？

当 前线程尝试获取锁的时候，AbstractQueuedSynchronizer会先调用tryAcquire或者tryAcquireShared来尝 试获取，如果得到false，那么把当前线程放到等待队列中去，然后再做进一步操作。我们来分析以下6种情况，前三种用于独占锁，后三者用于共享，独占锁 或者共享锁按照等待方式又分为三种：不可中断线程等待，可中断线程等待，尝试限时等待超时放弃。
这6种的方法都含有一个int类型的参数，这个是给上面的tryAcquire这种方法使用的，也就是说它一个自定义的参数，一般用来表示某个自定义的状态。

1. 独占锁，放入队列之后，直到成功获取，会忽略线程的中断
```
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }


```

## ReentrantLock
书上给的解释：  
`一个可重入的互斥锁定 Lock，它具有与使用 synchronized 方法和语句所访问的隐式监视器锁定相同的一些基本行为和语义，但功能更强大。ReentrantLock 将由最近成功获得锁定，并且还没有释放该锁定的线程所拥有。当锁定没有被另一个线程所拥有时，调用 lock 的线程将成功获取该锁定并返回。如果当前线程已经拥有该锁定，此方法将立即返回。可以使用 isHeldByCurrentThread() 和 getHoldCount() 方法来检查此情况是否发生。`

### 获取锁
```java
//非公平锁
ReentrantLock lock = new ReentrantLock();
lock.lock();
```
lock()
```java
public void lock() {
  sync.lock();
}
```
Sync为ReentrantLock里面的一个内部类，它继承AQS（AbstractQueuedSynchronizer），它有两个子类：公平锁FairSync和非公平锁NonfairSync。

大部分功能都是委托给sync来完成的
```java
final void lock() {
    //尝试获取锁
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        //获取失败，调用AQS的acquire(int arg)方法
        acquire(1);
}
```

尝试获取索，失败的话，就到AQS里面了。
```java
public final void acquire(int arg) {
      if (!tryAcquire(arg) &&
              acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
          selfInterrupt();
  }
```
## 释放索
```java
public void unlock() {
        sync.release(1);
  }
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

```
## 公平性
公平锁与非公平锁的区别在于获取锁的时候是否按照FIFO的顺序来。释放锁不存在公平性和非公平性
```java
protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
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

// 主要是判断当前线程是否位于CLH同步队列中的第一个。如果是则返回true，否则返回false。
public final boolean hasQueuedPredecessors() {
        Node t = tail;  //尾节点
        Node h = head;  //头节点
        Node s;

        //头节点 != 尾节点
        //同步队列第一个节点不为null
        //当前线程是同步队列第一个节点
        return h != t &&
                ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```
## 读写锁
读写锁维护着一对锁，一个读锁和一个写锁。通过分离读锁和写锁，使得并发性比一般的排他锁有了较大的提升：在同一时间可以允许多个读线程同时访问，但是在写线程访问时，所有读线程和写线程都会被阻塞。
*特性*  
1. 公平性：支持公平性和非公平性。
2. 重入性：支持重入。读写锁最多支持65535个递归写入锁和65535个递归读取锁。
3. 锁降级：遵循获取写锁、获取读锁在释放写锁的次序，写锁能够降级成为读锁
读写锁ReentrantReadWriteLock实现接口ReadWriteLock，该接口维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。

```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
public class ReentrantReadWriteLock
        implements ReadWriteLock, java.io.Serializable {
    private static final long serialVersionUID = -6992448646407690164L;
    /** Inner class providing readlock */
    private final ReentrantReadWriteLock.ReadLock readerLock;
    /** Inner class providing writelock */
    private final ReentrantReadWriteLock.WriteLock writerLock;
    /** Performs all synchronization mechanics */
    final Sync sync;

 abstract static class Sync extends AbstractQueuedSynchronizer {
 }
 public static class ReadLock implements Lock, java.io.Serializable {
 }
。。。。。。。
}
```
在ReentrantLock中使用一个int类型的state来表示同步状态，该值表示锁被一个线程重复获取的次数。但是读写锁ReentrantReadWriteLock内部维护着两个一对锁，需要用一个变量维护多种状态。所以读写锁采用“按位切割使用”的方式来维护这个变量，将其切分为两部分，高16为表示读，低16为表示写。分割之后，读写锁是如何迅速确定读锁和写锁的状态呢？通过位运算。假如当前同步状态为S，那么写状态等于 S & 0x0000FFFF（将高16位全部抹去），读状态等于S >>> 16(
无符号补0右移16位)。
###  写锁获取
```java
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    //当前锁个数
    int c = getState();
    //写锁
    int w = exclusiveCount(c);
    if (c != 0) {
        //c != 0 && w == 0 表示存在读锁
        //当前线程不是已经获取写锁的线程
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        //超出最大范围
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        setState(c + acquires);
        return true;
    }
    //是否需要阻塞
    if (writerShouldBlock() ||
            !compareAndSetState(c, c + acquires))
        return false;
    //设置获取锁的线程为当前线程
    setExclusiveOwnerThread(current);
    return true;
}
```
判断读锁是否存在。因为要确保写锁的操作对读锁是可见的，如果在存在读锁的情况下允许获取写锁，那么那些已经获取读锁的其他线程可能就无法感知当前写线程的操作。因此只有等读锁完全释放后，写锁才能够被当前线程所获取，一旦写锁获取了，所有其他读、写线程均会被阻塞。
### 写锁的释放
```java
public void unlock() {
    sync.release(1);
}

public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```
### 读锁的获取 与释放
`这个有点复杂，涉及到锁的降级，部分代码我现在看不懂以后再说吧。。。`
锁降级就意味着写锁是可以降级为读锁的，但是需要遵循先获取写锁、获取读锁在释放写锁的次序。

# Condition 在没有Lock之前，我们使用synchronized来控制同步，配合Object的wait()、notify()系列方法可以实现等待/通知模式。在Java SE5后，Java提供了Lock接口，相对于Synchronized而言，Lock提供了条件Condition，对线程的等待、唤醒操作更加详细和灵活。
Condition提供了一系列的方法来对阻塞和唤醒线程：
1. await() ：造成当前线程在接到信号或被中断之前一直处于等待状态。
2. await(long time, TimeUnit unit) ：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
3. awaitNanos(long nanosTimeout) ：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。返回值表示剩余时间，如果在nanosTimesout之前唤醒，那么返回值 = nanosTimeout - 消耗时间，如果返回值 <= 0 ,则可以认定它已经超时了。
4. awaitUninterruptibly()：造成当前线程在接到信号之前一直处于等待状态。
5. awaitUntil(Date deadline) ：造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。如果没有到指定时间就被通知，则返回true，否则表示到了指定时间，返回返回false。
6. signal()：唤醒一个等待线程。该线程从等待方法返回前必须获得与Condition相关的锁。
7. signalAll()：唤醒所有等待线程。能够从等待方法返回的线程必须获得与Condition相关的锁。

Condition 需要使用Lock的newCondition()方法来创建
Condition为一个接口，其下仅有一个实现类ConditionObject，由于Condition的操作需要获取相关的锁，而AQS则是同步锁的实现基础，所以ConditionObject则定义为AQS的内部类。定义如下：
```java
public class ConditionObject implements Condition, java.io.Serializable {
    private static final long serialVersionUID = 1173984872572414699L;

    //头节点
    private transient Node firstWaiter;
    //尾节点
    private transient Node lastWaiter;

    public ConditionObject() {
    }
}
```
具体的过程就是：  
一个线程获得锁后，通过调用Condition()的Await()会将当前线程先加入到条件队列中，然后释放锁，最后通过isOnSyncQueue(Node node)方法不断自检看节点是否已经在CLH同步队列了，如果是则尝试获取锁，否则一直挂起。当线程调用signal()方法后，程序首先检查当前线程是否获取了锁，然后通过doSignal(Node first)方法唤醒CLH同步队列的首节点。被唤醒的线程，将从await()方法中的while循环中退出来，然后调用acquireQueued()方法竞争同步状态。
自我实现一个生产者消费者模型
```java
import java.util.LinkedList;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
* @author 作者 : coderlong
* @version 创建时间：2018年4月9日 下午10:30:51
* 类说明:  使用Condition实现的生产者消费者模式
*/
public class ConditionTest {
  private LinkedList<String> buffer;    //容器
    private int maxSize ;           //容器最大
    private Lock lock;
    private Condition fullCondition;
    private Condition notFullCondition;
    
  public ConditionTest( int maxSize) {
    super();
    this.buffer = new LinkedList<>();
    this.maxSize = maxSize;
    this.lock = new ReentrantLock();
    this.fullCondition = lock.newCondition();
    this.notFullCondition = lock.newCondition();
  }
  // 生产操作
  public void set(String string) throws InterruptedException {
    lock.lock();
    try {
      while (maxSize == buffer.size()){
        notFullCondition.await(); // 慢了
      } 
      System.out.println("set" + string);
      buffer.add(string);
      fullCondition.signal();
    }
    finally {
      lock.unlock();
    }
  }
  
  // 消费
  public String get() throws InterruptedException {
    String string;
    lock.lock();
    try {
      while (buffer.size() == 0) {
        fullCondition.await(); // 
      }
      string = buffer.poll();
      notFullCondition.signal();
    } finally {
      lock.unlock();
    }
    return string;
  }

  public static void main(String[] args) {
    // TODO Auto-generated method stub
    ConditionTest conditionTest = new ConditionTest(3);
    while (true) {
      Thread setter = new Thread(new Runnable() {
        
        @Override
        public void run() {
          // TODO Auto-generated method stub
          try {
            conditionTest.set("abc");
          } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
          }
        }
      });
      setter.start();
      Thread getter = new Thread(new Runnable() {
        
        @Override
        public void run() {
          // TODO Auto-generated method stub
          try {
            
            System.out.println(conditionTest.get());
            
          } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
          }
        }
      });
      getter.start();
    }
  }

}
```






