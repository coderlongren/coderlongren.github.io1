---
title: JDK注解的理解与使用
date: 2018-3-1 14:18:40
categories:
	- JDK
tags:
	- 代码重构
	- JDK
---

# 一，什么是注解

我们大家都知道Java代码中使用注释是为了向以后阅读这份代码的人解释说明一些事情，注解是注释的升级版，它可以向编译器、虚拟机等解释说明一些事情。比如我们非常熟悉的@Override就是一种元注解，它的作用是告诉编译器它所注解的方法是重写父类的方法，这样编译器就会去检查父类是否存在这个方法，以及这个方法的签名与父类是否相同。

也就是说，注解是描述Java代码的代码，它能够被编译器解析，注解处理工具在运行时也能够解析注解。我们在Java源文件中使用注释，是为了以后我们或他人再来读这段代码时，能够更好地理解它。Javadoc工具可以解析我们在源代码中为类、方法、变量等添加的描述信息，并根据这些描述信息自动生成一个HTML文档，这些自动生成的文档即可作为API帮助文档。只要我们为类、方法等添加的描述信息符合Javadoc要求的语法，我们就能够使用Javadoc工具根据我们的描述信息自动生成一个帮助文档。而注解比java注释和Javadoc要强大得多，它们三者之间的重大的区别在于，Java注释和Javadoc描述所发挥的作用仅仅到编译时就止步了，而注解直到运行时都能够发挥作用。

我们知道，使用“transient”关键字可以告诉编译器这个域不可序列化。相比于用”transient“这样的关键字修饰一个属性，注解为我们提供了为类/方法/属性/变量添加描述信息的更通用的方式，而这些描述信息对于开发者、自动化工具、Java编译器和Java运行时来说都是有意义的，也就是说他们都能“读懂”注解信息。”transient“关键字是一个修饰符，而注解也是一种修饰符。除了传递信息，我们也可以使用注解生成代码。我们可以使用注解，然后让注解解析工具来解析它们，以此来生成一些”模板化“的代码。比如Hibernate、Spring、Axis这些框架大量使用了注解，来避免一些重复的工作。

# 元注解
元注解即用来描述注解的注解，比如以下代码中我们使用“@Target”元注解来说明MethodInfo这个注解只能应用于对方法进行注解：
```java
@Target(ElementType.METHOD)
public @interface MethodInfo { 
    ...
}
```
## 1 Docemented
当一个注解类型被@Documented元注解所描述时，那么无论在哪里使用这个注解，都会被Javadoc工具文档化。我们来看一下它的定义：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {

}
```

定义注解使用@interface关键字  
这个元注解被@Documented修饰，表示它本身也会被文档化。  @Retention元注解的值RetentionPolicy.RUNTIME表示@Documented这个注解能保留到运行时；@Target元注解的值ElementType.ANNOTATION_TYPE表示@Documented这个注解只能够用来描述注解类型。
## 2. Inherited
表明被修饰的注解类型是自动继承的。具体解释如下：若一个注解类型被Inherited元注解所修饰，则当用户在一个类声明中查询该注解类型时，若发现这个类声明中不包含这个注解类型，则会自动在这个类的父类中查询相应的注解类型，这个过程会被重复，直到该注解类型被找到或是查找完了Object类还未找到。这个元注解的定义如下：
```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```
这个元注解类型被@Documented所注解，能够保留到运行时，只能用来描述注解类型。
## 3. Retention
它表示一个注解类型会被保留到什么时候，比如以下代码表示Developer注解会被保留到运行时：

```
@Retention(RetentionPolicy.RUNTIME)
public @interface Developer {
    String value();
}

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    RetentionPolicy value();
}
```
我们在使用@Retention时，后面括号里的内容即表示他的取值，从以上定义我们可以看到，取值的类型为RetentionPolicy，这是一个枚举类型，它可以取以下值：
* SOURCE：表示在编译时这个注解会被移除，不会包含在编译后产生的class文件中；
* CLASS：表示这个注解会被包含在class文件中，但在运行时会被移除；
* RUNTIME：表示这个注解会被保留到运行时，在运行时可以JVM访问到，我们可以在运行时通过反射解析这个注解。

## 4 Target
这个元注解说明了被修饰的注解的应用范围，也就是被修饰的注解可以用来注解哪些程序元素，它的定义如下：

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

看到它也会保留到运行时，而且它的取值是为ElementType[]类型（一个数组，意思是可以指定多个值），ElementType是一个枚举类型，它可以取以下值：

* TYPE：表示可以用来注解类、接口、注解类型或枚举类型；
* PACKAGE：可以用来注解包；
* PARAMETER：可以用来注解参数；
* ANNOTATION_TYPE：可以用来注解 注解类型；
* METHOD：可以用来注解方法；
* FIELD：可以用来注解属性（包括枚举常量）；
* CONSTRUCTOR：可以用来注解构造器；
* LOCAL_VARIABLE：可用来注解局部变量。

# 三、常见内建注解

Java本身内建了一些注解，下面我们来介绍一下我们在日常开发中比较常见的注解：@Override、@Deprecated、@SuppressWarnings。相信我们大家或多或少都使用过这三个注解

## @Override注解
先看一下定义
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```
从它的定义我们可以看到，这个注解可以被用来修饰方法，并且它只在编译时有效，在编译后的class文件中便不再存在。这个注解的作用我们大家都不陌生，那就是告诉编译器被修饰的方法是重写的父类的中的相同签名的方法，编译器会对此做出检查，若发现父类中不存在这个方法或是存在的方法签名不同，则会报错。

## @Deprecated
定义如下:
```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```
从它的定义我们可以知道，它会被文档化，能够保留到运行时，能够修饰构造方法、属性、局部变量、方法、包、参数、类型。这个注解的作用是告诉编译器被修饰的程序元素已被“废弃”，不再建议用户使用。

## @SuppressWarnings
这个注解我们也比较常用到，先来看下它的定义：
```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```
它能够修饰的程序元素包括类型、属性、方法、参数、构造器、局部变量，只能存活在源码时，取值为String[]。它的作用是告诉编译器忽略指定的警告信息，它可以取的值如下所示：
* deprecation：忽略使用了废弃的类或方法时的警告；
* unchecked：执行了未检查的转换
* fallthrough：swich语句款中case忘加break从而直接“落入”下一个case；
* path：类路径或原文件路径等不存在；
* all：以上所有情况产生的警告均忽略。

这个注解如何使用呢？ 
```
@SuppressWarning(value={"deprecation", "unchecked"})
public void myMethos() {...}
// 通过使用以上注解，我们告诉编译器忽略myMethod方法中由于使用了废弃的类或方法或是做了未检查的转换而产生的警告。
```

# 四、自定义注解
我们可以创建我们自己的注解类型并使用它。
```java
Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Inherited
public @interface MethodInfo {
    String author() default "coderlong";
    String date();
    int version() default 1;
}
```

* 注解类型是通过 @interface关键字定义的
* 在”注解体“中，所有的方法均没有方法体且只允许public和abstract这两种修饰符号（不加修饰符缺省为public），注解方法不允许有throws子句；
* 注解方法的返回值只能为以下几种：原始数据类型）, String, Class, 枚举类型, 注解和它们的一维数组，可以为方法指定默认返回值。
