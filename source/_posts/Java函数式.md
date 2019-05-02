---
title: Java函数式接口与流
date: 2018-11-5 23:18:40
categories:
	- Java
tags:
	- Lambda
	- 函数式
---
摘要：Java8函数式的一点总结
<!-- more -->
### 函数式接口
在我们所知的Java方法里，所有的方法参数都有固定的类型，不管是 int 集合，亦或者Object， 都是有上限下限的类型，但是在Java8里，函数式接口被提了上来。
也就是Lambda, 如果你熟悉Python，JS的话，对这个肯定不陌生啦，Python本身就内置了lambda
那么Java的Lambda呢？  
```java
public interface ActionListener extends Eventlisner {
	public void actionPerformed(ActionEvent event);
}
```
如果说ActionListener代表了一个抽象方法的话，那么actionPerformed有代表什么呢？这就是函数式接口了。
Java内置了一个非常有用而且非常重要的函数接口，学会他们，在Java8的高级集合等方面大有裨益。 
```java 
predicate<T>  表示一个推测
Consumer<T> 表示一个消费行为
Function<T, R> 从一个类型转换到另一个类型
Supplier<T>  工厂方法
UnaryOperator<T> 逻辑运算或许会用到
BinaryOperator<T> 运算功能
```
两个函数式示例代码：  
```java
BinaryOperator<Integer> addAction = (a, b) -> a + b;
System.out.println(addAction.apply(1,2)); // 3
Predicate<Integer> testNum = x -> x > 5;
System.out.println(testNum.test(6)); // true
``

后面的作用是我能想到的，只代表了函数式接口诸多作用的一种吧。就比如Function 流式Map等等都可以用到，强大无疑了。

##常用的流操作
前面说了下函数式重要，下面写一点Java流式操作的小例子。
### collect(toList)
Stream的of方法可以用一组值，生成新的Stream，  
collect这是一个`及早求值`的操作。
```java
List<String> collected = Stream.of("a", "bD", "cc").collect(Collectors.toList());
```
这段代码展示了用collect方法从Stream中生成一个列表对象.  

由于很多Stream操作都是惰性求值， 因此在调用一系列的方法之后，都要跟一个collect()及早求值方法。
### map 
如果一个函数可以将一类型的值转换为另一种类型，Map操作就可以实现，使用它可以把一个流中的值转换为另一种类型值得流。
```java
List<String> collected = Stream.of("a", "bD", "cc").collect(Collectors.toList());
List<String> upperList = new ArrayList<>();
upperList = collected.stream().map(s -> s.toUpperCase()).collect(Collectors.toList());
```

### filter
遍历检查集合中元素，就可以使用filter方法
`List<String> isSafeList = collected.stream().filter(s -> safe(s.charAt(0))).collect(Collectors.toList());`

很简单是不是

### flatMap
flatmap方法可以用stream替换值，然后把多个stream连接成一个新的流
前面的map，用一个新的值来替换stream中的值，后者将其连接起来。
先上一段代码，
```java
List<Artist> list = new ArrayList<>();
Artist a1 = new Artist("a", null, "");
Artist a2 = new Artist("b", null, "");
list.add(a1);
list.add(a2);
List<Artist> list2 = new ArrayList<>();
Artist b1 = new Artist("c", null, "");
Artist b2 = new Artist("d", null, "");
list2.add(b1);
list2.add(b2);
List<List<Artist>> artists = new ArrayList<>();
artists.add(list);
artists.add(list2);
List<Artist> streamArtists = artists.stream().flatMap(Artist -> Artist.stream()).collect(Collectors.toList());

streamArtists.stream().forEach(item -> System.out.println(item.name));
```
### max min
可以对stream上进行求最大值，求最小值操作，
### reduce
reduce操作，可以实现从一组值中生成一个值，我们平常用到的 Count， Sum min 都可以用reduce来实现。
```java
// 使用Reduce实现Count方法
int cou = Stream.of(1,2,3,4).reduce(0, (a, b) -> a + b);
```

Stream接口的方法如此之多，真令人难以抉择，所以我们要根据我们的代码逻辑，慢慢链式求值，看能不能用Java8的风格来实现原来的代码，进而进行重构。
```java
 List<Integer> list = Stream.of(Stream.of(1,2,3).collect(Collectors.toList()), Stream.of(4,5).collect(Collectors.toList())).flatMap(numbers -> numbers.stream()).collect(Collectors.toList());
 ```
 像这样的函数式代码，谁看到了都头大啊。哈哈 :blush:
