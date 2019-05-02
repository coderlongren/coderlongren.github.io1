---
title: Iterator&Collection源码剖析
date: 2018-1-18 14:18:40
categories:
	- JDK
tags:
	- JDK
	- 代码重构
	- 源码学习
	- 设计模式
---

1. 统一接口，
ArrayList的定义
```java
public class ArrayList<E> extends AbstractList<E>
       implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

2. LinkedList的定义
```java
 public class LinkedList<E>
           extends AbstractSequentialList<E>
           implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```
从定义上可以看到ArrayList和LinkedList都实现了List接口，ok看下List接口的定义
```java
public interface List<E> extends Collection<E> {
         int size();
         boolean isEmpty();
         boolean contains(Object o);
         Iterator<E> iterator();
         Object[] toArray();
         <T> T[] toArray(T[] a);
         boolean add(E e);
         boolean remove(Object o);
         boolean containsAll(Collection<?> c);
         boolean addAll(Collection<? extends E> c);
         boolean addAll( int index, Collection<? extends E> c);
         boolean removeAll(Collection<?> c);
         boolean retainAll(Collection<?> c);
         void clear();
         boolean equals(Object o);
         int hashCode();
         E get( int index);
         E set( int index, E element);
         void add( int index, E element);
         E remove( int index);
         int indexOf(Object o);
         int lastIndexOf(Object o);
         ListIterator<E> listIterator();
         ListIterator<E> listIterator( int index);
         List<E> subList( int fromIndex, int toIndex);
}
```
可以看到List中对容器的各种操作add、remove、set、get、size等进行了统一定义，同时List实现了Collection接口，继续看下Collection接口的定义（先不关心Iterator）：
```java
public interface Collection<E> extends Iterable<E> {
          int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean remove(Object o);
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean removeAll(Collection<?> c);
    boolean retainAll(Collection<?> c);
    void clear();
    boolean equals(Object o);
    int hashCode();
}
```
有了这两个接口，对于ArrayList和LinkeList的操作是不是就可以这么写了呢？
```java
1 Collection<String> collection = new ArrayList<String>();
2 collection.add("hello");
3 collection.add("java");
4 collection.remove("hello");

// 对于ArrayList的实现不满意，ok换成LinkedList实现，
1 Collection<String> collection = new LinkedList<String>();
2 collection.add("hello");
3 collection.add("java");
4 collection.remove("hello");
```
## Iterator设计思想
需要统一一种遍历方式，提供一个统计遍历接口，而具体实现需要具体的容器自己根据自身数据结构特点来完成，而对于用户就只有一个接口而已，万年不变。于是就有了Iterator，接下来看下Iterator的实现吧。。。
     先回过头去看Collection的定义，发现Collection 实现了一个Iterable 接口（早就发现了好嘛？），来看下的Iterable的定义，

```java
/** Implementing this interface allows an object to be the target of
 *  the "foreach" statement.
 * @since 1.5
 */
public interface Iterable<T> {

    /**
     * Returns an iterator over a set of elements of type T.
     *
     * @return an Iterator.
     */
    Iterator<T> iterator();
}
```
英文注释说，实现iterable接口的类可以使用“foreach”操作，然后并要求实现Iterator<T> iterator()方法，该方法返回一个Iterator接口对象，来看下Iterator接口，
```java
public interface Iterator<E> {
    // 是否还有元素
    boolean hasNext();
    // 下一个元素
    E next(); 
    // 将迭代器返回的元素删除
    void remove();
}
```
Iterator接口一共有三个方法，通过这三个方法就可以实现通用遍历了，ok
```java
Collection<String> collection = new ArrayList<String>();
collection.add("hello");
collection.add("java");

Iterator<String> iterator = collection.iterator();
while (iterator.hasNext()) {
     System. out.println(iterator.next());
}
```
用户再也不用关心容器的底层实现了，不管你是数组还是链表还是其他什么，只需要用Iterator就可以进行遍历了。
     
到这里Iterator的设计原则和思路实际上已经讲完了，我们来整理下Iterator模式的几个角色：

1. 迭代器Iterator接口。
2. 迭代器Iterator的实现类，要求重写hasNext()，next()，remove()三个方法 。
3. 容器统一接口，要求有一个返回Iterator接口的方法。
4. 容器实现类，实现一个返回Iterator接口的方法。

---

## 其实到了这，我们完全可以自己实现一个iteretor模型，既然有现成的JDK，那就算了
ArrayList的Iterator实现
```
public Iterator<E> iterator() {
    return new Itr();
}
```
和自己实现的一样，采用 *内部类* 实现 Iterator接口。
```java
private class Itr implements Iterator<E> {
       // 将要next返回元素的索引
        int cursor = 0;
       // 当前返回的元素的索引，初始值-1
        int lastRet = -1;

        /**
        * The modCount value that the iterator believes that the backing
        * List should have.  If this expectation is violated, the iterator
        * has detected concurrent modification.
        */
        int expectedModCount = modCount;

        public boolean hasNext() {
            // 由于cursor是将要返回元素的索引，也就是下一个元素的索引，和size比较是否相等，也就是判断是否已经next到最后一个元素
            return cursor != size();
       }

