---
title: leetcode backtracking
date: 2017-10-30 21:48:38
categories:
	- Algorithm
tags:
	- Algorithm
	- Java
	- leetcode
---
摘要：回溯法
<!-- more -->
# 回溯法
---
![](/images/sahala1.jpg)
记录一下这道题的解题思路，刚开始我也不会啊:cry: 
## 在每种局面下，你都只有两种选择 
1. 加左括号
2. 加右括号
## 同时又有很多限制。
1 如果左括号用完了，你不能再加入左括号了 
2 如果已经出现的左括号和右括号一样多，则不能再加入右括号了
## 结束条件是： 
左右括号都已经用完
---
	public static List<String> generateParenthesis(int n) {
	    List<String> res = new ArrayList<String>();
	    backtracking("", res, n, n);
		
		return res;
	}
	public static void backtracking(String subString, List<String> res, int left,int right){
		//左右括号 皆用完，则得到的是正解 加入正解集
		if (left == 0 && right == 0){
			res.add(subString);
			return;
		}
		if (left > right || left < 0 || right < 0){
			return;
		}
		backtracking(subString + "(", res, left - 1, right);
		backtracking(subString + ")", res, left, right - 1);	
	}
---

