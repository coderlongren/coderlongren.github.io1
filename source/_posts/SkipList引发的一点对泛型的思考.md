---
title: 实现跳跃表引发的一点对泛型的思考
date: 2018-6-17 01:43:00
categories:
	- JDK
tags:
	- JDK
	- DataStruct
	- 源码剖析
---
摘要：数组和链表的折中--跳跃表，代码重构--泛型
<!-- more -->

在redis 设计实现的源代码里面看到了作者对跳跃表的实现，作者使用C语言来写的，
自己就花了一个晚上的时间用Java把他的代码重构了一下，支持类型化插入，查找，打印。  
那么问题来了， 我们想让构建的跳跃表数据结构支持各种类型，该使用何种泛型呢？首先想到了<T>,后来一想不对啊，我以后放进去的数据可能有很多种，他们可能是T，或者是T的子类。
 又去查看了JDK对ConcurrentSkipList原生的底层实现代码，发现JDK全部都是用的<? extends T> 来对泛型加以限制。  
 我们都知道，泛型只是Java的一种语法糖，经过编译进行了类型擦除之后，最后class文件生成的都是 Object类型的容器。
 那么我们就只用<Obejct>这一种泛型不就行了吗？ 为什么还要搞麻烦的 <? extends T> and <? super T>来对泛型上下界进行约束。  
 再次放大招[importNew上的解释](http://www.importnew.com/24029.html)
 原来这里还牵涉到了，泛型容器的生产者，消费者模式。
 <? extends T> 只允许我们get容器中T型元素，相当于生产者。  
 <? super T> 则只允许我们add元素，相当于消费者。

	List<? extends Fruit> flist = new ArrayList<Fruit>();
	List<? extends Fruit> flist = new ArrayList<Apple>();
	List<? extends Fruit> flist = new ArrayList<Orange>();
    当我们尝试add一个Apple的时候，flist可能指向new ArrayList<Orange>();
    当我们尝试add一个Orange的时候，flist可能指向new ArrayList<Apple>();
    当我们尝试add一个Fruit的时候，这个Fruit可以是任何类型的Fruit，而flis可能只想某种特定类型的Fruit，编译器无法识别所以会报错。

	List<? super Apple> list = new ArrayList<Apple>();
	List<? super Apple> list = new ArrayList<Fruit>();
	List<? super Apple> list = new ArrayList<Object>();

当我们尝试通过list来get一个Apple的时候，可能会get得到一个Fruit，这个Fruit可以是Orange等其他类型的Fruit。

下面再来说跳跃表 ，这种消耗空间换取时间的数据结构
### 跳跃表的基本构成
1. 由多层结构组成；
2. 每一层是一个有序的链表，排列顺序为由高层到底层，都至少包含两个链表节点，分别是前面的head节点和后面的tail节点；
3. 最底层的链表包含了所有的元素；
4. 如果一个元素出现在某一层的链表中，那么在该层之下的链表也全都会出现；
5. 链表中的每个节点都包含四个指针，up, down, right, left

这样可以维持，添加，删除搜索节点的时间复杂度为log(n),
#### 搜索
[](http://odwv9d2u8.bkt.clouddn.com/17-10-3/6620991.jpg)
```java
public SkipListNode<T> search(int key) {
		SkipListNode<T> p = findNode(key);
		
        if (key == p.getKey()) {
            return p;
        } else {
            return null;
        }
	}
```
`源代码` 加入了很详细的注释
#### 跳跃表节点结构
```java
package DataStruct;

import org.hamcrest.core.IsInstanceOf;

/**
* @author 作者 : coderlong
* @version 创建时间：2018年6月16日 晚上12:57:15
* 类说明:  跳跃表的节点类， 包含了key-value left, right,up,down 四个指针
*  借鉴了 https://blog.csdn.net/BrilliantEagle/article/details/52206261 的code
*/
public class SkipListNode<T> {
	public int key;
	public T value;
	public SkipListNode<T> up, down, left, right; // 上下左右四个指针
	public static final int HEAD_KEY = Integer.MIN_VALUE;
	public static final int TAIL_KEY = Integer.MAX_VALUE;
	// 构造函数
	public SkipListNode(int k, T V) {
		this.key = k;
		this.value = V;
	}
	public int getKey() {
		return key;
	}
	public void setKey(int key) {
		this.key = key;
	}
	public T getValue() {
		return value;
	}
	public void setValue(T value) {
		this.value = value;
	}
	// 重写了 equals() 用于比较
	@Override
	public boolean equals(Object obj) {
		if (this == obj) {
			return true;
		}
		if (obj == null) {
			return false;
		}
		if (!(obj instanceof SkipListNode<?>)) {
			return false;
		}
		SkipListNode<T> other = (SkipListNode<T>)obj; // 强转类型
		try{
			
		}
		catch (Exception e) {
			// TODO: handle exception
			System.err.println(e.toString());
		}
		return other.key == key && other.value == value;
		
	}
	@Override
	public String toString() {
		return "Key-Value: " + key + "-" + value;
	}

}

```
#### SkipList结构主要实现

```java
package DataStruct;
/**
* @author 作者 : coderlong
* @version 创建时间：2018年6月17日 下午11:32:42
* 类说明: 随机层数的额 跳跃表
*/

import java.util.Random;

class SkipList<T>{
	
	// 头尾指针
	private SkipListNode<T> head, tail;
	private int nodes; // 节点数
	private int listLevel; // 层数
	private static final double PROBABILITY=0.5;//向上提升一个的概率
	private Random random;
	public SkipList() {
		random = new Random();
		clear(); // 每一次创建一个新的跳跃表的时候，都要clear一下
	}
	
	// 清空跳跃表
	public void clear() {
		head = new SkipListNode<T>(SkipListNode.HEAD_KEY, null);
		tail = new SkipListNode<T>(SkipListNode.TAIL_KEY, null);
		
		listLevel = 0; // 层数为0
		nodes = 0; // 节点数为0
		horizontalLink(head, tail);
	}
	public boolean isEmpty() {
		return nodes == 0; // 节点数是否为0
	}
	public int size() {
		return nodes; // 返回节点总数
	}
	
	// 寻找节点
	private SkipListNode<T> findNode(int key) {
		SkipListNode<T> p = head; // 从头结点开始查找
		while (true) {
			while (p.right.key != SkipListNode.TAIL_KEY && p.right.key <= key) {
				p = p.right; // 往右 搜索
			}
			if (p.down != null) {
				p = p.down; // 往下一层 试探搜索
			}
			else {
				break;
			}
		}
		return p;
	}
	// 查找是否存储key, 存在返回该节点，否则返回null
	public SkipListNode<T> search(int key) {
		SkipListNode<T> p = findNode(key);
		
        if (key == p.getKey()) {
            return p;
        } else {
            return null;
        }
	}
	
	// 向跳跃表中增加 key-value对
	public void put(int k, T v) {
		// 先试探搜索，表中是否含有该key的节点, 有则直接替换v， 即可
		SkipListNode<T> p = findNode(k);
		if (k == p.getKey()) {
			p.value = v;
			return;
		}
		// 创建新节点插入到 跳跃表中
		SkipListNode<T> newNode = new SkipListNode<T>(k, v);
		backlink(p, newNode);
		int currentLevel=0; //当前所在的层级是0
		// 随机选择 插入的节点是否上升
		while (random.nextDouble() < PROBABILITY) {
			//如果超出了高度，需要重新建一个顶层
            if (currentLevel >= listLevel) {
            	
                listLevel++;
                SkipListNode<T> p1=new SkipListNode<T>(SkipListNode.HEAD_KEY, null);
                SkipListNode<T> p2=new SkipListNode<T>(SkipListNode.TAIL_KEY, null);
                horizontalLink(p1, p2);
                vertiacallLink(p1, head);
                vertiacallLink(p2, tail);
                
                head=p1;
                tail=p2;
            }
            //将p移动到上一层
            while (p.up==null) {
                p=p.left;
            }
            p=p.up;

            SkipListNode<T> e=new SkipListNode<T>(k, null);//只保存key就ok
            backlink(p, e);//将e插入到p的后面
            vertiacallLink(e, newNode);//将e和q上下连接
            newNode=e;
            currentLevel++;
		}
		nodes++;
		
	}
	
	// 水平双向连接两个节点
	private void horizontalLink(SkipListNode<T> a, SkipListNode<T> b) {
		a.right = b;
		b.left = a; 
	}
	// 我们要把after节点插入到before节点之后
	// 这不正是我们学过的双向链表的 节点插入方法吗？
	private void backlink(SkipListNode<T> before, SkipListNode<T> after) {
		after.left = before;
		after.right = before.right;
		before.right.left = after;
		before.right = after;
	}
	
	
	// 垂直连接两个节点
	private void vertiacallLink(SkipListNode<T> node1,SkipListNode<T> node2){
        node1.down=node2;
        node2.up=node1;
    }
	
	// 打印出 跳跃表
	@Override
	public String toString() {
		if (isEmpty()) {
            return "跳跃表为空！";
        }
        StringBuilder builder=new StringBuilder();
        SkipListNode<T> p=head;
        while (p.down!=null) {
            p=p.down;
        }

        while (p.left!=null) {
            p=p.left;
        }
        if (p.right!=null) {
            p=p.right;
        }
        while (p.right!=null) {
            builder.append(p);
            builder.append("\n");
            p=p.right;
        }

        return builder.toString();
	}
	
}

```
#### 测试程序
```java
public static void main(String[] args) {
	// TODO Auto-generated method stub
	SkipList<String> list=new SkipList<String>();
    System.out.println(list);
    list.put(1, "coderlong");
    list.put(2, "yake");
    list.put(3, "sbsb");
    list.put(1, "new coderlong");
    System.out.println(list);
    System.out.println(list.size());
    
}
```
结果为:
	跳跃表为空！
	Key-Value: 1-new coderlong
	Key-Value: 2-yake
	Key-Value: 3-sbsb

	3
第一节点被覆盖。

