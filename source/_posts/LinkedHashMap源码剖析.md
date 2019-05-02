---
title: LinkedHashMap源码剖析
date: 2018-3-8 23:18:40
categories:
	- JDK
tags:
	- JDK
	- 代码重构
	- 源码学习
---

前面分析了HashMap的实现，我们知道其底层数据存储是一个hash表（数组+单向链表）。接下来我们看一下另一个LinkedHashMap，它是HashMap的一个子类，他在HashMap的基础上维持了一个双向链表（hash表+双向链表），在遍历的时候可以使用插入顺序（先进先出，类似于FIFO），或者是最近最少使用（LRU）的顺序。
     来具体看下LinkedHashMap的实现。

	public class LinkedHashMap<K,V>
	    extends HashMap<K,V>
	    implements Map<K,V>


## 底层存储
LinkedHashMap是基于HashMap，并在其基础上维持了一个双向链表，也就是说LinkedHashMap是一个hash表（数组+单向链表） +双向链表的实现
```java

/**
     * The head of the doubly linked list.
     */
    private transient Entry<K,V> header ;

    /**
     * The iteration ordering method for this linked hash map: <tt>true</tt>
     * for access -order, <tt> false</tt> for insertion -order.
     *
     * @serial
     */
    private final boolean accessOrder;
```

英文注释很详细：
head就是这个双向链表的头结点  
accessOrder 为true表示最近较少使用顺序(可以作为一个天生的LRU队列)，false表示插入顺序

