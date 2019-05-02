---
title: AQS阻塞队列
date: 2018-5-17 12:50:17
categories:
	- concurrent
tags:
	- Java
	- concurrent
	- JDK源码
---

摘要: 并发编程 - AQS同步器
<!--More-->

AQS，AbstractQueuedSynchronizer，即队列同步器。它是构建锁或者其他同步组件的基础框架（如ReentrantLock、ReentrantReadWriteLock、Semaphore等），JUC并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。它是JUC并发包中的核心基础组件。
AQS解决了子啊实现同步器时涉及当的大量细节问题，例如获取同步状态、FIFO同步队列。基于AQS来构建同步器可以带来很多好处。它不仅能够极大地减少实现工作，而且也不必处理在多个位置上发生的竞争问题。AQS的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态。
AQS的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态。

AQS使用一个int类型的成员变量state来表示同步状态，当state>0时表示已经获取了锁，当state = 0时表示释放了锁。它提供了三个方法（getState()、setState(int newState)、compareAndSetState(int expect,int update)）来对同步状态state进行操作，当然AQS可以确保对state的操作是安全的。

AQS通过内置的FIFO同步队列来完成资源获取线程的排队工作，如果当前线程获取同步状态失败（锁）时，AQS则会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，则会把节点中的线程唤醒，使其再次尝试获取同步状态。
AQS的一些主要方法：  

1. getState()：返回同步状态的当前值；
2. setState(int newState)：设置当前同步状态；
3. compareAndSetState(int expect, int update)：使用CAS设置当前状态，该方法能够保证状态设置的原子性；
4. tryAcquire(int arg)：独占式获取同步状态，获取同步状态成功后，其他线程需要等待该线程释放同步状态才能获取同步状态；
5. tryRelease(int arg)：独占式释放同步状态；
6. tryAcquireShared(int arg)：共享式获取同步状态，返回值大于等于0则表示获取成功，否则获取失败；
7. tryReleaseShared(int arg)：共享式释放同步状态；
8. isHeldExclusively()：当前同步器是否在独占式模式下被线程占用，一般该方法表示是否被当前线程所独占；
9. acquire(int arg)：独占式获取同步状态，如果当前线程获取同步状态成功，则由该方法返回，否则，将会进入同步队列等待，该方法将会调用可重写的tryAcquire(int arg)方法；
10. acquireInterruptibly(int arg)：与acquire(int arg)相同，但是该方法响应中断，当前线程为获取到同步状态而进入到同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException异常并返回；
11. tryAcquireNanos(int arg,long nanos)：超时获取同步状态，如果当前线程在nanos时间内没有获取到同步状态，那么将会返回false，已经获取则返回true；
12. acquireShared(int arg)：共享式获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，与独占式的主要区别是在同一时刻可以有多个线程获取到同步状态；
13. acquireSharedInterruptibly(int arg)：共享式获取同步状态，响应中断；
14. tryAcquireSharedNanos(int arg, long nanosTimeout)：共享式获取同步状态，增加超时限制；
15. release(int arg)：独占式释放同步状态，该方法会在释放同步状态之后，将同步队列中第一个节点包含的线程唤醒；
16. releaseShared(int arg)：共享式释放同步状态；

## CLH阻塞队列
CLH同步队列是一个FIFO双向队列，AQS依赖它来完成同步状态的管理，当前线程如果获取同步状态失败时，AQS则会将当前线程已经等待状态等信息构造成一个节点（Node）并将其加入到CLH同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点唤醒（公平锁），使其再次尝试获取同步状态。
在CLH同步队列中，一个节点表示一个线程，它保存着线程的引用（thread）、状态（waitStatus）、前驱节点（prev）、后继节点（next），定义：  
AQS 内部类 Node的源码
```java
static final class Node {
    /** 共享 */
    static final Node SHARED = new Node();

    /** 独占 */
    static final Node EXCLUSIVE = null;

    /**
     * 因为超时或者中断，节点会被设置为取消状态，被取消的节点时不会参与到竞争中的，他会一直保持取消状态不会转变为其他状态；
     */
    static final int CANCELLED =  1;

    /**
     * 后继节点的线程处于等待状态，而当前节点的线程如果释放了同步状态或者被取消，将会通知后继节点，使后继节点的线程得以运行
     */
    static final int SIGNAL    = -1;

    /**
     * 节点在等待队列中，节点线程等待在Condition上，当其他线程对Condition调用了signal()后，改节点将会从等待队列中转移到同步队列中，加入到同步状态的获取中
     */
    static final int CONDITION = -2;

    /**
     * 表示下一次共享式同步状态获取将会无条件地传播下去
     */
    static final int PROPAGATE = -3;

    /** 等待状态 */
    volatile int waitStatus;

    /** 前驱节点 */
    volatile Node prev;

    /** 后继节点 */
    volatile Node next;

    /** 获取同步状态的线程 */
    volatile Thread thread;

    Node nextWaiter;

    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {
    }

    Node(Thread thread, Node mode) {
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) {
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```
## CLH入队
### addWaiter()方法，增加等待线程
```java
 /**
     * Creates and enqueues node for current thread and given mode.
     *
     * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
     * @return the new node
     */
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            //CAS
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        // 多次尝试
        enq(node);
        return node;
    }

```
先通过快速尝试设置为节点，如果失败则通过enq(node)设置尾节点
```java
private Node enq(final Node node) {
        //多次尝试，直到成功为止
        for (;;) {
            Node t = tail;
            //tail不存在，设置为首节点
            if (t == null) {
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                //设置为尾节点
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```
上述两个方法都是使用`compareAndSetTail(t, node)`来死循环添加尾部节点，

