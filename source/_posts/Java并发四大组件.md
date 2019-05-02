---
title:  JUC的四大组件
date: 2018-4-3 21:18:40
categories:
	- JUC
tags:
	- JUC
	- 并发处理
	- 源码剖析
---
# CyclicBarrier
它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。   
提供了两个构造： 
```java
 public CyclicBarrier(int parties) {
        this(parties, null);
    }
 ``` 
1. CyclicBarrier(int parties)：创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。
```java
 public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }
 ```
2. CyclicBarrier(int parties, Runnable barrierAction) ：创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 barrier 的线程执行。

### await()
```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}

```
内部调用 dowait()
```java
private int dowait(boolean timed, long nanos)
            throws InterruptedException, BrokenBarrierException,
            TimeoutException {
        //获取锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            //分代
            final Generation g = generation;

            //当前generation“已损坏”，抛出BrokenBarrierException异常
            //抛出该异常一般都是某个线程在等待某个处于“断开”状态的CyclicBarrie
            if (g.broken)
                //当某个线程试图等待处于断开状态的 barrier 时，或者 barrier 进入断开状态而线程处于等待状态时，抛出该异常
                throw new BrokenBarrierException();

            //如果线程中断，终止CyclicBarrier
            if (Thread.interrupted()) {
                breakBarrier();
                throw new InterruptedException();
            }

            //进来一个线程 count - 1
            int index = --count;
            //count == 0 表示所有线程均已到位，触发Runnable任务
            if (index == 0) {  // tripped
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    //触发任务
                    if (command != null)
                        command.run();
                    ranAction = true;
                    //唤醒所有等待线程，并更新generation
                    nextGeneration();
                    return 0;
                } finally {
                    if (!ranAction)
                        breakBarrier();
                }
            }


            for (;;) {
                try {
                    //如果不是超时等待，则调用Condition.await()方法等待
                    if (!timed)
                        trip.await();
                    else if (nanos > 0L)
                        //超时等待，调用Condition.awaitNanos()方法等待
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // We're about to finish waiting even if we had not
                        // been interrupted, so this interrupt is deemed to
                        // "belong" to subsequent execution.
                        Thread.currentThread().interrupt();
                    }
                }

                if (g.broken)
                    throw new BrokenBarrierException();

                //generation已经更新，返回index
                if (g != generation)
                    return index;

                //“超时等待”，并且时间已到,终止CyclicBarrier，并抛出异常
                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            //释放锁
            lock.unlock();
        }
    }
```
这里的await() 主要逻辑就是,如果次线程不是最后到达的，就处于等待状态：  
1. 最后一个线程到达时候，index = 0;  
2. 超时；
3. 其他的某个线程中断当前线程
4. 其他的某个线程中断另一个等待的线程
5. 其他的某个线程在等待barrier超时
6. 其他的某个线程在此barrier调用reset()方法。reset() 方法用于将屏障重置为初始状态。


在CyclicBarrier中，同一批线程属于同一代。当有parties个线程到达barrier，generation就会被更新换代。其中broken标识该当前CyclicBarrier是否已经处于中断状态。
示例代码，开会协议： 
```java
import java.util.concurrent.CyclicBarrier;

/**
* @author 作者 : coderlong
* @version 创建时间：2018年4月1日 下午10:06:51
* 类说明: 
*/
public class CyclicbarrierTest {
	private static CyclicBarrier cyclicBarrier;
	static class CyclicBarrierThread extends Thread {
		@Override
		public void run() {
			// TODO Auto-generated method stub
			System.out.println("一个成员到达");
			try {
				cyclicBarrier.await(); // 等待
			} catch (Exception e) {
				// TODO: handle exception
			}
		}
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		cyclicBarrier = new CyclicBarrier(10, new Runnable() {
			
			@Override
			public void run() {
				// TODO Auto-generated method stub
				System.out.println("人都到齐了，开会！");
			}
		});
		for (int i = 0; i < 10; i++) {
			new CyclicBarrierThread().start();
		}
	}

}
```

	一个成员到达
	一个成员到达
	一个成员到达
	一个成员到达
	一个成员到达
	一个成员到达
	一个成员到达
	一个成员到达
	一个成员到达
	一个成员到达
	人都到齐了，开会！

