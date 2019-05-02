---
title: Hash
date: 2018-2-5 14:18:40
categories:
    - JDK
tags:
    - JDK
---
## Java中hashCode()方法以及HashMap()中 hash()方法
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
hashCode()是一个native方法，意味着方法的实现和硬件平台有关，默认实现和虚拟机是相关的，  
对于有些JVM，hashcode()返回的就是对象的地址。  
在Java中，hashCode()方法的主要作用是为了配合基于散列的集合（HashSet、HashMap）一起正常运行。  
当向集合中插入对象时，调用equals()逐个进行比较，这个方法可行却效率低下。  
因此，先比较hashCode再调用equals()会快很多。下面这段代码是java.util.HashMap的中put方法的具体实现： 
```
public V put(K key, V value) {
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key.hashCode());
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
  
        modCount++;
        addEntry(hash, key, value, i);
        return null;
```


![](/images/huda1.jpeg)




## 附上，native关键字的说明
native关键字说明其修饰的方法是一个原生态方法，方法对应的实现不是在当前文件，
而是在用其他语言（如C和C++）实现的文件中。Java语言本身不能对操作系统底层进行访问和操作，
但是可以通过JNI接口调用其他语言来实现对底层的访问。
JNI是Java本机接口（Java Native Interface），是一个本机编程接口，它是Java软件开发工具箱（Java Software Development Kit，SDK）
的一部分。JNI允许Java代码使用以其他语言编写的代码和代码库。Invocation API（JNI的一部分）
可以用来将Java虚拟机（JVM）嵌入到本机应用程序中，从而允许程序员从本机代码内部调用Java代码。