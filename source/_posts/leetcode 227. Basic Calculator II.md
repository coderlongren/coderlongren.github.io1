---
title: leetcode 227. Basic Calculator II
date: 2018-5-17 21:18:40
categories:
    - leetcode
tags:
    - leetcode
---
摘要：简易表达式求值
<!-- more -->

Implement a basic calculator to evaluate a simple expression string.

The expression string contains only non-negative integers, `+, -, *, / `operators and empty spaces . The integer division should truncate toward zero.

**Example 1:**

Input: `"3+2*2"`
**Output**: `7`

**Example 2:**

Input: `" 3/2 "`
**Output:**` 1`

**Example 3:**

Input:` " 3+5 / 2 "`
**Output:** `5`

我们在上数据结构课程的时候，都学过用后缀表达式来计算 字符串表达式的值。
这道题目，就是字符串求值的简易版本，明确的告诉了我们，只会出现 `+` `- ` `* ` `/ `四种计算符号，不会出现 `( )` ，这就降低了我们难度，直接来写后缀表达式的算法，还是挺长的。
LeetCode一向以面试算法为主，如何在短时间内写出一个计算值得算法呢？ 
可以利用 `Stack `先行标记`sing = '+'`, 数字全都进栈，遇到了`* / `,我们就把栈顶元素出栈，计算之后在进栈。最后`Stack`里面的所有的数，都是 + - 自然计算顺序的数字，直接求和就行了。
`c++版本`
```c++
class Solution {
public:
    int calculate(string s) {
        stack<int> myStack;
        char sign = '+';
        int res = 0, tmp = 0;
        for (unsigned int i = 0; i < s.size(); i++) {
            if (isdigit(s[i]))
                tmp = 10*tmp + s[i]-'0';
            if (!isdigit(s[i]) && !isspace(s[i]) || i == s.size()-1) {
                if (sign == '-')
                    myStack.push(-tmp);
                else if (sign == '+')
                    myStack.push(tmp);
                else {
                    int num;
                    if (sign == '*' )
                        num = myStack.top()*tmp;
                    else
                        num = myStack.top()/tmp;
                    myStack.pop();
                    myStack.push(num);
                } 
                sign = s[i];
                tmp = 0;
            }
        }
        while (!myStack.empty()) {
            res += myStack.top();
            myStack.pop();
        }
        return res;
    }
};
```

`Java版本`
```java
public static int calculate(String s) {
        char[] chars = s.toCharArray();
        int res = 0;
        Stack<Integer> stack = new Stack<>();
        char sign = '+';
        int temp = 0;
        for (int i = 0; i < chars.length; i++) {
        	if (Character.isDigit(chars[i])) { // 数字入栈
        		temp = 10*temp + (chars[i] - '0');
        	}
        	if (!Character.isDigit(chars[i]) && chars[i] != ' ' || i == chars.length - 1) {
        		boolean flag = false;
        		if (sign == '+') {
        			stack.push(temp);
        		}
        		if (sign == '-') {
        			stack.push(-temp);
        		}
        		if (sign == '*') {
        			int num = stack.pop();
        			num = num * temp;
        			stack.push(num);
        		}
        		if (sign == '/') {
        			int num = stack.pop();
        			num = num/temp;
        			stack.push(num);
        		}

        		sign = chars[i]; // sign 始终记录上一个符号位
        		temp = 0; // temp 复位0
        	}
        }
        
        for (int i : stack) {
        	res += i;
        }
        return res;
    }
```
`python版本`
