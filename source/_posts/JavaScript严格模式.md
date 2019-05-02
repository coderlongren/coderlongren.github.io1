---
title: Javascript use strict
date: 2018-1-21 14:18:40
categories:
	- javascript
tags:
	- javascript
---
转载[博客园](https://www.cnblogs.com/jiqing9006/p/5091491.html)
## 1 概述
除了正常运行模式，ECMAscript 5添加了第二种运行模式："严格模式"（strict mode）。顾名思义，这种模式使得Javascript在更严格的条件下运行。
## 2 为何使用严格模式
- 消除Javascript语法的一些不合理、不严谨之处，减少一些怪异行为;

- 消除代码运行的一些不安全之处，保证代码运行的安全；

- 提高编译器效率，增加运行速度；

- 为未来新版本的Javascript做好铺垫。

"严格模式"体现了Javascript更合理、更安全、更严谨的发展方向，包括IE 10在内的主流浏览器，都已经支持它，许多大项目已经开始全面拥抱它。

另一方面，同样的代码，在"严格模式"中，可能会有不一样的运行结果；一些在"正常模式"下可以运行的语句，在"严格模式"下将不能运行。掌握这些内容，有助于更细致深入地理解Javascript，让你变成一个更好的程序员。
## 3 进入标志
```javascript
"use strict"
```

## 4 如何调用
```javascript
1
<script>
　 "use strict";
　 console.log("这是严格模式。");
</script>

2 
function strict(){
　　　　"use strict";
　　　　return "这是严格模式。";
　　}
　　function notStrict() {
　　　　return "这是正常模式。";
　　}
```

## 5 语法行为改变
5.1 在正常模式中，如果一个变量没有声明就赋值，默认是全局变量。严格模式禁止这种用法，全局变量必须显式声明。
```javascript
"use strict";
　　v = 1; // 报错，v未声明
　　for(i = 0; i < 2; i++) { // 报错，i未声明
　　}
```
5.2 禁止this关键字指向全局对象
```javascript
function f(){
　　　　return !this;
　　} 
　　// 返回false，因为"this"指向全局对象，"!this"就是false
　　function f(){ 
　　　　"use strict";
　　　　return !this;
　　} 
　　// 返回true，因为严格模式下，this的值为undefined，所以"!this"为true。
```
5.3 禁止删除变量
5.4 对象不能有重名的属性