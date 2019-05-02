---
title: LeetCode007
date: 2017-10-22 20:01:47
categories:
	- Algorithm
tags:
	- Algorithm
	- java
---
摘要：溢出很扯淡啊

<!-- more -->
正文：
# reverse the Integer int32
![](/images/suanfa.jpeg)

---
但是反反复复输出好几遍，都是提示stackFlow,最够直接用Integer.MAX_VALUE来测试
输出的所有结果，就是到了最后一个溢出了，最后还是百度了，在while循环中加了一个
if判断，
``` bash
	((newresult - tail)/10 != result)
```
说明是溢出，return0jieshu 
---

## 这样也算是解决了吗？
反正提交通过了，我还是没理解啊 

	public int reverse(int x){
		int result = 0;
		while (x != 0) {
			int tail = x%10;
			int newresult = result*10 + tail;
			if ((newresult - tail)/10 != result){
				//如果 这里不相同的话  说明是溢出 
				return 0;
			}
			result = newresult;
			x = x/10;
		}
		return result;
		
	}
---


