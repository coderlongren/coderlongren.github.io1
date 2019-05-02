---
title: Arrays.copyof()&&System.arraycopy()
date: 2018-2-1 16:18:40
categories:
	- 有趣的API
tags:
	- JDK
	- API解析
---
System.arraycopy()的声明
```java
public static native void arraycopy(Object src,int srcPos, Object dest, int destPos,int length);
```
一个native方法 ，说明是调用的其他语言的底层函数  
src - 源数组。   
srcPos - 源数组中的起始位置。  
dest - 目标数组。  
destPos - 目标数据中的起始位置。  
length - 要复制的数组元素的数量。  

---
Arrays.copyof()的声明呢？

```java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

original - 要复制的数组  
newLength - 要返回的副本的长度  
newType - 要返回的副本的类型  

---

copyof内部调用了system.arraycopy()方法
arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置   

copyOf()是系统自动在内部新建一个数组，调用arraycopy()
将original内容复制到copy中去，并且长度为newLength。返回copy; 
即将原数组拷贝到一个长度为newLength的新数组中，并返回该数组。