继续看看 LinkedHashMap的Entry源码
```java
/**
     * LinkedHashMap entry.
     */
    private static class Entry<K,V> extends HashMap.Entry<K,V> {
        // These fields comprise the doubly linked list used for iteration.
        // 双向链表的上一个节点before和下一个节点after
        Entry<K,V> before, after ;

       // 构造方法直接调用父类HashMap的构造方法（super）
       Entry( int hash, K key, V value, HashMap.Entry<K,V> next) {
            super(hash, key, value, next);
        }

        /**
         * 从链表中删除当前节点的方法
         */
        private void remove() {
            // 改变当前节点前后两个节点的引用关系，当前节点没有被引用后，gc可以回收
            // 将上一个节点的after指向下一个节点
            before.after = after;
            // 将下一个节点的before指向前一个节点
            after.before = before;
        }

        /**
         * 在指定的节点前加入一个节点到链表中（也就是加入到链表尾部）
         */
        private void addBefore(Entry<K,V> existingEntry) {
            // 下面改变自己对前后的指向
            // 将当前节点的after指向给定的节点（加入到existingEntry前面嘛）
            after  = existingEntry;
            // 将当前节点的before指向给定节点的上一个节点
            before = existingEntry.before ;

            // 下面改变前后最自己的指向
            // 上一个节点的after指向自己
            before.after = this;
            // 下一个几点的before指向自己
            after.before = this;
        }

        // 当向Map中获取查询元素或修改元素（put相同key）的时候调用这个方法
        void recordAccess(HashMap<K,V> m) {
            LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
            // 如果accessOrder为true，也就是使用最近较少使用顺序
            if (lm.accessOrder ) {
                lm. modCount++;
                // 先删除，再添加，也就相当于移动了
                // 删除当前元素
                remove();
                // 将当前元素加入到header前（也就是链表尾部）
                addBefore(lm. header);
            }
        }

        // 当从Map中删除元素的时候调动这个方法
        void recordRemoval(HashMap<K,V> m) {
            remove();
        }
}
```
Entry继承了 HashMap.Entry, 但是自身还有before after指针，这就是双向链表 + 单相链表了.
![LinkedHashMap](https://images2015.cnblogs.com/blog/681047/201512/681047-20151219185808288-1330656739.png)

## 构造方法
```java
/**
     * 构造一个指定初始容量和加载因子的LinkedHashMap，默认accessOrder为false
     */
    public LinkedHashMap( int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }

    /**
     * 构造一个指定初始容量的LinkedHashMap，默认accessOrder为false
     */
    public LinkedHashMap( int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }

    /**
     * 构造一个使用默认初始容量(16)和默认加载因子(0.75)的LinkedHashMap，默认accessOrder为false
     */
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }

    /**
     * 构造一个指定map的LinkedHashMap，所创建LinkedHashMap使用默认加载因子(0.75)和足以容纳指定map的初始容量，默认accessOrder为false 。
     */
    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super(m);
        accessOrder = false;
    }

    /**
     * 构造一个指定初始容量、加载因子和accessOrder的LinkedHashMap
     */
    public LinkedHashMap( int initialCapacity,
                      float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
}
```

```java
/**
     * Called by superclass constructors and pseudoconstructors (clone,
     * readObject) before any entries are inserted into the map.  Initializes
     * the chain.
     */
    void init() {
        // 初始化话header，将hash设置为-1，key、value、next设置为null
        header = new Entry<K,V>(-1, null, null, null);
        // header的before和after都指向header自身
        header.before = header. after = header ; // 双向链表，就是这样创建头结点的
```

## put()
```java
/**
     * This override alters behavior of superclass put method. It causes newly
     * allocated entry to get inserted at the end of the linked list and
     * removes the eldest entry if appropriate.
     */
    void addEntry( int hash, K key, V value, int bucketIndex) {
        // 调用createEntry方法创建一个新的节点
        createEntry(hash, key, value, bucketIndex);

        // Remove eldest entry if instructed, else grow capacity if appropriate
        // 取出header后的第一个节点（因为header不保存数据，所以取header后的第一个节点）
        Entry<K,V> eldest = header.after ;
        // 判断是容量不够了是要删除第一个节点还是需要扩容
        if (removeEldestEntry(eldest)) {
            // 删除第一个节点（可实现FIFO、LRU策略的Cache）
            removeEntryForKey(eldest. key);
        } else {
            // 和HashMap一样进行扩容
            if (size >= threshold)
                resize(2 * table.length );
        }
    }

    /**
     * This override differs from addEntry in that it doesn't resize the
     * table or remove the eldest entry.
     */
    void createEntry( int hash, K key, V value, int bucketIndex) {
        // 下面三行代码的逻辑是，创建一个新节点放到单向链表的头部
        // 取出数组bucketIndex位置的旧节点 
        HashMap.Entry<K,V> old = table[bucketIndex];
        // 创建一个新的节点，并将next指向旧节点
       Entry<K,V> e = new Entry<K,V>(hash, key, value, old);
        // 将新创建的节点放到数组的bucketIndex位置
        table[bucketIndex] = e;

        // 维护双向链表，将新节点添加在双向链表header前面（链表尾部）
        e.addBefore( header);
        // 计数器size加1
        size++;
    }

    /**
     * 默认返回false，也就是不会进行元素删除了。如果想实现cache功能，只需重写该方法
     */
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
}
```

## 最后附上 LinkedHashMap的全部源码 
```java
package java.util;
import java.io.*;


public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{

    private static final long serialVersionUID = 3801124242820219131L;

    //双向循环链表的头结点，整个LinkedHa只哟shMap中只有一个header，
    //它将哈希表中所有的Entry贯穿起来，header中不保存key-value对，只保存前后节点的引用
    private transient Entry<K,V> header;

    //双向链表中元素排序规则的标志位。
    //accessOrder为false，表示按插入顺序排序
    //accessOrder为true，表示按访问顺序排序
    private final boolean accessOrder;

    //调用HashMap的构造方法来构造底层的数组
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;    //链表中的元素默认按照插入顺序排序
    }

    //加载因子取默认的0.75f
    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }

    //加载因子取默认的0.75f，容量取默认的16
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }

    //含有子Map的构造方法，同样调用HashMap的对应的构造方法
    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super(m);
        accessOrder = false;
    }

    //该构造方法可以指定链表中的元素排序的规则
    public LinkedHashMap(int initialCapacity,float loadFactor,boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }

    //覆写父类的init()方法（HashMap中的init方法为空），
    //该方法在父类的构造方法和Clone、readObject中在插入元素前被调用，
    //初始化一个空的双向循环链表，头结点中不保存数据，头结点的下一个节点才开始保存数据。
    void init() {
        header = new Entry<K,V>(-1, null, null, null);
        header.before = header.after = header;
    }


    //覆写HashMap中的transfer方法，它在父类的resize方法中被调用，
    //扩容后，将key-value对重新映射到新的newTable中
    //覆写该方法的目的是为了提高复制的效率，
    //这里充分利用双向循环链表的特点进行迭代，不用对底层的数组进行for循环。
    void transfer(HashMap.Entry[] newTable) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e = header.after; e != header; e = e.after) {
            int index = indexFor(e.hash, newCapacity);
            e.next = newTable[index];
            newTable[index] = e;
        }
    }


    //覆写HashMap中的containsValue方法，
    //覆写该方法的目的同样是为了提高查询的效率，
    //利用双向循环链表的特点进行查询，少了对数组的外层for循环
    public boolean containsValue(Object value) {
        // Overridden to take advantage of faster iterator
        if (value==null) {
            for (Entry e = header.after; e != header; e = e.after)
                if (e.value==null)
                    return true;
        } else {
            for (Entry e = header.after; e != header; e = e.after)
                if (value.equals(e.value))
                    return true;
        }
        return false;
    }


    //覆写HashMap中的get方法，通过getEntry方法获取Entry对象。
    //注意这里的recordAccess方法，
    //如果链表中元素的排序规则是按照插入的先后顺序排序的话，该方法什么也不做，
    //如果链表中元素的排序规则是按照访问的先后顺序排序的话，则将e移到链表的末尾处。
    public V get(Object key) {
        Entry<K,V> e = (Entry<K,V>)getEntry(key);
        if (e == null)
            return null;
        e.recordAccess(this);
        return e.value;
    }

    //清空HashMap，并将双向链表还原为只有头结点的空链表
    public void clear() {
        super.clear();
        header.before = header.after = header;
    }

    //Enty的数据结构，多了两个指向前后节点的引用
    private static class Entry<K,V> extends HashMap.Entry<K,V> {
        // These fields comprise the doubly linked list used for iteration.
        Entry<K,V> before, after;

        //调用父类的构造方法
        Entry(int hash, K key, V value, HashMap.Entry<K,V> next) {
            super(hash, key, value, next);
        }

        //双向循环链表中，删除当前的Entry
        private void remove() {
            before.after = after;
            after.before = before;
        }

        //双向循环立链表中，将当前的Entry插入到existingEntry的前面
        private void addBefore(Entry<K,V> existingEntry) {
            after  = existingEntry;
            before = existingEntry.before;
            before.after = this;
            after.before = this;
        }


        //覆写HashMap中的recordAccess方法（HashMap中该方法为空），
        //当调用父类的put方法，在发现插入的key已经存在时，会调用该方法，
        //调用LinkedHashmap覆写的get方法时，也会调用到该方法，
        //该方法提供了LRU算法的实现，它将最近使用的Entry放到双向循环链表的尾部，
        //accessOrder为true时，get方法会调用recordAccess方法
        //put方法在覆盖key-value对时也会调用recordAccess方法
        //它们导致Entry最近使用，因此将其移到双向链表的末尾
        void recordAccess(HashMap<K,V> m) {
            LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
            //如果链表中元素按照访问顺序排序，则将当前访问的Entry移到双向循环链表的尾部，
            //如果是按照插入的先后顺序排序，则不做任何事情。
            if (lm.accessOrder) {
                lm.modCount++;
                //移除当前访问的Entry
                remove();
                //将当前访问的Entry插入到链表的尾部
                addBefore(lm.header);
            }
        }

        void recordRemoval(HashMap<K,V> m) {
            remove();
        }
    }

    //迭代器
    private abstract class LinkedHashIterator<T> implements Iterator<T> {
    Entry<K,V> nextEntry    = header.after;
    Entry<K,V> lastReturned = null;

    /**
     * The modCount value that the iterator believes that the backing
     * List should have.  If this expectation is violated, the iterator
     * has detected concurrent modification.
     */
    int expectedModCount = modCount;

    public boolean hasNext() {
            return nextEntry != header;
    }

    public void remove() {
        if (lastReturned == null)
        throw new IllegalStateException();
        if (modCount != expectedModCount)
        throw new ConcurrentModificationException();

            LinkedHashMap.this.remove(lastReturned.key);
            lastReturned = null;
            expectedModCount = modCount;
    }

    //从head的下一个节点开始迭代
    Entry<K,V> nextEntry() {
        if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
            if (nextEntry == header)
                throw new NoSuchElementException();

            Entry<K,V> e = lastReturned = nextEntry;
            nextEntry = e.after;
            return e;
    }
    }

    //key迭代器
    private class KeyIterator extends LinkedHashIterator<K> {
    public K next() { return nextEntry().getKey(); }
    }

    //value迭代器
    private class ValueIterator extends LinkedHashIterator<V> {
    public V next() { return nextEntry().value; }
    }

    //Entry迭代器
    private class EntryIterator extends LinkedHashIterator<Map.Entry<K,V>> {
    public Map.Entry<K,V> next() { return nextEntry(); }
    }

    // These Overrides alter the behavior of superclass view iterator() methods
    Iterator<K> newKeyIterator()   { return new KeyIterator();   }
    Iterator<V> newValueIterator() { return new ValueIterator(); }
    Iterator<Map.Entry<K,V>> newEntryIterator() { return new EntryIterator(); }


    //覆写HashMap中的addEntry方法，LinkedHashmap并没有覆写HashMap中的put方法，
    //而是覆写了put方法所调用的addEntry方法和recordAccess方法，
    //put方法在插入的key已存在的情况下，会调用recordAccess方法，
    //在插入的key不存在的情况下，要调用addEntry插入新的Entry
    void addEntry(int hash, K key, V value, int bucketIndex) {
        //创建新的Entry，并插入到LinkedHashMap中
        createEntry(hash, key, value, bucketIndex);

        //双向链表的第一个有效节点（header后的那个节点）为近期最少使用的节点
        Entry<K,V> eldest = header.after;
        //如果有必要，则删除掉该近期最少使用的节点，
        //这要看对removeEldestEntry的覆写,由于默认为false，因此默认是不做任何处理的。
        if (removeEldestEntry(eldest)) {
            removeEntryForKey(eldest.key);
        } else {
            //扩容到原来的2倍
            if (size >= threshold)
                resize(2 * table.length);
        }
    }

    void createEntry(int hash, K key, V value, int bucketIndex) {
        //创建新的Entry，并将其插入到数组对应槽的单链表的头结点处，这点与HashMap中相同
        HashMap.Entry<K,V> old = table[bucketIndex];
        Entry<K,V> e = new Entry<K,V>(hash, key, value, old);
        table[bucketIndex] = e;
        //每次插入Entry时，都将其移到双向链表的尾部，
        //这便会按照Entry插入LinkedHashMap的先后顺序来迭代元素，
        //同时，新put进来的Entry是最近访问的Entry，把其放在链表末尾 ，符合LRU算法的实现
        e.addBefore(header);
        size++;
    }

    //该方法是用来被覆写的，一般如果用LinkedHashmap实现LRU算法，就要覆写该方法，
    //比如可以将该方法覆写为如果设定的内存已满，则返回true，这样当再次向LinkedHashMap中put
    //Entry时，在调用的addEntry方法中便会将近期最少使用的节点删除掉（header后的那个节点）。
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }
}

```

