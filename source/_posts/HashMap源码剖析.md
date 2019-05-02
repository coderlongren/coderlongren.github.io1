---
title: HashMap源码剖析
date: 2018-2-5 14:18:40
categories:
	- JDK
tags:
	- JDK
	- 代码重构
	- 源码学习
---
## HashMap源码剖析
前面了解了jdk容器中的两种List，回忆一下怎么从list中取值（也就是做查询），是通过index索引位置对不对，由于存入list的元素时安装插入顺序存储的，所以index索引也就是插入的次序。
　　Map呢是这样一种容器，它可以存储两个元素键和值，根据键这个关键字可以明确且唯一的查出一个值，这个过程很像查字典，考虑一下使用什么样的数据结构才能实现这种效果呢？
### 1 自己实现一个map
jdk中map的定义
```java
public interface Map<K,V> {
    int size();
    boolean isEmpty();
    boolean containsKey(Object key);
    boolean containsValue(Object value);
    V get(Object key);
    V put(K key, V value);
    V remove(Object key);
    void putAll(Map<? extends K, ? extends V> m);
    void clear();
    Set<K> keySet();
    Collection<V> values();
    Set<Map.Entry<K, V>> entrySet();
    interface Entry<K,V> {
       V getValue();
       V setValue(V value);
        boolean equals(Object o);
        int hashCode();
    }
    boolean equals(Object o);
    int hashCode();
}
```
可以看到Map并没有实现Collection接口，也没有实现List接口，因为它可以保存两个属性key-value，和List容器一样还是包含增删改查等基本操作，同时可以看到Map中还定义了一个用来表示键值K-V的接口Entry。 

在了解了map的概念和定义后，首先我们自己先来简单写一个Map的实现，看看会遇到什么样的问题。
```java
public class MyMap {

        private Entry[] data = new Entry[100];
        private int size;

        public Object put(Object key, Object value) {
               // 检查key是否存在，存在则覆盖
               for (int i = 0; i < size; i++) {
                      if (key.equals(data [i].key)) {
                           Object oldValue = data[i].value ;
                            data[i].value = value;
                            return oldValue;
                     }
              }
              
              Entry e = new Entry(key, value);
               data[size ] = e;
               size++;
              
               return null;
       }

        public Object get(Object key) {
               for (int i = 0; i < size; i++) {
                      if (key.equals(data [i].key)) {
                            return data [i].value;
                     }
              }

               return null;
       }
       
        public int size() {
               return size ;
       }
       
        private class Entry {
              Object key;
              Object value;

               public Entry(Object key, Object value) {
                      this.key = key;
                      this.value = value;
              }

       }
}
```
上面我们简单实现了一下map的put、get、size等方法，从代码可以看到底层是使用数组来存储数据的。
```java
public class Test {

        public static void main(String[] args) {
              MyMap map = new MyMap();
              map.put( "tstd", "angelababy" );
              map.put( "张三" , "郭靖");
              map.put( "tstd", "黄蓉" );
              
              System. out.println(map.size());
              System. out.println(map.get("tstd" ));
              System. out.println(map.get("张三" ));
       }
}
```
*致命缺陷* ：get方法中，通过key获取value的方式是通过遍历数组实现，这样显然是非常低效的，同样在put方法中由于要检查key是否已经存在也是通过遍历数组实现
## HashMap的定义
在看HashMap定义前，我们首先需要了解hash是什么意思，hash通常被翻译成“散列”，简单解析下（不对的话还请指出^_^），* hash就是通过散列算法，将一个任意长度关键字转换为一个固定长度的散列值 *，但是有一点要指出的是，不同的关键字可能会散列出相同的散列值。什么意思呢？也就是关键字和散列值不是一一对应的，散列值会出现冲突。但是为什么会出现这种情况呢，原因是hash是一种压缩映射，举个例子就是将一个8个字节（二进制64位）的long值转换为一个4个字节（二进制32位）的int值，也就是说需要砍掉4个字节（32位），坑位有限，人太多，所以只能两个人一个坑喽。
     ok、了解了hash的概念和特点后，来看下HashMap的定义：

```java
public class HashMap<K,V>
     extends AbstractMap<K,V>
     implements Map<K,V>, Cloneable, Serializable
```
可以看出HashMap集成了AbstractMap抽象类，实现了Map，Cloneable，Serializable接口，AbstractMap抽象类继承了Map提供了一些基本的实现。

## 底层存储
```java
// 默认初始容量为16，必须为2的n次幂
  static final int DEFAULT_INITIAL_CAPACITY = 16;

  // 最大容量为2的30次方
  static final int MAXIMUM_CAPACITY = 1 << 30;

  // 默认加载因子为0.75f
  static final float DEFAULT_LOAD_FACTOR = 0.75f;

  // Entry数组，长度必须为2的n次幂
  transient Entry[] table;

  // 已存储元素的数量
  transient int size ;

  // 下次扩容的临界值，size>=threshold就会扩容，threshold等于capacity*load factor
  int threshold;

  // 加载因子
  final float loadFactor ;

```

可以看到hashMap底层使用Entry数组存储数据，同时定义了初始容量，最大容量，加载因子等参数，
  

  那么我们来看一下Entry的定义吧。

```java
static class Entry<K,V> implements Map.Entry<K,V> {
        final K key ; 
        V value;
        Entry<K,V> next; // 指向下一个节点
        final int hash;

        Entry( int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }

        public final K getKey() {
            return key ;
        }

        public final V getValue() {
            return value ;
        }

        public final V setValue(V newValue) {
           V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            Object k1 = getKey();
            Object k2 = e.getKey();
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {
                Object v1 = getValue();
                Object v2 = e.getValue();
                if (v1 == v2 || (v1 != null && v1.equals(v2)))
                    return true;
            }
            return false;
        }

        public final int hashCode() {
            return (key ==null   ? 0 : key.hashCode()) ^
                   ( value==null ? 0 : value.hashCode());
        }

        public final String toString() {
            return getKey() + "=" + getValue();
        }

        // 当向HashMap中添加元素的时候调用这个方法，这里没有实现是供子类回调用
        void recordAccess(HashMap<K,V> m) {
        }

        // 当从HashMap中删除元素的时候调动这个方法 ，这里没有实现是供子类回调用
        void recordRemoval(HashMap<K,V> m) {
        }
}
```

