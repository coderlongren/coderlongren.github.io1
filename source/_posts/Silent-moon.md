---
title: Silent moon
date: 2017-10-26 21:00:54
categories:
	- 生活
tags:
	- moon
	- 生活
	- algorithm
---
摘要： 很久没见到月亮了
<!-- more -->
## 大数问题
---
![](/images/moon.jpeg)
以前做过一道，求2的几百次方的大数运算，用的字符数组存储大数的每一位
Java就更加方便了，有专门的解决方案

下面是 求两个大数的乘积 
---
	public static String multiple(String x,String y){
		String result = "";
		if (x == "1"){
			return y;
		}
		if (y == "1"){
			return x;
		}
		char[] str1 = x.toCharArray();
		char[] str2 = y.toCharArray();
		int length1 = str1.length;
		int length2 = str2.length;
		int[] res =new int[length1 + length2 - 1];
		//进行笛卡尔积 
		for (int i = 0; i < length1; i++){
			int n1 = str1[i] - '0';
			for (int j = 0; j < length2; j++){
				int n2 = str2[j] - '0';
				res[i + j] += n1*n2;
			}
		}
			
		for (int i = res.length - 1; i > 0; i--){
				res[i - 1] += res[i]/10;
				res[i] = res[i]%10;		
		}
		for (int i = 0; i< res.length ;i++){
			result+=res[i];
		}
		return result;
	}
---
生活中没有算法！
有点麻烦的就是那个进位，出错了很多次，才找到合适的方法 