        public E next() {
            checkForComodification();
           try {
              // 根据下一个元素索引取出对应元素
              E next = get( cursor);
              // 更新lastRet为当前元素的索引，cursor加1
               lastRet = cursor ++;
              // 返回元素
               return next;
           } catch (IndexOutOfBoundsException e) {
              checkForComodification();
               throw new NoSuchElementException();
           }
       }

        public void remove() {
           // remove前必须先next一下，先取得当前元素
           if (lastRet == -1)
               throw new IllegalStateException();
            checkForComodification();

           try {
              AbstractList. this.remove(lastRet );
               // 确保lastRet比cursor小、理论上永远lastRet比cursor小1
               if (lastRet < cursor)
                  // 由于删除了一个元素cursor回退1
                  cursor--;
               // 重置为-1
               lastRet = -1;
               expectedModCount = modCount ;
           } catch (IndexOutOfBoundsException e) {
               throw new ConcurrentModificationException();
           }
       }

        final void checkForComodification() {
           if (modCount != expectedModCount)
               throw new ConcurrentModificationException();
       }
}
```
ArrayList的Iterator实现起来和我们自己实现的大同小异，很好理解。  
有一点需要指出的是这里的checkForComodification 检查，之前ArrayList中留了个悬念，modCount这个变量到底是做什么的，这里简单解释一下：迭代器Iterator允许在容器遍历的时候对元素进行删除，但是有一点问题那就是当多线程操作容器的时候，在用Iterator对容器遍历的同时，其他线程可能已经改变了该容器的内容（add、remove等操作），所以每次对容器内容的更改操作（add、remove等）都会对modCount+1，这样相当于乐观锁的版本号+1，当在Iterator遍历中remove时，检查到modCount发生了变化，则马上抛出异常ConcurrentModificationException，这就是Java容器中的“fail-fast”机制。			
## .LinkedList的Iterator实现
 
 LinkedList的Iterator实现定义在AbstracSequentialtList中，实现在AbstractList中，看一下：
     AbstracSequentialtList的定义：
 ```
  public Iterator<E> iterator() {
         return listIterator();
     }
 ```
 AbstractList的实现：
```java
public ListIterator<E> listIterator() {
        return listIterator(0);
    }
   public ListIterator<E> listIterator(final int index) {
        if (index<0 || index>size())
         throw new IndexOutOfBoundsException( "Index: "+index);

        return new ListItr(index);
    }
```
ListItr的实现
```java
private class ListItr implements ListIterator<E> {
        // 最后一次返回的节点，默认位header节点
        private Entry<E> lastReturned = header;
        // 将要返回的节点
        private Entry<E> next ;
        // 将要返回的节点index索引
        private int nextIndex;
        private int expectedModCount = modCount;

       ListItr( int index) {
           // 索引越界检查
           if (index < 0 || index > size)
               throw new IndexOutOfBoundsException( "Index: "+index+
                                             ", Size: "+size );
           // 简单二分，判断遍历的方向
           if (index < (size >> 1)) {
               // 取得index位置对应的节点
               next = header .next;
               for (nextIndex =0; nextIndex<index; nextIndex++)
                  next = next .next;
           } else {
               next = header ;
               for (nextIndex =size; nextIndex>index; nextIndex --)
                  next = next .previous;
           }
       }

        public boolean hasNext() {
           // 根据下一个节点index是否等于size，判断是否有下一个节点
           return nextIndex != size;
       }

        public E next() {
           checkForComodification();
           // 遍历完成
           if (nextIndex == size)
               throw new NoSuchElementException();

           // 赋值最近一次返回的节点
           lastReturned = next ;
           // 赋值下一次要返回的节点（next后移）
           next = next .next;
           // 将要返回的节点index索引+1
           nextIndex++;
           return lastReturned .element;
       }

        public boolean hasPrevious() {
           return nextIndex != 0;
       }

       //  返回上一个节点（双向循环链表嘛、可以两个方向遍历）
        public E previous() {
           if (nextIndex == 0)
               throw new NoSuchElementException();

           lastReturned = next = next. previous;
           nextIndex--;
           checkForComodification();
           return lastReturned .element;
       }

        public int nextIndex() {
           return nextIndex ;
       }

        public int previousIndex() {
           return nextIndex -1;
       }

        public void remove() {
            checkForComodification();
            // 取出当前返回节点的下一个节点
            Entry<E> lastNext = lastReturned.next ;
            try {
                LinkedList. this.remove(lastReturned );
            } catch (NoSuchElementException e) {
                throw new IllegalStateException();
            }
           // 确认下次要返回的节点不是当前节点，如果是则修正
           if (next ==lastReturned)
                next = lastNext;
            else
             // 由于删除了一个节点，下次要返回的节点索引-1
               nextIndex--;
           // 重置lastReturned为header节点
           lastReturned = header ;
           expectedModCount++;
       }

        public void set(E e) {
           if (lastReturned == header)
               throw new IllegalStateException();
           checkForComodification();
           lastReturned.element = e;
       }

        public void add(E e) {
           checkForComodification();
           lastReturned = header ;
           addBefore(e, next);
           nextIndex++;
           expectedModCount++;
       }

        final void checkForComodification() {
           if (modCount != expectedModCount)
               throw new ConcurrentModificationException();
       }
}
```