　Entry是HashMap的内部类，它继承了Map中的Entry接口，它定义了键(key)，值(value)，和下一个节点的引用(next)，以及hash值。很明确的可以看出Entry是什么结构，它是单线链表的一个节点。也就是说HashMap的底层结构是一个数组，而数组的元素是一个单向链表。

![图片展示](https://images2015.cnblogs.com/blog/681047/201512/681047-20151217204107396-207877456.jpg)

## 再来看看hashMap的构造方法
```java
/**
     * 构造一个指定初始容量和加载因子的HashMap
     */
    public HashMap( int initialCapacity, float loadFactor) {
        // 初始容量和加载因子合法校验
        if (initialCapacity < 0)
            throw new IllegalArgumentException( "Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException( "Illegal load factor: " +
                                               loadFactor);

        // Find a power of 2 >= initialCapacity
        // 确保容量为2的n次幂，是capacity为大于initialCapacity的最小的2的n次幂
        int capacity = 1;
        while (capacity < initialCapacity)
            capacity <<= 1;

        // 赋值加载因子
        this.loadFactor = loadFactor;
        // 赋值扩容临界值
        threshold = (int)(capacity * loadFactor);
        // 初始化hash表
        table = new Entry[capacity];
        init();
    }

    /**
     * 构造一个指定初始容量的HashMap
     */
    public HashMap( int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * 构造一个使用默认初始容量(16)和默认加载因子(0.75)的HashMap
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
        table = new Entry[DEFAULT_INITIAL_CAPACITY];
        init(); // 这个init()方法 是在哪里调用的呢？ 看我的LinkedhashMap源码分析就知道了. 

    }

    /**
     * 构造一个指定map的HashMap，所创建HashMap使用默认加载因子(0.75)和足以容纳指定map的初始容量。
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        // 确保最小初始容量为16，并保证可以容纳指定map
        this(Math.max(( int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                      DEFAULT_INITIAL_CAPACITY ), DEFAULT_LOAD_FACTOR);
        putAllForCreate(m);
}
```
最后一个putAllForCreate()是什么呢？ 
```java
private void putAllForCreate(Map<? extends K, ? extends V> m) {
      for(Iterator<?extendsMap.Entry<?extendsK, ?extendsV>> i = m.entrySet().iterator(); i.hasNext(); ) {
            Map.Entry<? extends K, ? extends V> e = i.next();
            putForCreate(e.getKey(), e.getValue());
        }
    }

    /**
     * This method is used instead of put by constructors and
     * pseudoconstructors (clone, readObject).  It does not resize the table,
     * check for comodification, etc.  It calls createEntry rather than
     * addEntry.
     */
    private void putForCreate(K key, V value) {
        int hash = (key == null) ? 0 : hash(key.hashCode());
        int i = indexFor(hash, table.length );

        for (Entry<K,V> e = table [i]; e != null; e = e. next) {
            Object k;
            if (e.hash == hash &&
                ((k = e. key) == key || (key != null && key.equals(k)))) {
                e. value = value;
                return;
            }
        }

        createEntry(hash, key, value, i);
    }
   
   void createEntry(int hash, K key, V value, int bucketIndex) {
       Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
        size++;
}
```
你也理解为没有扩容的增加

## 增加

```java
public V put(K key, V value) {
        // 如果key为null，调用putForNullKey方法进行存储
        if (key == null)
            return putForNullKey(value);
        // 使用key的hashCode计算key对应的hash值
        int hash = hash(key.hashCode());
        // 通过key的hash值查找在数组中的index位置
        int i = indexFor(hash, table.length );
        // 取出数组index位置的链表，遍历链表找查看是有已经存在相同的key
        for (Entry<K,V> e = table [i]; e != null; e = e. next) {
            Object k;
            // 通过对比hash值、key判断是否已经存在相同的key
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                // 如果存在，取出当前key对应的value，供返回
                V oldValue = e. value;
                // 用新value替换之旧的value
                e. value = value;
                e.recordAccess( this);
                // 返回旧value，退出方法
                return oldValue;
            }
        }

        // 如果不存在相同的key
        // 修改版本+1
        modCount++;
        // 在数组i位置处添加一个新的链表节点
        addEntry(hash, key, value, i);
        // 没有相同key的情况，返回null
        return null;
    }

    private V putForNullKey(V value) {
        // 取出数组第1个位置（下标等于0）的节点，如果存在则覆盖不存在则新增，和上面的put一样不多讲，
        for (Entry<K,V> e = table [0]; e != null; e = e. next) {
            if (e.key == null) {
                V oldValue = e. value;
                e. value = value;
                e.recordAccess( this);
                return oldValue;
            }
        }
        modCount++;
        // 如果key等于null，则hash值等于0
        addEntry(0, null, value, 0);
        return null;
}
```
再说一遍 put元素的步骤
1. 使用key的hashCode计算key对应的hash值
2. 通过对比hash值，查找再数组中的位置，index
3. 取出数组index位置的链表，遍历链表找查看是有已经存在相同的key

这里面非常精彩的就是Hash算法了，不同版本JDK之间 Hash算法有差异
```java
/**
     * Applies a supplemental hash function to a given hashCode, which
     * defends against poor quality hash functions.  This is critical
     * because HashMap uses power -of- two length hash tables, that
     * otherwise encounter collisions for hashCodes that do not differ
     * in lower bits. Note: Null keys always map to hash 0, thus index 0.
     */
    static int hash(int h) {
        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

    /**
     * Returns index for hash code h.
     */
    static int indexFor(int h, int length) {
        return h & (length-1);
}
```
indexFor方法里面的h & (length-1)，其实就是就是当length=2的n次幂的时候，h & (length-1)的结果，就是0~(length-1)之间的数，而这个结果和h % length是一样的,  
如果你还研读过其它集合的源码，就会发现hashtable里面的 indexFor方法就是用的h % (length - 1),当然位运算是更快的.
只要知道这个结果，就ok，如果还想更加深入，有个大神写了一篇文章可以参考下http://yananay.iteye.com/blog/910460


## 增加一个节点

```java
/**
     * 增加一个k-v，hash组成的节点在数组内，同时可能会进行数组扩容。
     */
    void addEntry( int hash, K key, V value, int bucketIndex) {
        // 下面两行行代码的逻辑是，创建一个新节点放到单向链表的头部，旧节点向后移
        // 取出索引bucketIndex位置处的链表节点，如果节点不存在那就是null，也就是说当数组该位置处还不曾存放过节点的时候，这个地方就是null，
       Entry<K,V> e = table[bucketIndex];
       // 创建一个节点，并放置在数组的bucketIndex索引位置处，并让新的节点的next指向原来的节点
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
       // 如果当前HashMap中的元素已经到达了临界值，则将容量扩大2倍，并将size计数+1
        if (size ++ >= threshold)
            resize(2 * table.length );
}
```

这里面有一个需要注意的地方，将新节点指向原来的节点，这里虽然是next，但是却是往回指向的，而不是像上面图中画的由数组第1个节点往后指向，就是说第1个节点指向null，第2个节点指向第1个，第3个节点指向第2个。也就是新节点一直插入在最前端，新节点始终是单向列表的头节点。

再看下HashMap扩容

```java
/**
     * Rehashes the contents of this map into a new array with a
     * larger capacity.  This method is called automatically when the
     * number of keys in this map reaches its threshold.
     *
     * If current capacity is MAXIMUM_CAPACITY, this method does not
     * resize the map, but sets threshold to Integer.MAX_VALUE.
     * This has the effect of preventing future calls.
     *
     * @param newCapacity the new capacity, MUST be a power of two;
     *        must be greater than current capacity unless current
     *        capacity is MAXIMUM_CAPACITY (in which case value
     *        is irrelevant).
     */
    void resize( int newCapacity) {
        // 当前数组
        Entry[] oldTable = table;
        // 当前数组容量
        int oldCapacity = oldTable.length ;
        // 如果当前数组已经是默认最大容量MAXIMUM_CAPACITY ，则将临界值改为Integer.MAX_VALUE 返回
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        // 使用新的容量创建一个新的链表数组
        Entry[] newTable = new Entry[newCapacity];
        // 将当前数组中的元素都移动到新数组中
        transfer(newTable);
        // 将当前数组指向新创建的数组
        table = newTable;
        // 重新计算临界值
        threshold = (int)(newCapacity * loadFactor);
    }

    /**
     * Transfers all entries from current table to newTable.
     */
    void transfer(Entry[] newTable) {
        // 当前数组
        Entry[] src = table;
        // 新数组长度
        int newCapacity = newTable.length ;
        // 遍历当前数组的元素，重新计算每个元素所在数组位置
        for (int j = 0; j < src. length; j++) {
            // 取出数组中的链表第一个节点
            Entry<K,V> e = src[j];
            if (e != null) {
                // 将旧链表位置置空
                src[j] = null;
                // 循环链表，挨个将每个节点插入到新的数组位置中
                do {
                    // 取出链表中的当前节点的下一个节点
                    Entry<K,V> next = e. next;
                    // 重新计算该链表在数组中的索引位置
                    int i = indexFor(e. hash, newCapacity);
                    // 将下一个节点指向newTable[i]
                    e. next = newTable[i];
                    // 将当前节点放置在newTable[i]位置
                    newTable[i] = e;
                    // 下一次循环
                    e = next;
                } while (e != null);
            }
        }
}
```
HashTable默认的初始大小为11，之后每次扩充为原来的2n+1。HashMap默认的初始化大小为16，之后每次扩充为原来的2倍。还有我没列出代码的一点，就是如果在创建时给定了初始化大小，那么HashTable会直接使用你给定的大小，而HashMap会将其扩充为2的幂次方大小。

##  删除
```java
/**
     * 根据key删除元素
     */
    public V remove(Object key) {
        Entry<K,V> e = removeEntryForKey(key);
        return (e == null ? null : e. value);
    }

    /**
     * 根据key删除链表节点
     */
    final Entry<K,V> removeEntryForKey(Object key) {
        // 计算key的hash值
        int hash = (key == null) ? 0 : hash(key.hashCode());
        // 根据hash值计算key在数组的索引位置
        int i = indexFor(hash, table.length );
        // 找到该索引出的第一个节点
        Entry<K,V> prev = table[i];
        Entry<K,V> e = prev;

        // 遍历链表（从链表第一个节点开始next），找出相同的key，
        while (e != null) {
            Entry<K,V> next = e. next;
            Object k;
            // 如果hash值和key都相等，则认为相等
            if (e.hash == hash &&
                ((k = e. key) == key || (key != null && key.equals(k)))) {
                // 修改版本+1
                modCount++;
                // 计数器减1
                size--;
                // 如果第一个就是要删除的节点（第一个节点没有上一个节点，所以要分开判断）
                if (prev == e)
                    // 则将下一个节点放到table[i]位置（要删除的节点被覆盖）
                    table[i] = next;
                else
                 // 否则将上一个节点的next指向当要删除节点下一个（要删除节点被忽略，没有指向了）
                    prev. next = next;
                e.recordRemoval( this);
                // 返回删除的节点内容
                return e;
            }
            // 保存当前节点为下次循环的上一个节点
            prev = e;
            // 下次循环
            e = next;
        }

        return e;
}
```
## 查找
```java
public V get(Object key) {
        // 如果key等于null，则调通getForNullKey方法
        if (key == null)
            return getForNullKey();
        // 计算key对应的hash值
        int hash = hash(key.hashCode());
        // 通过hash值找到key对应数组的索引位置，遍历该数组位置的链表
        for (Entry<K,V> e = table [indexFor (hash, table .length)];
             e != null;
             e = e. next) {
            Object k;
            // 如果hash值和key都相等，则认为相等
            if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
                // 返回value
                return e.value ;
        }
        return null;
    }

    private V getForNullKey() {
        // 遍历数组第一个位置处的链表
        for (Entry<K,V> e = table [0]; e != null; e = e. next) {
            if (e.key == null)
                return e.value ;
        }
        return null;
}
```

## 是否包含 
```java
/**
     * Returns <tt>true</tt> if this map contains a mapping for the
     * specified key.
     *
     * @param   key   The key whose presence in this map is to be tested
     * @return <tt> true</tt> if this map contains a mapping for the specified
     * key.
     */
    public boolean containsKey(Object key) {
        return getEntry(key) != null;
    }
 
    /**
     * Returns the entry associated with the specified key in the
     * HashMap.  Returns null if the HashMap contains no mapping
     * for the key.
     */
    final Entry<K,V> getEntry(Object key) {
        int hash = (key == null) ? 0 : hash(key.hashCode());
        for (Entry<K,V> e = table [indexFor (hash, table .length)];
             e != null;
             e = e. next) {
            Object k;
            if (e.hash == hash &&
                ((k = e. key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
}

/**
     * Returns <tt>true</tt> if this map maps one or more keys to the
     * specified value.
     *
     * @param value value whose presence in this map is to be tested
     * @return <tt> true</tt> if this map maps one or more keys to the
     *         specified value
     */
    public boolean containsValue(Object value) {
        if (value == null)
            return containsNullValue();

       Entry[] tab = table;
       // 遍历整个table查询是否有相同的value值
        for (int i = 0; i < tab. length ; i++)
            // 遍历数组的每个链表
            for (Entry e = tab[i] ; e != null ; e = e.next)
                if (value.equals(e.value ))
                    return true;
        return false;
    }

    /**
     * Special -case code for containsValue with null argument
     */
    private boolean containsNullValue() {
       Entry[] tab = table;
        for (int i = 0; i < tab. length ; i++)
            for (Entry e = tab[i] ; e != null ; e = e.next)
                if (e.value == null)
                    return true;
        return false;
}
```
## 容量检查
```java
/**
     * Returns the number of key -value mappings in this map.
     *
     * @return the number of key- value mappings in this map
     */
    public int size() {
        return size ;
    }

    /**
     * Returns <tt>true</tt> if this map contains no key -value mappings.
     *
     * @return <tt> true</tt> if this map contains no key -value mappings
     */
    public boolean isEmpty() {
        return size == 0;
}
```
## HashMap源码
```java
package java.util;  
import java.io.*;  
 
public class HashMap<K,V>  
    extends AbstractMap<K,V>  
    implements Map<K,V>, Cloneable, Serializable  
{  
 
    // 默认的初始容量（容量为HashMap中槽的数目）是16，且实际容量必须是2的整数次幂。  
    static final int DEFAULT_INITIAL_CAPACITY = 16;  
 
    // 最大容量（必须是2的幂且小于2的30次方，传入容量过大将被这个值替换）  
    static final int MAXIMUM_CAPACITY = 1 << 30;  
 
    // 默认加载因子为0.75 
    static final float DEFAULT_LOAD_FACTOR = 0.75f;  
 
    // 存储数据的Entry数组，长度是2的幂。  
    // HashMap采用链表法解决冲突，每一个Entry本质上是一个单向链表  
    transient Entry[] table;  
 
    // HashMap的底层数组中已用槽的数量  
    transient int size;  
 
    // HashMap的阈值，用于判断是否需要调整HashMap的容量（threshold = 容量*加载因子）  
    int threshold;  
 
    // 加载因子实际大小  
    final float loadFactor;  
 
    // HashMap被改变的次数  
    transient volatile int modCount;  
 
    // 指定“容量大小”和“加载因子”的构造函数  
    public HashMap(int initialCapacity, float loadFactor) {  
        if (initialCapacity < 0)  
            throw new IllegalArgumentException("Illegal initial capacity: " +  
                                               initialCapacity);  
        // HashMap的最大容量只能是MAXIMUM_CAPACITY  
        if (initialCapacity > MAXIMUM_CAPACITY)  
            initialCapacity = MAXIMUM_CAPACITY;  
    //加载因此不能小于0
        if (loadFactor <= 0 || Float.isNaN(loadFactor))  
            throw new IllegalArgumentException("Illegal load factor: " +  
                                               loadFactor);  
 
        // 找出“大于initialCapacity”的最小的2的幂  
        int capacity = 1;  
        while (capacity < initialCapacity)  
            capacity <<= 1;  
 
        // 设置“加载因子”  
        this.loadFactor = loadFactor;  
        // 设置“HashMap阈值”，当HashMap中存储数据的数量达到threshold时，就需要将HashMap的容量加倍。  
        threshold = (int)(capacity * loadFactor);  
        // 创建Entry数组，用来保存数据  
        table = new Entry[capacity];  
        init();  
    }  
 
 
    // 指定“容量大小”的构造函数  
    public HashMap(int initialCapacity) {  
        this(initialCapacity, DEFAULT_LOAD_FACTOR);  
    }  
 
    // 默认构造函数。  
    public HashMap() {  
        // 设置“加载因子”为默认加载因子0.75  
        this.loadFactor = DEFAULT_LOAD_FACTOR;  
        // 设置“HashMap阈值”，当HashMap中存储数据的数量达到threshold时，就需要将HashMap的容量加倍。  
        threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);  
        // 创建Entry数组，用来保存数据  
        table = new Entry[DEFAULT_INITIAL_CAPACITY];  
        init();  
    }  
 
    // 包含“子Map”的构造函数  
    public HashMap(Map<? extends K, ? extends V> m) {  
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,  
                      DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);  
        // 将m中的全部元素逐个添加到HashMap中  
        putAllForCreate(m);  
    }  
 
    //求hash值的方法，重新计算hash值
    static int hash(int h) {  
        h ^= (h >>> 20) ^ (h >>> 12);  
        return h ^ (h >>> 7) ^ (h >>> 4);  
    }  
 
    // 返回h在数组中的索引值，这里用&代替取模，旨在提升效率 
    // h & (length-1)保证返回值的小于length  
    static int indexFor(int h, int length) {  
        return h & (length-1);  
    }  
 
    public int size() {  
        return size;  
    }  
 
    public boolean isEmpty() {  
        return size == 0;  
    }  
 
    // 获取key对应的value  
    public V get(Object key) {  
        if (key == null)  
            return getForNullKey();  
        // 获取key的hash值  
        int hash = hash(key.hashCode());  
        // 在“该hash值对应的链表”上查找“键值等于key”的元素  
        for (Entry<K,V> e = table[indexFor(hash, table.length)];  
             e != null;  
             e = e.next) {  
            Object k;  
      //判断key是否相同
            if (e.hash == hash && ((k = e.key) == key || key.equals(k)))  
                return e.value;  
        }
    //没找到则返回null
        return null;  
    }  
 
    // 获取“key为null”的元素的值  
    // HashMap将“key为null”的元素存储在table[0]位置，但不一定是该链表的第一个位置！  
    private V getForNullKey() {  
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {  
            if (e.key == null)  
                return e.value;  
        }  
        return null;  
    }  
 
    // HashMap是否包含key  
    public boolean containsKey(Object key) {  
        return getEntry(key) != null;  
    }  
 
    // 返回“键为key”的键值对  
    final Entry<K,V> getEntry(Object key) {  
        // 获取哈希值  
        // HashMap将“key为null”的元素存储在table[0]位置，“key不为null”的则调用hash()计算哈希值  
        int hash = (key == null) ? 0 : hash(key.hashCode());  
        // 在“该hash值对应的链表”上查找“键值等于key”的元素  
        for (Entry<K,V> e = table[indexFor(hash, table.length)];  
             e != null;  
             e = e.next) {  
            Object k;  
            if (e.hash == hash &&  
                ((k = e.key) == key || (key != null && key.equals(k))))  
                return e;  
        }  
        return null;  
    }  
 
    // 将“key-value”添加到HashMap中  
    public V put(K key, V value) {  
        // 若“key为null”，则将该键值对添加到table[0]中。  
        if (key == null)  
            return putForNullKey(value);  
        // 若“key不为null”，则计算该key的哈希值，然后将其添加到该哈希值对应的链表中。  
        int hash = hash(key.hashCode());  
        int i = indexFor(hash, table.length);  
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {  
            Object k;  
            // 若“该key”对应的键值对已经存在，则用新的value取代旧的value。然后退出！  
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {  
                V oldValue = e.value;  
                e.value = value;  
                e.recordAccess(this);  
                return oldValue;  
            }  
        }  
 
        // 若“该key”对应的键值对不存在，则将“key-value”添加到table中  
        modCount++;
    //将key-value添加到table[i]处
        addEntry(hash, key, value, i);  
        return null;  
    }  
 
    // putForNullKey()的作用是将“key为null”键值对添加到table[0]位置  
    private V putForNullKey(V value) {  
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {  
            if (e.key == null) {  
                V oldValue = e.value;  
                e.value = value;  
                e.recordAccess(this);  
                return oldValue;  
            }  
        }  
        // 如果没有存在key为null的键值对，则直接题阿见到table[0]处!  
        modCount++;  
        addEntry(0, null, value, 0);  
        return null;  
    }  
 
    // 创建HashMap对应的“添加方法”，  
    // 它和put()不同。putForCreate()是内部方法，它被构造函数等调用，用来创建HashMap  
    // 而put()是对外提供的往HashMap中添加元素的方法。  
    private void putForCreate(K key, V value) {  
        int hash = (key == null) ? 0 : hash(key.hashCode());  
        int i = indexFor(hash, table.length);  
 
        // 若该HashMap表中存在“键值等于key”的元素，则替换该元素的value值  
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {  
            Object k;  
            if (e.hash == hash &&  
                ((k = e.key) == key || (key != null && key.equals(k)))) {  
                e.value = value;  
                return;  
            }  
        }  
 
        // 若该HashMap表中不存在“键值等于key”的元素，则将该key-value添加到HashMap中  
        createEntry(hash, key, value, i);  
    }  
 
    // 将“m”中的全部元素都添加到HashMap中。  
    // 该方法被内部的构造HashMap的方法所调用。  
    private void putAllForCreate(Map<? extends K, ? extends V> m) {  
        // 利用迭代器将元素逐个添加到HashMap中  
        for (Iterator<? extends Map.Entry<? extends K, ? extends V>> i = m.entrySet().iterator(); i.hasNext(); ) {  
            Map.Entry<? extends K, ? extends V> e = i.next();  
            putForCreate(e.getKey(), e.getValue());  
        }  
    }  
 
    // 重新调整HashMap的大小，newCapacity是调整后的容量  
    void resize(int newCapacity) {  
        Entry[] oldTable = table;  
        int oldCapacity = oldTable.length; 
    //如果就容量已经达到了最大值，则不能再扩容，直接返回
        if (oldCapacity == MAXIMUM_CAPACITY) {  
            threshold = Integer.MAX_VALUE;  
            return;  
        }  
 
        // 新建一个HashMap，将“旧HashMap”的全部元素添加到“新HashMap”中，  
        // 然后，将“新HashMap”赋值给“旧HashMap”。  
        Entry[] newTable = new Entry[newCapacity];  
        transfer(newTable);  
        table = newTable;  
        threshold = (int)(newCapacity * loadFactor);  
    }  
 
    // 将HashMap中的全部元素都添加到newTable中  
    void transfer(Entry[] newTable) {  
        Entry[] src = table;  
        int newCapacity = newTable.length;  
        for (int j = 0; j < src.length; j++) {  
            Entry<K,V> e = src[j];  
            if (e != null) {  
                src[j] = null;  
                do {  
                    Entry<K,V> next = e.next;  
                    int i = indexFor(e.hash, newCapacity);  
                    e.next = newTable[i];  
                    newTable[i] = e;  
                    e = next;  
                } while (e != null);  
            }  
        }  
    }  
 
    // 将"m"的全部元素都添加到HashMap中  
    public void putAll(Map<? extends K, ? extends V> m) {  
        // 有效性判断  
        int numKeysToBeAdded = m.size();  
        if (numKeysToBeAdded == 0)  
            return;  
 
        // 计算容量是否足够，  
        // 若“当前阀值容量 < 需要的容量”，则将容量x2。  
        if (numKeysToBeAdded > threshold) {  
            int targetCapacity = (int)(numKeysToBeAdded / loadFactor + 1);  
            if (targetCapacity > MAXIMUM_CAPACITY)  
                targetCapacity = MAXIMUM_CAPACITY;  
            int newCapacity = table.length;  
            while (newCapacity < targetCapacity)  
                newCapacity <<= 1;  
            if (newCapacity > table.length)  
                resize(newCapacity);  
        }  
 
        // 通过迭代器，将“m”中的元素逐个添加到HashMap中。  
        for (Iterator<? extends Map.Entry<? extends K, ? extends V>> i = m.entrySet().iterator(); i.hasNext(); ) {  
            Map.Entry<? extends K, ? extends V> e = i.next();  
            put(e.getKey(), e.getValue());  
        }  
    }  
 
    // 删除“键为key”元素  
    public V remove(Object key) {  
        Entry<K,V> e = removeEntryForKey(key);  
        return (e == null ? null : e.value);  
    }  
 
    // 删除“键为key”的元素  
    final Entry<K,V> removeEntryForKey(Object key) {  
        // 获取哈希值。若key为null，则哈希值为0；否则调用hash()进行计算  
        int hash = (key == null) ? 0 : hash(key.hashCode());  
        int i = indexFor(hash, table.length);  
        Entry<K,V> prev = table[i];  
        Entry<K,V> e = prev;  
 
        // 删除链表中“键为key”的元素  
        // 本质是“删除单向链表中的节点”  
        while (e != null) {  
            Entry<K,V> next = e.next;  
            Object k;  
            if (e.hash == hash &&  
                ((k = e.key) == key || (key != null && key.equals(k)))) {  
                modCount++;  
                size--;  
                if (prev == e)  
                    table[i] = next;  
                else 
                    prev.next = next;  
                e.recordRemoval(this);  
                return e;  
            }  
            prev = e;  
            e = next;  
        }  
 
        return e;  
    }  
 
    // 删除“键值对”  
    final Entry<K,V> removeMapping(Object o) {  
        if (!(o instanceof Map.Entry))  
            return null;  
 
        Map.Entry<K,V> entry = (Map.Entry<K,V>) o;  
        Object key = entry.getKey();  
        int hash = (key == null) ? 0 : hash(key.hashCode());  
        int i = indexFor(hash, table.length);  
        Entry<K,V> prev = table[i];  
        Entry<K,V> e = prev;  
 
        // 删除链表中的“键值对e”  
        // 本质是“删除单向链表中的节点”  
        while (e != null) {  
            Entry<K,V> next = e.next;  
            if (e.hash == hash && e.equals(entry)) {  
                modCount++;  
                size--;  
                if (prev == e)  
                    table[i] = next;  
                else 
                    prev.next = next;  
                e.recordRemoval(this);  
                return e;  
            }  
            prev = e;  
            e = next;  
        }  
 
        return e;  
    }  
 
    // 清空HashMap，将所有的元素设为null  
    public void clear() {  
        modCount++;  
        Entry[] tab = table;  
        for (int i = 0; i < tab.length; i++)  
            tab[i] = null;  
        size = 0;  
    }  
 
    // 是否包含“值为value”的元素  
    public boolean containsValue(Object value) {  
    // 若“value为null”，则调用containsNullValue()查找  
    if (value == null)  
            return containsNullValue();  
 
    // 若“value不为null”，则查找HashMap中是否有值为value的节点。  
    Entry[] tab = table;  
        for (int i = 0; i < tab.length ; i++)  
            for (Entry e = tab[i] ; e != null ; e = e.next)  
                if (value.equals(e.value))  
                    return true;  
    return false;  
    }  
 
    // 是否包含null值  
    private boolean containsNullValue() {  
    Entry[] tab = table;  
        for (int i = 0; i < tab.length ; i++)  
            for (Entry e = tab[i] ; e != null ; e = e.next)  
                if (e.value == null)  
                    return true;  
    return false;  
    }  
 
    // 克隆一个HashMap，并返回Object对象  
    public Object clone() {  
        HashMap<K,V> result = null;  
        try {  
            result = (HashMap<K,V>)super.clone();  
        } catch (CloneNotSupportedException e) {  
            // assert false;  
        }  
        result.table = new Entry[table.length];  
        result.entrySet = null;  
        result.modCount = 0;  
        result.size = 0;  
        result.init();  
        // 调用putAllForCreate()将全部元素添加到HashMap中  
        result.putAllForCreate(this);  
 
        return result;  
    }  
 
    // Entry是单向链表。  
    // 它是 “HashMap链式存储法”对应的链表。  
    // 它实现了Map.Entry 接口，即实现getKey(), getValue(), setValue(V value), equals(Object o), hashCode()这些函数  
    static class Entry<K,V> implements Map.Entry<K,V> {  
        final K key;  
        V value;  
        // 指向下一个节点  
        Entry<K,V> next;  
        final int hash;  
 
        // 构造函数。  
        // 输入参数包括"哈希值(h)", "键(k)", "值(v)", "下一节点(n)"  
        Entry(int h, K k, V v, Entry<K,V> n) {  
            value = v;  
            next = n;  
            key = k;  
            hash = h;  
        }  
 
        public final K getKey() {  
            return key;  
        }  
 
        public final V getValue() {  
            return value;  
        }  
 
        public final V setValue(V newValue) {  
            V oldValue = value;  
            value = newValue;  
            return oldValue;  
        }  
 
        // 判断两个Entry是否相等  
        // 若两个Entry的“key”和“value”都相等，则返回true。  
        // 否则，返回false  
        public final boolean equals(Object o) {  
            if (!(o instanceof Map.Entry))  
                return false;  
            Map.Entry e = (Map.Entry)o;  
            Object k1 = getKey();  
            Object k2 = e.getKey();  
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {  
                Object v1 = getValue();  
                Object v2 = e.getValue();  
                if (v1 == v2 || (v1 != null && v1.equals(v2)))  
                    return true;  
            }  
            return false;  
        }  
 
        // 实现hashCode()  
        public final int hashCode() {  
            return (key==null   ? 0 : key.hashCode()) ^  
                   (value==null ? 0 : value.hashCode());  
        }  
 
        public final String toString() {  
            return getKey() + "=" + getValue();  
        }  
 
        // 当向HashMap中添加元素时，绘调用recordAccess()。  
        // 这里不做任何处理  
        void recordAccess(HashMap<K,V> m) {  
        }  
 
        // 当从HashMap中删除元素时，绘调用recordRemoval()。  
        // 这里不做任何处理  
        void recordRemoval(HashMap<K,V> m) {  
        }  
    }  
 
    // 新增Entry。将“key-value”插入指定位置，bucketIndex是位置索引。  
    void addEntry(int hash, K key, V value, int bucketIndex) {  
        // 保存“bucketIndex”位置的值到“e”中  
        Entry<K,V> e = table[bucketIndex];  
        // 设置“bucketIndex”位置的元素为“新Entry”，  
        // 设置“e”为“新Entry的下一个节点”  
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);  
        // 若HashMap的实际大小 不小于 “阈值”，则调整HashMap的大小  
        if (size++ >= threshold)  
            resize(2 * table.length);  
    }  
 
    // 创建Entry。将“key-value”插入指定位置。  
    void createEntry(int hash, K key, V value, int bucketIndex) {  
        // 保存“bucketIndex”位置的值到“e”中  
        Entry<K,V> e = table[bucketIndex];  
        // 设置“bucketIndex”位置的元素为“新Entry”，  
        // 设置“e”为“新Entry的下一个节点”  
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);  
        size++;  
    }  
 
    // HashIterator是HashMap迭代器的抽象出来的父类，实现了公共了函数。  
    // 它包含“key迭代器(KeyIterator)”、“Value迭代器(ValueIterator)”和“Entry迭代器(EntryIterator)”3个子类。  
    private abstract class HashIterator<E> implements Iterator<E> {  
        // 下一个元素  
        Entry<K,V> next;  
        // expectedModCount用于实现fast-fail机制。  
        int expectedModCount;  
        // 当前索引  
        int index;  
        // 当前元素  
        Entry<K,V> current;  
 
        HashIterator() {  
            expectedModCount = modCount;  
            if (size > 0) { // advance to first entry  
                Entry[] t = table;  
                // 将next指向table中第一个不为null的元素。  
                // 这里利用了index的初始值为0，从0开始依次向后遍历，直到找到不为null的元素就退出循环。  
                while (index < t.length && (next = t[index++]) == null)  
                    ;  
            }  
        }  
 
        public final boolean hasNext() {  
            return next != null;  
        }  
 
        // 获取下一个元素  
        final Entry<K,V> nextEntry() {  
            if (modCount != expectedModCount)  
                throw new ConcurrentModificationException();  
            Entry<K,V> e = next;  
            if (e == null)  
                throw new NoSuchElementException();  
 
            // 注意！！！  
            // 一个Entry就是一个单向链表  
            // 若该Entry的下一个节点不为空，就将next指向下一个节点;  
            // 否则，将next指向下一个链表(也是下一个Entry)的不为null的节点。  
            if ((next = e.next) == null) {  
                Entry[] t = table;  
                while (index < t.length && (next = t[index++]) == null)  
                    ;  
            }  
            current = e;  
            return e;  
        }  
 
        // 删除当前元素  
        public void remove() {  
            if (current == null)  
                throw new IllegalStateException();  
            if (modCount != expectedModCount)  
                throw new ConcurrentModificationException();  
            Object k = current.key;  
            current = null;  
            HashMap.this.removeEntryForKey(k);  
            expectedModCount = modCount;  
        }  
 
    }  
 
    // value的迭代器  
    private final class ValueIterator extends HashIterator<V> {  
        public V next() {  
            return nextEntry().value;  
        }  
    }  
 
    // key的迭代器  
    private final class KeyIterator extends HashIterator<K> {  
        public K next() {  
            return nextEntry().getKey();  
        }  
    }  
 
    // Entry的迭代器  
    private final class EntryIterator extends HashIterator<Map.Entry<K,V>> {  
        public Map.Entry<K,V> next() {  
            return nextEntry();  
        }  
    }  
 
    // 返回一个“key迭代器”  
    Iterator<K> newKeyIterator()   {  
        return new KeyIterator();  
    }  
    // 返回一个“value迭代器”  
    Iterator<V> newValueIterator()   {  
        return new ValueIterator();  
    }  
    // 返回一个“entry迭代器”  
    Iterator<Map.Entry<K,V>> newEntryIterator()   {  
        return new EntryIterator();  
    }  
 
    // HashMap的Entry对应的集合  
    private transient Set<Map.Entry<K,V>> entrySet = null;  
 
    // 返回“key的集合”，实际上返回一个“KeySet对象”  
    public Set<K> keySet() {  
        Set<K> ks = keySet;  
        return (ks != null ? ks : (keySet = new KeySet()));  
    }  
 
    // Key对应的集合  
    // KeySet继承于AbstractSet，说明该集合中没有重复的Key。  
    private final class KeySet extends AbstractSet<K> {  
        public Iterator<K> iterator() {  
            return newKeyIterator();  
        }  
        public int size() {  
            return size;  
        }  
        public boolean contains(Object o) {  
            return containsKey(o);  
        }  
        public boolean remove(Object o) {  
            return HashMap.this.removeEntryForKey(o) != null;  
        }  
        public void clear() {  
            HashMap.this.clear();  
        }  
    }  
 
    // 返回“value集合”，实际上返回的是一个Values对象  
    public Collection<V> values() {  
        Collection<V> vs = values;  
        return (vs != null ? vs : (values = new Values()));  
    }  
 
    // “value集合”  
    // Values继承于AbstractCollection，不同于“KeySet继承于AbstractSet”，  
    // Values中的元素能够重复。因为不同的key可以指向相同的value。  
    private final class Values extends AbstractCollection<V> {  
        public Iterator<V> iterator() {  
            return newValueIterator();  
        }  
        public int size() {  
            return size;  
        }  
        public boolean contains(Object o) {  
            return containsValue(o);  
        }  
        public void clear() {  
            HashMap.this.clear();  
        }  
    }  
 
    // 返回“HashMap的Entry集合”  
    public Set<Map.Entry<K,V>> entrySet() {  
        return entrySet0();  
    }  
 
    // 返回“HashMap的Entry集合”，它实际是返回一个EntrySet对象  
    private Set<Map.Entry<K,V>> entrySet0() {  
        Set<Map.Entry<K,V>> es = entrySet;  
        return es != null ? es : (entrySet = new EntrySet());  
    }  
 
    // EntrySet对应的集合  
    // EntrySet继承于AbstractSet，说明该集合中没有重复的EntrySet。  
    private final class EntrySet extends AbstractSet<Map.Entry<K,V>> {  
        public Iterator<Map.Entry<K,V>> iterator() {  
            return newEntryIterator();  
        }  
        public boolean contains(Object o) {  
            if (!(o instanceof Map.Entry))  
                return false;  
            Map.Entry<K,V> e = (Map.Entry<K,V>) o;  
            Entry<K,V> candidate = getEntry(e.getKey());  
            return candidate != null && candidate.equals(e);  
        }  
        public boolean remove(Object o) {  
            return removeMapping(o) != null;  
        }  
        public int size() {  
            return size;  
        }  
        public void clear() {  
            HashMap.this.clear();  
        }  
    }  
 
    // java.io.Serializable的写入函数  
    // 将HashMap的“总的容量，实际容量，所有的Entry”都写入到输出流中  
    private void writeObject(java.io.ObjectOutputStream s)  
        throws IOException  
    {  
        Iterator<Map.Entry<K,V>> i =  
            (size > 0) ? entrySet0().iterator() : null;  
 
        // Write out the threshold, loadfactor, and any hidden stuff  
        s.defaultWriteObject();  
 
        // Write out number of buckets  
        s.writeInt(table.length);  
 
        // Write out size (number of Mappings)  
        s.writeInt(size);  
 
        // Write out keys and values (alternating)  
        if (i != null) {  
            while (i.hasNext()) {  
            Map.Entry<K,V> e = i.next();  
            s.writeObject(e.getKey());  
            s.writeObject(e.getValue());  
            }  
        }  
    }  
 
 
    private static final long serialVersionUID = 362498820763181265L;  
 
    // java.io.Serializable的读取函数：根据写入方式读出  
    // 将HashMap的“总的容量，实际容量，所有的Entry”依次读出  
    private void readObject(java.io.ObjectInputStream s)  
         throws IOException, ClassNotFoundException  
    {  
        // Read in the threshold, loadfactor, and any hidden stuff  
        s.defaultReadObject();  
 
        // Read in number of buckets and allocate the bucket array;  
        int numBuckets = s.readInt();  
        table = new Entry[numBuckets];  
 
        init();  // Give subclass a chance to do its thing.  
 
        // Read in size (number of Mappings)  
        int size = s.readInt();  
 
        // Read the keys and values, and put the mappings in the HashMap  
        for (int i=0; i<size; i++) {  
            K key = (K) s.readObject();  
            V value = (V) s.readObject();  
            putForCreate(key, value);  
        }  
    }  
 
    // 返回“HashMap总的容量”  
    int   capacity()     { return table.length; }  
    // 返回“HashMap的加载因子”  
    float loadFactor()   { return loadFactor;   }  
} 
```