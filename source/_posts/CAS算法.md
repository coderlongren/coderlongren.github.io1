  ---
title: CAS算法
date: 2018-01-25 12:50:17
categories:
	- concurrent
tags:
	- Java
	- concurrent
	- JDK源码
---
## 写在前面的话
CAS（Compare and Swap），即比较并替换，实现并发算法时常用到的一种技术，Doug lea大神在java同步器中大量使用了CAS技术，鬼斧神工的实现了多线程执行的安全性。
## CAS算法思想：
三个参数，一个当前内存值V、旧的预期值A、即将更新的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做，并返回false。

### 我们可以先来看一段Java自增代码 n++
```
public class Case {
 
    public volatile int n;
 
    public void add() {
        n++;
    }
}
```
通过 javap-verbose Case指令看add()方法的字节码
```
public void add();
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0       
         1: dup           
         2: getfield      #2                  // Field n:I
         5: iconst_1      
         6: iadd          
         7: putfield      #2                  // Field n:I
        10: return
```
n++被分成了下面几个指令，所有Volatile是无法保证原子性得。

1. 执行getfield拿到原始n；
2. 执行iadd进行加1操作；
3. 执行putfield写把累加后的值写回n；

通过volatile修饰的变量可以保证线程之间的可见性，但并不能保证这3个指令的原子执行，在多线程并发执行下，无法做到线程安全。
在add（）方法上加上synchronized关键字
```
public class Case {
		public volatile int n;
		public synchronized void add(){
		n++;
	}
}
```
除了低性能的加锁方案，我们还可以使用JDK自带的CAS方案，在CAS中，比较和替换是一组原子操作，不会被外部打断，且在性能上更占有优势。
下面可以用AtomicInteger实现为例，分析一下CAS算法的实现
```
public class AtomicInteger extends Number implements java.io.Serializable {
    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;
 
    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
 
    private volatile int value;
    public final int get() {return value;}
}
```

 1. Unsafe，是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法	来访问，Unsafe相当于一个后门，基于该类可以直接操作特定内存的数据。
 2. 变量valueOffset，表示该变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的。
 3. 变量value用volatile修饰，保证了多线程之间的内存可见性。
 ```
 public final int getAndAdd(int delta) {    
    return unsafe.getAndAddInt(this, valueOffset, delta);
}
 
//unsafe.getAndAddInt
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
 ```

  AtomicInteger里面的value原始值为3，即主内存中AtomicInteger的value为3，根据Java内存模型，线程A和线程B各自持有一份value的副本，值为3。  
    线程A通过getIntVolatile(var1, var2)拿到value值3，这时线程A被挂起。
    线程B也通过getIntVolatile(var1, var2)方法获取到value值3，运气好，线程B没有被挂起，并执行compareAndSwapInt方法比较内存值也为3，成功修改内存值为2。
    这时线程A恢复，执行compareAndSwapInt方法比较，发现自己手里的值(3)和内存的值(2)不一致，说明该值已经被其它线程提前修改过了，那只能重新来一遍了。
    重新获取value值，因为变量value被volatile修饰，所以其它线程对它的修改，线程A总是能够看到，线程A继续执行compareAndSwapInt进行比较替换，直到成功。

unsafe是一个在unsafe.cpp下面的native方法，再往下面就是汇编代码了，我就不回了。
## CAS缺点
典型的ABA问题，也是很明显的  
如果变量V初次读取的时候是A，并且在准备赋值的时候检查到它仍然是A，那能说明它的值没有被其他线程修改过了吗？
如果在这段期间曾经被改成B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。针对这种情况，java并发包中提供了一个带有标记的原子引用类AtomicStampedReference，它可以通过控制变量值的版本来保证CAS的正确性。AtomicStampedReference通过包装[E,Integer]的元组来对对象标记版本戳stamp，从而避免ABA问题。
