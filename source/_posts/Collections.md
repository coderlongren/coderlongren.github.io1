---
title: Collections工具类使用
date: 2017-12-27 12:49:59
categories:
	- Java
tags:
	- Java
	- java集合
	- JDK
---
摘要：没有摘要
![](/images/collection.jpg)
<!-- more -->
正文：

# 排序操作
void reverse(List list)：反转  

void shuffle(List list),随机排序  

void sort(List list),按自然排序的升序排序  

void sort(List list, Comparator c);定制排序，由Comparator控制排序逻辑  

void swap(List list, int i , int j),交换两个索引位置的元素  

void rotate(List list, int distance),  旋转。当distance为正数时，将list后distance个元素整体移到前面。当distance为负数时，将 list的前distance个元素整体移到后面。  
```
package collection;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;

public class CollectionsTest {
    public static void main(String[] args) {
        ArrayList nums =  new ArrayList();
        nums.add(8);
        nums.add(-3);
        nums.add(2);
        nums.add(9);
        nums.add(-2);
        System.out.println(nums);
        Collections.reverse(nums);
        System.out.println(nums);
        Collections.sort(nums);
        System.out.println(nums);
        Collections.shuffle(nums);
        System.out.println(nums);
        //下面只是为了演示定制排序的用法，将int类型转成string进行比较
        Collections.sort(nums, new Comparator() {
            
            @Override
            public int compare(Object o1, Object o2) {
                // TODO Auto-generated method stub
                String s1 = String.valueOf(o1);
                String s2 = String.valueOf(o2);
                return s1.compareTo(s2);
            }
            
        });
        System.out.println(nums);
    }
}
```
# 查找，替换操作
int binarySearch(List list, Object key), 对List进行二分查找，返回索引，注意List必须是有序的

int max(Collection coll),根据元素的自然顺序，返回最大的元素。 类比int min(Collection coll)

int max(Collection coll, Comparator c)，根据定制排序，返回最大元素，排序规则由Comparatator类控制。类比int min(Collection coll, Comparator c)

void fill(List list, Object obj),用元素obj填充list中所有元素

int frequency(Collection c, Object o)，统计元素出现次数

int indexOfSubList(List list, List target), 统计targe在list中第一次出现的索引，找不到则返回-1，类比int lastIndexOfSubList(List source, list target).

boolean replaceAll(List list, Object oldVal, Object newVal), 用新元素替换旧元素。

# 同步控制
Collections中几乎对每个集合都定义了同步控制方法，例如 SynchronizedList(), SynchronizedSet()等方法，来将集合包装成线程安全的集合。下面是Collections将普通集合包装成线程安全集合的用法
```
package collection.collections;

import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;

public class SynchronizedTest {
    public static void main(String[] args) {
        Collection c = Collections.synchronizedCollection(new ArrayList());
        List list = Collections.synchronizedList(new ArrayList());
        Set s = Collections.synchronizedSet(new HashSet());
        Map m = Collections.synchronizedMap(new HashMap());
    }
}
```

设置不可变（只读）集合

Collections提供了三类方法返回一个不可变集合，

emptyXXX(),返回一个空的只读集合（这不知用意何在？）

singleXXX()，返回一个只包含指定对象，只有一个元素，只读的集合。

unmodifiablleXXX()，返回指定集合对象的只读视图。

##  我顺便看了一下，java.util.concueent包的源码结构
大概就是五部分
1. Aomic数据类型
这部分都被放在java.util.concurrent.atomic这个包里面，实现了原子化操作的数据类型，包括 Boolean, Integer, Long, 和Referrence这四种类型以及这四种类型的数组类型。
2. 锁机制
在java.util.concurrent.lock这个包里面，实现了并发操作中的几种类型的锁
3. 对现有Java集合的并发实现
有List, Queue和Map。
4. 多线程部分
Callable     被执行的任务
Executor  执行任务
Future      异步提交任务的返回数据
5. 线程管理
对线程集合的管理的实现，有CyclicBarrier, CountDownLatch,Exchanger等一些类
