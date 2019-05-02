---
title: 浅谈java里面的语法糖
date: 2018-3-17 20:18:40
categories:
	- JDK
tags:
	- JDK
---
转载自http://blog.csdn.net/danchu/article/details/54986442

## 泛型与类型擦除
> Java语言并不是一开始就支持泛型的。在早期的JDK中，只能通过Object类是所有类型的父类和强制类型转换来实现泛型的功能。强制类型转换的缺点就是把编译期间的问题延迟到运行时，JVM并不能为我们提供编译期间的检查。

> 在JDK1.5中，Java语言引入了泛型机制。但是这种泛型机制是通过类型擦除来实现的，即Java中的泛型只在程序源代码中有效（源代码阶段提供类型检查），在编译后的字节码中自动用强制类型转换进行替代。也就是说，Java语言中的泛型机制其实就是一颗语法糖，相较与C++、C#相比，其泛型实现实在是不那么优雅
```java
/**
* 在源代码中存在泛型
*/
public static void main(String[] args) {
    Map<String,String> map = new HashMap<String,String>();
    map.put("hello","你好");
    String hello = map.get("hello");
    System.out.println(hello);
}
// 当上述源代码被编译为class文件后，泛型被擦除且引入强制类型转换
public static void main(String[] args) {
    HashMap map = new HashMap(); //类型擦除
    map.put("hello", "你好");
    String hello = (String)map.get("hello");//强制转换
    System.out.println(hello);
}
```

## 自动装箱与拆箱
> 下面代码演示了自动装箱和拆箱功能
```java
public static void main(String[] args) {
    Integer a = 1;
    int b = 2;
    int c = a + b;
    System.out.println(c);
}

// 经过编译后，代码如下

public static void main(String[] args) {
    Integer a = Integer.valueOf(1); // 自动装箱
    byte b = 2;
    int c = a.intValue() + b;//自动拆箱
    System.out.println(c);
}

```
## 变长参数
> 变长参数特性是在JDK1.5中引入的，使用变长参数有两个条件，一是变长的那一部分参数具有相同的类型，二是变长参数必须位于方法参数列表的最后面。变长参数同样是Java中的语法糖，其内部实现是Java数组。

```java
public class Varargs {
    public static void print(String... args) {
        for(String str : args){
            System.out.println(str);
        }
    }

    public static void main(String[] args) {
        print("hello", "world");
    }
}

// 编译为class文件后如下，从中可以很明显的看出变长参数内部是通过数组实现的
public class Varargs {
    public Varargs() {
    }

    public static void print(String... args) {
        String[] var1 = args;
        int var2 = args.length;
        //增强for循环的数组实现方式
        for(int var3 = 0; var3 < var2; ++var3) {
            String str = var1[var3];
            System.out.println(str);
        }

    }

    public static void main(String[] args) {
        //变长参数转换为数组
        print(new String[]{"hello", "world"});
    }
}
```

## 变长参数
> 所谓变长参数，就是方法可以接受长度不定确定的参数

变长参数特性是在JDK1.5中引入的，使用变长参数有两个条件，一是变长的那一部分参数具有相同的类型，二是变长参数必须位于方法参数列表的最后面。变长参数同样是Java中的语法糖，其内部实现是Java数组。

```java
public class Varargs {
    public static void print(String... args) {
        for(String str : args){
            System.out.println(str);
        }
    }

    public static void main(String[] args) {
        print("hello", "world");
    }
}

// 编译为class文件后如下，从中可以很明显的看出变长参数内部是通过数组实现的
public class Varargs {
    public Varargs() {
    }

    public static void print(String... args) {
        String[] var1 = args;
        int var2 = args.length;
        //增强for循环的数组实现方式
        for(int var3 = 0; var3 < var2; ++var3) {
            String str = var1[var3];
            System.out.println(str);
        }

    }

    public static void main(String[] args) {
        //变长参数转换为数组
        print(new String[]{"hello", "world"});
    }
}
```

## 增强的for循环
> 增强for循环与普通for循环相比，功能更强并且代码更简洁

## 内部类
> 内部类就是定义在一个类内部的类
Java语言中之所以引入内部类，是因为有些时候一个类只在另一个类中有用，我们不想让其在另外一个地方被使用。内部类之所以是语法糖，是因为其只是一个编译时的概念，一旦编译完成，编译器就会为内部类生成一个单独的class文件，名为outer$innter.class。
## 枚举类型
> 枚举类型就是一些具有相同特性的类常量
```java
public enum Fruit {
    APPLE,ORINGE
}
```

jad编译之后再反编译
```java
//继承java.lang.Enum并声明为final
public final class Fruit extends Enum
{

    public static Fruit[] values()
    {
        return (Fruit[])$VALUES.clone();
    }

    public static Fruit valueOf(String s)
    {
        return (Fruit)Enum.valueOf(Fruit, s);
    }

    private Fruit(String s, int i)
    {
        super(s, i);
    }
    //枚举类型常量
    public static final Fruit APPLE;
    public static final Fruit ORANGE;
    private static final Fruit $VALUES[];//使用数组进行维护

    static
    {
        APPLE = new Fruit("APPLE", 0);
        ORANGE = new Fruit("ORANGE", 1);
        $VALUES = (new Fruit[] {
            APPLE, ORANGE
        });
    }
}
```