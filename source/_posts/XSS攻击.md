---
title: XSS攻击
date: 2018-05-04 23:26:52
categories:
	- Web
tags:
	- Web安全
---
摘要：XSS攻击原理解释
<!-- more -->

### 概念：  
XSS攻击是Web攻击中最常见的攻击方法之一，它是通过对网页注入可执行代码且成功地被浏览器执行，达到攻击的目的，形成了一次有效XSS攻击，一旦攻击成功，它可以获取用户的联系人列表，然后向联系人发送虚假诈骗信息，可以删除用户的日志等等，有时候还和其他攻击方式同时实施比如SQL注入攻击服务器和数据库、Click劫持、相对链接劫持等实施钓鱼，它带来的危害是巨大的，是web安全的头号大敌。  
攻击成功的条件：  
1. 必须往Web页面注入恶意代码
2. 这些代码可以被浏览器所执行
### 给出一些例子
```html
<div id="el" style="background:url('javascript:eval(document.getElementById("el").getAttribute("code")) ')"
    code="var a = document.createElement('a');
    a.innerHTML= 'ke 执行代码';
    document.body.appendChild(a);//这这里执行代码
    ">
</div>
```
```html
<div>

  <img src="/images/handler.ashx?id=<%= Request.QueryString["id"] %>" />
  </div>
```
		|
		|
		|
```html
http://www.xxx.com/?id=" /><script>alert(/xss/)</script><br x="

最终反射出来的HTML代码：
    <div>
    <img src="/images/handler.ashx?id=" /><script>alert(/xss/)</script><br x="" />
    </div>
```
// TODO, 下面的代码自己还没搞清楚暂时不粘贴


// TODO 如何解决呢

// TODO 
