---
title: String类的indexOf用法举例
date: 2017-11-08 20:16:10
categories:
	- java
tags:
	- tomcat
	- java
---
摘要: String类的indexof方法
<!-- more -->

最近在学习Tomcat得组织结构，在自组建socket的解析 uri的方法中，多次使用了String的
indexOf方法，所以摘录了一下，常用的方法  
``` bash
int indexOf(String str)
```
返回第一次出现的指定子字符串在此字符串的索引

``` bash
int indexOf(String str,int startIndex)
```
从指定的索引处开始，返回第一次出现的指定子字符串在此字符串中的索引。

``` bash
int lastIndexOf(String str)
```
返回此字符串中最右边 出现的指定子字符串在此字符串中的索引
``` bash
int lastIndexOf(String str,int startIndex)
```
从指定的索引处开始向后搜索，返回在此字符串中最后一次出现的指定子字符串的索引。

---