## CLH出队
CLH同步队列遵循FIFO，首节点的线程释放同步状态后，将会唤醒它的后继节点（next），而后继节点将会在获取同步状态成功时将自己设置为首节点，这个过程非常简单，head执行该节点并断开原首节点的next和当前节点的prev即可，注意在这个过程是不需要使用CAS来保证的，因为只有一个线程能够成功获取到同步状态。
# 同步状态的获取与释放
AQS提供了大量的模板方法（被叫做模板模式）来实现同步，主要是分为三类：独占式获取和释放同步状态、共享式获取和释放同步状态、查询同步队列中的等待线程情况。自定义子类使用AQS提供的模板方法就可以实现自己的同步语义。
## 独占式
每次只有一个线程持有同步状态
#### 独占式同步状态的获取
对中断状态不敏感
```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
}
```
1. tryAcquire：去尝试获取锁，获取成功则设置锁状态并返回true，否则返回false。该方法自定义同步组件自己实现，该方法必须要保证线程安全的获取同步状态。
2. addWaiter：如果tryAcquire返回FALSE（获取同步状态失败），则调用该方法将当前线程加入到CLH同步队列尾部。
3. acquireQueued：当前线程会根据公平性原则来进行阻塞等待（自旋）,直到获取锁为止；并且返回当前线程在等待过程中有没有中断过。
4. selfInterrupt：产生一个中断。

acquireQueued方法为一个自旋的过程，也就是说当前线程进入同步队列后，就会进入一个自旋的过程，每个节点都会自省地观察，当条件满足，获取到同步状态后，就可以从这个自旋过程中退出，否则会一直执行下去
```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        //中断标志
        boolean interrupted = false;
        /*
         * 自旋过程，其实就是一个死循环而已
         */
        for (;;) {
            //当前线程的前驱节点
            final Node p = node.predecessor();
            //当前线程的前驱节点是头结点，且同步状态成功
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //获取失败，线程等待
            if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
### 独占式获取响应中断
```java
public final void acquireInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (!tryAcquire(arg))
            doAcquireInterruptibly(arg);
    }
```
如果当前线程被中断了，会立刻响应中断抛出异常InterruptedException。
doAcquireInterruptibly()
```java
private void doAcquireInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
### 独占同步状态释放
```java
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
先调用自定义同步器自定义的tryRelease(int arg)方法来释放同步状态，释放成功后，会调用unparkSuccessor(Node node)方法唤醒后继节点

`在AQS中维护着一个FIFO的同步队列，当线程获取同步状态失败后，则会加入到这个CLH同步队列的对尾并一直保持着自旋。在CLH同步队列中的线程在自旋时会判断其前驱节点是否为首节点，如果为首节点则不断尝试获取同步状态，获取成功则退出CLH同步队列。当线程执行完逻辑后，会释放同步状态，释放后会唤醒其后继节点。`
## 共享式
最主要区别在于同一时刻独占式只能有一个线程获取同步状态，而共享式在同一时刻可以有多个线程获取同步状态
### 共享获取状态
acquireShared(int arg)
```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        //获取失败，自旋获取同步状态
        doAcquireShared(arg);
}
```
调用tryAcquireShared(int arg)方法尝试获取同步状态，如果获取失败则调用doAcquireShared(int arg)自旋方式获取同步状态，共享式获取同步状态的标志是返回 >= 0 的值表示获取成功。
```java
 private void doAcquireShared(int arg) {
    /共享式节点
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            //前驱节点
            final Node p = node.predecessor();
            //如果其前驱节点，获取同步状态
            if (p == head) {
                //尝试获取同步
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
获得状态之后，完成自己的逻辑，就释放掉
```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```
# AQS阻塞与唤醒
在线程获取同步状态时如果获取失败，则加入CLH同步队列，通过通过自旋的方式不断获取同步状态，但是在自旋的过程中则需要判断当前线程是否需要阻塞，主要方法在acquireQueued()：
如果获取同步状态失败之后，不是立刻阻塞，而是检查前驱结点，是否需要阻塞
```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    //前驱节点
    int ws = pred.waitStatus;
    //状态为signal，表示当前线程处于等待状态，直接放回true
    if (ws == Node.SIGNAL)
        return true;
    //前驱节点状态 > 0 ，则为Cancelled,表明该节点已经超时或者被中断了，需要从同步队列中取消
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } 
    //前驱节点状态为Condition、propagate
    else {
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
 ```
 1. 前驱结点 Sinnal, 表明当前节点需要阻塞，
 2. 前驱结点 cancelled(ws>0) 则表明该线程的前驱节点已经等待超时或者被中断了，则需要从CLH队列中将该前驱节点删除掉，直到回溯到前驱节点状态 <= 0 ，返回false
 3. 前驱节点非SINNAL，非CANCELLED
```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
``` 
parkAndCheckInterrupt() 方法主要是把当前线程挂起
```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            //唤醒后继节点
            unparkSuccessor(h);
        return true;
    }
    return false;
}
private void unparkSuccessor(Node node) {
    //当前节点状态
    int ws = node.waitStatus;
    //当前状态 < 0 则设置为 0
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    //当前节点的后继节点
    Node s = node.next;
    //后继节点为null或者其状态 > 0 (超时或者被中断了)
    if (s == null || s.waitStatus > 0) {
        s = null;
        //从tail节点来找可用节点
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    //唤醒后继节点
    if (s != null)
        LockSupport.unpark(s.thread);
}

```
### LockSupport是用来创建锁和其他同步类的基本线程阻塞原语
好像LockSupport底层还是依赖UNSAFE native类来搞得，













