---
title:  字符串编辑距离
date: 2018-6-8 21:18:40
categories:
	- Algorithm
tags:
	- Algorithm

---

摘要： 基本动态规划
<!-- mroe -->
给你两个字符串  
例如： `word1: horse`, `word2:ros`  
你有三种选择： 
1. 插入
2. 删除
3. 替换
把一个字符串变为另一个字符串  
```
horse -> rorse (replace 'h' with 'r')
rorse -> rose (remove 'r')
rose -> ros (remove 'e')
```

我们可以构造一张二维表
	   __________
	 0 s t r 1
	|s 0 1 2 3
	|t 1
	|r 2
	|2 3


`代码就是最好的注释`
```python
class Solution:
    def minDistance(self, word1, word2):
        """
        :type word1: str
        :type word2: str
        :rtype: int
        """
        len1 = len(word1)
        len2 = len(word2)
        arr = [[0 for i in range(len1 + 1)] for i in range(len2 + 1)]
        arr[0][0] = 0
        i = 1
        while i in range(1, len1 + 1):
            arr[0][i] = i
            i += 1
        i = 1
        while i in range(1, len2 + 1):
            arr[i][0] = i
            i += 1
        temp = 0
        i = 1
        j = 1
        while i < (len2 + 1):
            while j < (len1 + 1):
                if word1[j - 1] == word2[i - 1]:
                    temp = 0
                else:
                    temp = 1
                arr[i][j] = min(arr[i - 1][j] + 1, arr[i][j - 1] + 1, arr[i - 1][j - 1] + temp)
                j += 1
            j = 1
            i += 1
        return arr[len2][len1]
              
```