# CountDownLatch
CountDownLatch所描述的是”在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待“。
教科书--》用给定的计数 初始化 CountDownLatch。由于调用了 countDown() 方法，所以在当前计数到达零之前，await 方法会一直受阻塞。之后，会释放所有等待的线程，await 的所有后续调用都将立即返回。这种现象只出现一次——计数无法被重置。如果需要重置计数，请考虑使用 CyclicBarrier。 
## gouzaofangfa
```java
public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }
```
此处的sync就是前面说过那个鼎鼎大名的AQS同步器，所以学好AQS还是很有必要的。
```java
private static final class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 4982264981922014374L;

    Sync(int count) {
        setState(count);
    }

    //获取同步状态
    int getCount() {
        return getState();
    }

    //获取同步状态
    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }

    //释放同步状态
    protected boolean tryReleaseShared(int releases) {
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```
await()方法，使当前线程在计数器至0之前，一直等待，除非发生中断
```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```
内部调用sync的acquireSharedInterruptibly(int arg)
```java
public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
```
sync重写了tryAcquireShared(int arg)
```java
protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}
```
CAS方法getState()获取同步状态，如果计数器值不等于0，则会调用doAcquireSharedInterruptibly(int arg)，该方法为一个自旋方法会尝试一直去获取同步状态
```java
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                /**
                 * 对于CountDownLatch而言，如果计数器值不等于0，那么r 会一直小于0
                 */
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            //等待
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
### countDown()
CountDownLatch提供countDown() 方法递减锁存器的计数，如果计数到达零，则释放所有等待的线程。

### 示例代码，开会
```java
import java.util.concurrent.CountDownLatch;

/**
* @author 作者 : coderlong
* @version 创建时间：2018年4月1日 下午10:26:19
* 类说明: 
*/
public class CountDownLatchTest {
	private static CountDownLatch countDownLatch;
	
	static class Bossthread extends Thread {
		@Override
		public void run() {
			// TODO Auto-generated method stub
			System.out.println("老板在等待，本次会议需要" + countDownLatch.getCount() + "位员工");
			try {
				countDownLatch.await();
			} catch (Exception e) {
				// TODO: handle exception
			}
			System.out.println("所有员工到达，开始开会");
		}
	}
	//员工到达会议室
    static class EmpleoyeeThread  extends Thread{
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "，到达会议室....");
            //员工到达会议室 count - 1
            countDownLatch.countDown();
        }
    }
    
    
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		countDownLatch = new CountDownLatch(5);
		new Bossthread().start();
		for (int i = 0; i < 5; i++) {
			new EmpleoyeeThread().start();
		}
	}

}
```
# Semaphore 
信号量Semaphore是一个控制访问多个共享资源的计数器，和CountDownLatch一样，其本质上是一个“共享锁”。
Semaphore提供了两个构造函数：
1. Semaphore(int permits) ：创建具有给定的许可数和非公平的公平设置的 Semaphore。
2. Semaphore(int permits, boolean fair) ：创建具有给定的许可数和给定的公平设置的 Semaphore。
所以内部的sync有公平和非公平之分
当然 Semaphore默认选择非公平锁。绝大时候Java默认都是非公平  
当信号量Semaphore = 1 时，它可以当作互斥锁使用。
信号量的获取  

```java
public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
```
调用AQS的acquireSharedInterruptibly(int arg)，该方法以共享模式获取同步状态：
```java
public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
```
上面的这个方法一般都是交付给子类来完成的,而且分公平，非公平之说:  
```java
protected int tryAcquireShared(int acquires) {
    for (;;) {
        //判断该线程是否位于CLH队列的列头
        if (hasQueuedPredecessors())
            return -1;
        //获取当前的信号量许可
        int available = getState();

        //设置“获得acquires个信号量许可之后，剩余的信号量许可数”
        int remaining = available - acquires;

        //CAS设置信号量
        if (remaining < 0 ||
                compareAndSetState(available, remaining))
            return remaining;
    }
}
```
非公平
```java
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```
用完之后释放就OK了。当然还是交给了内部的AQS来实现的
# Exchanger
好吧，这个我并不是很懂。  
API这样说：  
可以在对中对元素进行配对和交换的线程的同步点。每个线程将条目上的某个方法呈现给 exchange 方法，与伙伴线程进行匹配，并且在返回时接收其伙伴的对象。Exchanger 可能被视为 SynchronousQueue 的双向形式。Exchanger 可能在应用程序（比如遗传算法和管道设计）中很有用。原谅我的算法渣渣，什么遗传算法，也只是听说过而已，从未动手写过。
可以去看[这位大神的分析](http://brokendreams.iteye.com/blog/2253956)。
四大组件，就到这里吧，杀死我N个脑细胞，不过使用JUC自带的并发结构，好处多多，自己体会吧。
