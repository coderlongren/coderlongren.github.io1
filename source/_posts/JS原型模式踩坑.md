---
title: JS踩坑系列1
date: 2018-6-1 13:18:40
categories:
    - JavaScript
tags:
    - JavaScript
---
摘要： JS原型踩坑
<!-- more -->

面相对象的语言（OO）都有一个标志，就那就是Class， 比如Java，C++，Python然后通过这个Class来创建不同的实例，ECMAJS里面貌似没有Class的概念，所以JS的类和其他的语言的类也不同、  
我们可以把ECMAScript的对象想象成一个散列表的情形，散列表就是一组名值对，这里的值可以是函数或者一般数据。  
## 创建对象
我们都是用过ECMASscript的Object原生对象，
```javascript

var person = new Object();
person.name = "张文鸽";
person.age = 22;
person.job = "Chemistry";
person.sayName = function() {
	console.log(this.name);
};
person.satName();

```
这种方法有个致命的缺点，那就是使用同一个接口会产生大量的对象，产生大量的重复代码，为了解决这个问题，人们开始用工厂模式的变体形式代码。  
## 工厂模式
```javascript
function createPerson(name, age, job) {
	.
	.
	.
}

var person1 = createPerson("SB", "SB", "SB");
```
## 构造函数模式
```javascript
function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function(){
        alert(this.name);
    };    
}

var person1 = new Person("Nicholas", 22, "Software Engineer");
var person2 = new Person("张文鸽", 22, "SB");

person1.sayName();   //"Nicholas"
person2.sayName();   //"张文鸽"
```
## 原型模式
创建的每个函数都有一个prototype对象，这个属性也是一个对象，它用于包含可以由特定类型的所有实例共享的属性和方法。
```javascript

function Person(){
}

Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
    alert(this.name);
};

var person1 = new Person();
person1.sayName();   //"Nicholas"

var person2 = new Person();
person2.sayName();   //"Nicholas"
```
我们把sayName()方法和所有的属性都直接添加到Persion的 prototype属行中，构造函数成了一个空的函数，但我们仍然可以用它。  
1. 理解原型： 
  无论何时，我们创建一个对象，都会根据一组特殊的规则为该函数创建一个prototype属性， 在默认情况，所有的prototype属性都会自动获得一个constructor属性，这个属性指向原有函数，
  `Person.prototype.constructor -- > Person`  
```javascript
function Person(){
}

Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function(){
    alert(this.name);
};

var person1 = new Person();
var person2 = new Person();

person1.name = "Greg";
alert(person1.name);   //"Greg" from 实例
alert(person2.name);   //"Nicholas" – from 原型
```
用 `in`可以查看是否存在 `name`属性，他会搜索实例 && 原型  
`hasOwnPrototype()`可以检测属性是否存在实例中  
## 组合使用构造函数和原型模式

```javascript
 // 相当于 private 
 function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ["Shelby", "Court"];
}

// 相当于Java里面的  public 
Person.prototype = {
    constructor: Person,
    sayName : function () {
        alert(this.name);
    }
};

var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```
这种混合使用的模式，是目前ECMAscript中使用最广泛的创建自定义类型的方法。