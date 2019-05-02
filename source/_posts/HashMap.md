---
title: HashMap
date: 2018-01-04 21:44:09
categories:
	- Java
tags:
	- Java
---
摘要：HashMap
![](/images/huakai.jpeg)
<!-- more -->
正文：

# LZ在这里总结了一些关于HashMap的面试题
## HashMap 中的 key 如果是 Object 则需要实现哪些方法？
hashCode 方法和 equals 方法。
因为 hashCode 方法用来计算 Entry 在数组中的 index 索引位置，equals 方法用来比较数组指定 index 索引位置上链表的节点 Entry 元素是否相等。否则由于 hashCode 方法实现不恰当会导致严重的 hash 碰撞，从而使 HashMap 会退化成链表结构而影响性能。

## hashmap的两种遍历方式 
`问`下面两种遍历方式有什么区别？为什么？
```
		Map<Integer, Integer> map = new HashMap<>();
		map.put(1, 2);
		map.put(2, 3);
//		Map.Entry<Integer, Integer> entry = (Entry<Integer, Integer>) map.entrySet();
		Iterator iterator = map.entrySet().iterator();
		while (iterator.hasNext()){
			System.out.println(iterator.next());
		}
```
`答`：第一种方式效率高且推荐用。  
因为 HashMap 的这两种遍历是分别对 keySet 和 entrySet 进行迭代，对于 keySet 实质上是遍历了两次，一次是转为 iterator 迭代器遍历，一次就从 HashMap 中取出 key 所对于的 value 操作（通过 key 值 hashCode 和 equals 索引）；而 entrySet 方式只遍历了一次，它把 key 和 value 都放到了 Entry 中，所以效率高。
`问`：为什么 HashMap 中 String、Integer 这样的包装类适合作为 key 键，即为什么使用它们可以减少哈希碰撞？
因为 String、Integer 等包装类是 final 类型的，具有不可变性，而且已经重写了 equals() 和 hashCode() 方法。不可变性保证了计算 hashCode() 后键值的唯一性和缓存特性，不会出现放入和获取时哈希码不同的情况且读取哈希值的高效性，此外官方实现的 equals() 和 hashCode() 都是严格遵守相关规范的，不会出现错误。
`问` 下面的程序输出结果是什么？
```
    class Item {

        public String name;

        public int age;


        @Override

        public int hashCode() {

            return this.name.hashCode() + this.age;

        }


        public Item(String name, int age) {

            this.name = name;

            this.age = age;

        }

    }


    public class Demo {

        public static void main(String[] args) {  

            Item item1 = new Item("android", 4);

            Item item2 = new Item("java", 3);

            Map<Item, Item> map = new HashMap<Item, Item>();

            map.put(item1, item1);

            map.put(item2, item2);

            item2.name = "unix c";

            Item value = map.get(item2);

            System.out.println("value="+value);

        }

    }
```
`答`： 输出结果为 value=null。  

因为 key 更新后 hashCode 也更新了，而 HashMap 里面的对象是我们原来哈希值的对象，在 get 时由于哈希值已经变了，原来的对象不会被索引到了，所以结果为 null，因此当把对象放到 HashMap 后就不要尝试对 key 进行修改操作，谨防出现哈希值变化或者 equals 比较不等的情况导致无法索引。
为什么String, Interger这样的wrapper类适合作为键？ String, Interger这样的wrapper类作为HashMap的键是再适合不过了，而且String最为常用。因为String是不可变的，也是final的，而且已经重写了equals()和hashCode()方法了。其他的wrapper类也有这个特点。不可变性是必要的，因为为了要计算hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象。不可变性还有其他的优点如线程安全。如果你可以仅仅通过将某个field声明成final就能保证hashCode是不变的，那么请这么做吧。因为获取对象的时候要用到equals()和hashCode()方法，那么键对象正确的重写这两个方法是非常重要的。如果两个不相等的对象返回不同的hashcode的话，那么碰撞的几率就会小些，这样就能提高HashMap的性能。
