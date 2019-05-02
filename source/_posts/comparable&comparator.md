---
title: Comparable与Comparator的区别
date: 2018-05-13 12:50:17
categories:
	- jdk
tags:
	- Java
	- concurrent
	- JDK源码
---

摘要：String 类 如何实现Comparator
<!-- more -->

# String如何策略模式来实现自己的Comparator
EffectiveJava 看到Comparator。  
我们在使用String的compareToIgnoreCase方法时，String类是如何提供这个经常会使用的String功能呢？一般我们自己写的时候，估计都会想到传进去一个Comparator<T> 也就是说实现一个匿名内部类就行了。但是前面说了，String会经常用到这个比较。每次都创建一个类花销太大。 
这时候，就可以考虑把这个比较器放到一个私有的static final 域内，重用他。  
`因为所有的策略接口被用作所有具体策略实例的类型，我们应该声明一个接口来表示我们的策略，并且为每个具体的策略声明一个实现了该接口的类，` 当这个具体的策略只使用一次， 可以匿名实例化这个接口类，当他用在JDK设计上的时候，他的实现类就要被声明private static final类，通过public static final 来导出这个策略类。  
自己下去翻了翻String的源码，真的和EffectiveJava书中所讲的分毫不差。 
源码：
忽略大小写比较
```java
  public int compareToIgnoreCase(String str) {
      return CASE_INSENSITIVE_ORDER.compare(this, str);
  }
```
如何构造那个CASE_INSENSITIVE_ORDER呢？  
```java
  public static final Comparator<String> CASE_INSENSITIVE_ORDER
                                       = new CaseInsensitiveComparator();

```

```
  private static class CaseInsensitiveComparator
          implements Comparator<String>, java.io.Serializable {
      // use serialVersionUID from JDK 1.2.2 for interoperability
      private static final long serialVersionUID = 8575799808933029326L;

      public int compare(String s1, String s2) {
          int n1 = s1.length();
          int n2 = s2.length();
          int min = Math.min(n1, n2);
          for (int i = 0; i < min; i++) {
              char c1 = s1.charAt(i);
              char c2 = s2.charAt(i);
              if (c1 != c2) {
                  c1 = Character.toUpperCase(c1);
                  c2 = Character.toUpperCase(c2);
                  if (c1 != c2) {
                      c1 = Character.toLowerCase(c1);
                      c2 = Character.toLowerCase(c2);
                      if (c1 != c2) {
                          // No overflow because of numeric promotion
                          return c1 - c2;
                      }
                  }
              }
          }
          return n1 - n2;
      }

      /** Replaces the de-serialized object. */
      private Object readResolve() { return CASE_INSENSITIVE_ORDER; }
  }
```

### Comparable 自然排序
Comparable 在 java.lang 包下，是一个接口，内部只有一个方法 compareTo()：
```
public interface Comparable<T> {
    public int compareTo(T o);
}
```
Comparable 可以让实现它的类的对象进行比较，具体的比较规则是按照 compareTo 方法中的规则进行。这种顺序称为 自然顺序。
compareTo方法有三种返回值
* e1.compareTo(e2) > 0 即 e1 > e2
* e1.compareTo(e2) = 0 即 e1 = e2
* e1.compareTo(e2) < 0 即 e1 < e2

一个简单的例子
```
public class Domain implements Comparable<Domain>
{
    private String str;

    public Domain(String str)
    {
        this.str = str;
    }

    public int compareTo(Domain domain)
    {
        if (this.str.compareTo(domain.str) > 0)
            return 1;
        else if (this.str.compareTo(domain.str) == 0)
            return 0;
        else 
            return -1;
    }
    
    public String getStr()
    {
        return str;
    }
}


public static void main(String[] args)
    {
        Domain d1 = new Domain("c");
        Domain d2 = new Domain("c");
        Domain d3 = new Domain("b");
        Domain d4 = new Domain("d");
        System.out.println(d1.compareTo(d2));
        System.out.println(d1.compareTo(d3));
        System.out.println(d1.compareTo(d4));
    }
```
## comparator
如果我们想对一个整形数组，比价绝对值？

1. 首先写一个自己想要的实现comparator的比较器
```
// AbsComparator.java     
import   java.util.*;
 
public   class   AbsComparator   implements   Comparator   {     
    public   int   compare(Object   o1,   Object   o2)   {     
      int   v1   =   Math.abs(((Integer)o1).intValue());     
      int   v2   =   Math.abs(((Integer)o2).intValue());     
      return   v1   >   v2   ?   1   :   (v1   ==   v2   ?   0   :   -1);     
    }     
}
```
2. 然后在调用JDK比较函数，Arrays.sort(), 或者 Collections.sort()时，传入自己的比较器即可。
```
import   java.util.*;     
 
public   class   Test   {     
    public   static   void   main(String[]   args)   {     
 
      //产生一个20个随机整数的数组（有正有负）     
      Random   rnd   =   new   Random();     
      Integer[]   integers   =   new   Integer[20];     
      for(int   i   =   0;   i   <   integers.length;   i++)     
      integers[i]   =   new   Integer(rnd.nextInt(100)   *   (rnd.nextBoolean()   ?   1   :   -1));     
 
      System.out.println("用Integer内置方法排序：");     
      Arrays.sort(integers);     
      System.out.println(Arrays.asList(integers));     
 
      System.out.println("用AbsComparator排序：");     
      Arrays.sort(integers,   new   AbsComparator());     
      System.out.println(Arrays.asList(integers));     
    }     
}
```
