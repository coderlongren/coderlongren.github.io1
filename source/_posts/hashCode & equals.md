---
title: HashCode 
date: 2018-2-5 14:18:40
categories:
    - JDK
tags:
    - JDK
---
#  Java中的equals()和hashCode() 以及 '=='的比较辨析：
1. java中的==是比较两个对象在JVM中的地址
```
public class ComAddr{
    public static void main(String[] args) throws Exception {
        String s1 = "nihao";
        String s2 = "nihao";
        String s3 = new String("nihao");
        System.out.println(s1 == s2);    //    true
        System.out.println(s1 == s3);    //    false
    }
}
```
因为s1 == s2 为true，因为s1,s2都是字符串字面值"nihao"的引用，指向同一块地址，所以相等。  
s1 == s3为false，是因为通过new产生的对象在堆中，s3是堆中变量的引用，而是s1是指向字符串字面值"nihao"的引用，地址不同所以不相等。
2. equals() 是object根类中的方法，源代码如下：
```
public boolean equals(Object obj) {
    return (this == obj);
}
```
==比较的是两个对象是否是同一个对象，这并不能满足很多需求。  
有时候当两个对象不==的时候，我们仍然会认为两者是“相等”的，比如对于String对象，  
当两个对象的字符串序列是一致的，我们就认为他们是“相等”的。对于这样的需求，需要equals()来实现。  
对于有这种需求的对象的类，重写其equals()方法便可，具体的“相等”逻辑可以根据需要自己定义。  
equals()和hashCode()都是从Object类中继承而来的，而Object中equals()默认实现的是比较两个对象是否==，  
即默认的equals()和==的效果是一样的。 
值得注意的是，当String 、Math、还有Integer、Double。。。。等这些封装类在使用equals()方法时，  
已经覆盖了object类的equals（）方法
下面是string类中覆盖的equals()方法 
```
public boolean equals(Object anObject) {
	if (this == anObject) {
	    return true;
	}
	if (anObject instanceof String) {
	    String anotherString = (String)anObject;
	    int n = count;
	    if (n == anotherString.count) {
		char v1[] = value;
		char v2[] = anotherString.value;
		int i = offset;
		int j = anotherString.offset;
		while (n-- != 0) {
		    if (v1[i++] != v2[j++])
			return false;
		}
		return true;
	    }
	}
	return false;
    }
```
indexFor() 源码如下:
```
static int indexFor(int h, int length) { 
    return h & (length-1); 
} 
```
hash()方法在JDk1.7中如下
```
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```
这样设计保证了对象的hashCode的32位值只要有一位发生改变，整个hash()返回值就会改变，高位的变化会反应到低位里。  
String类中equals()方法很明显是仅仅进行了对象内容的比较，而没有比较对象存储地址的比较
## 关于HashCode()方法
先加上一段JDK中hashcCode() 方法的源代码[我在我的另一片文章里面已经分析了HashCode()方法得底层原理]
```
public final native Class<?> getClass(); 
public native int hashCode(); 
public boolean equals(Object obj) { 
  return (this == obj); 
}  
public String toString() { 
 return getClass().getName() + "@" +  Integer.toHexString(hashCode()); 
} 
```
比如String、Integer、Double等等这些类都是覆盖了hashCode()方法,再来看一下String类中覆盖的HashCode()方法
```
public int hashCode() { 
int h = hash; 
if (h == 0) { 
    int off = offset; 
    char val[] = value; 
    int len = count; 

            for (int i = 0; i < len; i++) { 
                h = 31*h + val[off++]; 
            } 
            hash = h; 
        } 
        return h; 
} 
```
![](/images/huda3.jpeg)
未完，待续。。。。。




