---
title: Full GC的触发原因
date: 2019-04-19 10:29:27
categories:
	- JVM
tags:
	- JVM
---
Full GC探究 
<!-- more -->

组内revirew故障的时候，经常提到提到有的机器抽风，Full GC太频繁，萌新的我:flush:还没有JVM线上排查问题的经验，自己下去查了下JVM触发Full GC的原因。
* 代码中执行System.gc()回向JVM发送执行fullGC的命令，但不一定会立即执行。
* 用jmap 命令查看堆内存快照，这个会立即触发, 一般是排查问题使用。
* 年轻代执行minor gc会通过一系列条件来判断是否执行 full gc.
```
执行Minor GC的时候，JVM会检查老年代中最大连续可用空间是否大于了当前新生代所有对象的总大小。 
--> 如果大于，则直接执行Full GC。 
--> 如果小于了，JVM会检查是否开启了空间分配担保机制，如果没有开启则直接改为执行Full GC。这也是复制算法的缺点。
--> 如果开启了，则JVM会检查老年代中最大连续可用空间是否大于了历次晋升到老年代中的平均大小，如果小于则执行改为执行Full GC。 如果大于则会执行Minor GC，如果Minor GC执行失败则会执行Full GC
```

# JVM内存分配策略
## 对象优先在Eden分配
在大多数情况下，对象会现在新生代Eden中分配，当Eden中没有足够的空间，jvm会触发一次minor GC.  
## 大对象直接进入老年代
大对象也就是需要大的连续空间的Java Bean.典型的就是大的数组和字符串。
## 长期存活的对象进入老年代
-XX:MaxTenuringThreshold参数用来设置新生代存活对象的年龄阈值(默认是15), 我们知道新生代使用复制算法分为8:1:1的内存空间。始终用一块Survivor空间来保存存活对象，每经历一次Minor GC 对象从Eden Survivor移动到另一块Survivor区间，对象年龄就会+1, 如果达到了XX:MaxTenuringThreshold就会晋升到老年代中。
## 对象年龄动态判定
为了能够更好地适应不同代码的内存状况，JVM并不总是要求对象年龄到达XX:MaxTenuringThreshold采取移动到老年代。如果Survivor空间中相同年龄所有对象大小之和大于Survivor空间的一半，那么那些年龄大于这个年龄的对象就会直接进入到老年代。无需等待XX:MaxTenuringThreshold阈值。
## 空间内存担保
前面提到了Minor GC时如果老年代大小小于新生代的总大小，会判定JVM是否开启内存担保。其实这也是JVM应对大量对象在Minor GC后存活的情况，老年代进行内存担保，每次Minor GC会有多少对象活下来，在实际完成内存回收之前是不知道的，所以只好取之前每次晋升到老年代的平均大小作为判定条件经验值，来决定是否执行一次Full GC腾出更多的空间。
内存担保使用HandlePromitionFailure参数来控制,值为true or false.