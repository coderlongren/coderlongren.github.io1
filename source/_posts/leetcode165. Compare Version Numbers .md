---
title: leetcode 165. Compare Version Numbers 
date: 2018-5-17 21:18:40
categories:
    - Algorithm
tags:
    - Algorithm
---
摘要 ：leetcode165 Middle
<!-- More -->
Compare two version numbers version1 and version2.
If `version1 > version2` return` 1`; if `version1 < version2` return` -1`;otherwise return 0.

You may assume that the version strings are non-empty and contain only digits and the . character.
The . character does not represent a decimal point and is used to separate number sequences.
For instance, `2.5 `is not "two and a half" or "half way to version three", it is the fifth second-level revision of the second first-level revision.

**Example 1:**

Input: `version1 = "0.1"`, `version2 = "1.1"`
Output: -1

**Example 2:**

Input: `version1 = "1.0.1"`, `version2 = "1"`
Output: 1

**Example 3:**

Input: `version1 = "7.5.2.4"`, `version2 = "7.5.3"`
**Output: -1**

版本号的比较，我们可以认为是浮点数的比较。split之后，用长的字符串减去短的字符串，忽略其中的0元素。
```python
class Solution(object):
    def compareVersion(self, version1, version2):
        """
        :type version1: str
        :type version2: str
        :rtype: int
        """
        v1 = [int(x) for x in version1.split(".")]
        v2 = [int(x) for x in version2.split(".")]
        
        diff = []
        for i in range(min(len(v1), len(v2))):
            diff.append(v1[i] - v2[i])
        
        if len(v1) > len(v2):
            diff += [x for x in v1[len(v2):]]
        else:
            diff += [-x for x in v2[len(v1):]]
        if all(x == 0 for x in diff):
            return 0
        
        for i in range(len(diff)):
            if diff[i] == 0:
                continue
                
            if diff[i] > 0:
                return 1
            else:
                return -1`
```


