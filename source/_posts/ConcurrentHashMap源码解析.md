---
title: ConcurrentHashMap源码剖析
date: 2018-3-5 11:18:40
categories:
	- JDK
tags:
	- JDK
	- 代码重构
	- 源码学习
---
转载出处: https://www.cnblogs.com/chengxiao/p/6842045.html

Segment构造FANGFA	
```java
Segment(float lf, int threshold, HashEntry<K,V>[] tab) {
            this.loadFactor = lf;//负载因子
            this.threshold = threshold;//阈值
            this.table = tab;//主干数组即HashEntry数组
        }
```
concurrenthashmap构造方法
```java
public ConcurrentHashMap(int initialCapacity,
                               float loadFactor, int concurrencyLevel) {
          if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
              throw new IllegalArgumentException();
          //MAX_SEGMENTS 为1<<16=65536，也就是最大并发数为65536
          if (concurrencyLevel > MAX_SEGMENTS)
              concurrencyLevel = MAX_SEGMENTS;
          //2的sshif次方等于ssize，例:ssize=16,sshift=4;ssize=32,sshif=5
         int sshift = 0;
         //ssize 为segments数组长度，根据concurrentLevel计算得出
         int ssize = 1;
         while (ssize < concurrencyLevel) {
             ++sshift;
             ssize <<= 1;
         }
         //segmentShift和segmentMask这两个变量在定位segment时会用到，后面会详细讲
         this.segmentShift = 32 - sshift;
         this.segmentMask = ssize - 1;
         if (initialCapacity > MAXIMUM_CAPACITY)
             initialCapacity = MAXIMUM_CAPACITY;
         //计算cap的大小，即Segment中HashEntry的数组长度，cap也一定为2的n次方.
         int c = initialCapacity / ssize;
         if (c * ssize < initialCapacity)
             ++c;
         int cap = MIN_SEGMENT_TABLE_CAPACITY;
         while (cap < c)
             cap <<= 1;
         //创建segments数组并初始化第一个Segment，其余的Segment延迟初始化
         Segment<K,V> s0 =
             new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                              (HashEntry<K,V>[])new HashEntry[cap]);
         Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
         UNSAFE.putOrderedObject(ss, SBASE, s0); 
         this.segments = ss;
     }
```

put()
```java
public V put(K key, V value) {
        Segment<K,V> s;
        //concurrentHashMap不允许key/value为空
        if (value == null)
            throw new NullPointerException();
        //hash函数对key的hashCode重新散列，避免差劲的不合理的hashcode，保证散列均匀
        int hash = hash(key);
        //返回的hash值无符号右移segmentShift位与段掩码进行位运算，定位segment
        int j = (hash >>> segmentShift) & segmentMask;
        if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
             (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
            s = ensureSegment(j);
        return s.put(key, hash, value, false);
    }
 ```

关于segmentShift和segmentMask

　　segmentShift和segmentMask这两个全局变量的主要作用是用来定位Segment，int j =(hash >>> segmentShift) & segmentMask。

　　segmentMask：段掩码，假如segments数组长度为16，则段掩码为16-1=15；segments长度为32，段掩码为32-1=31。这样得到的所有bit位都为1，可以更好地保证散列的均匀性

　　segmentShift：2的sshift次方等于ssize，segmentShift=32-sshift。若segments长度为16，segmentShift=32-4=28;若segments长度为32，segmentShift=32-5=27。而计算得出的hash值最大为32位，无符号右移segmentShift，则意味着只保留高几位（其余位是没用的），然后与段掩码segmentMask位运算来定位Segment。

get()
```java
public V get(Object key) {
        Segment<K,V> s; 
        HashEntry<K,V>[] tab;
        int h = hash(key);
        long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
        //先定位Segment，再定位HashEntry
        if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
            (tab = s.table) != null) {
            for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                     (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
                 e != null; e = e.next) {
                K k;
                if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                    return e.value;
            }
        }
        return null;
    }
 ```

 为什么这里的get()不需要加锁呢？ 

	 static final class HashEntry<K,V> {
	        final int hash;
	        final K key;
	        volatile V value;
	        volatile HashEntry<K,V> next;
	        //其他省略
	}
value 和next指针都是volatile修饰的，可以保证可见性

segment的put方法
```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    HashEntry<K,V> node = tryLock() ? null :
        scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;//定位HashEntry，可以看到，这个hash值在定位Segment时和在Segment中定位HashEntry都会用到，只不过定位Segment时只用到高几位。
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                K k;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            else {
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
　　　　　　　　　　　　　　//若c超出阈值threshold，需要扩容并rehash。
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        unlock();
    }
    return oldValue;
}
```