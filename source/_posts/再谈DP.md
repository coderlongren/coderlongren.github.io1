---
title: 再谈Dp，路径求和
date: 2018-3-30 22:18:40
categories:
	- Algorithm
tags:
	- Algorithm
---
<!-- more -->
Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right which minimizes the sum of all numbers along its path.

* Note *: You can only move either down or right at any point in time.

`Example 1:`  
	[[1,3,1],
	 [1,5,1],
	 [4,2,1]]
Given the above grid map, return 7. Because the path 1→3→1→1→1 minimizes the sum. 

基本和以前的那几道二维数组求动态规划的差不多，从后向前构造。
```java
public static int minPathSum(int[][] grid) {
	if (grid == null || grid.length == 0) {
		return 0;
	}
	return  minPath(grid, grid.length - 1, grid[0].length - 1);
}
// 一提交，就会栈溢出
public static int minPath (int[][] grid, int x, int y) {
	if ((x - 1) >= 0 && (y - 1) >= 0) {
		return grid[x][y] + Math.min(minPath(grid, x - 1, y), minPath(grid, x, y - 1));
	}
	else if ((x - 1) >= 0) {
		return grid[x][y] + minPath(grid, x - 1, y);
	}
	else if ((y - 1) >= 0){
		return grid[x][y] + minPath(grid, x, y - 1);
	}
	else {
		return grid[x][y];
	}
}
```

`使用DP数组，保存结果`
```java
public static int minPathSum2(int[][] grid) {
	if (grid == null || grid.length == 0) {
		return 0;
	}
	int m = grid.length;
	int n = grid[0].length;
	int[][] dp = new int[m][n];
	dp[0][0] = grid[0][0];
	// 第一列DP赋值
	for (int i = 1; i < m; i++) {
		dp[i][0] = dp[i - 1][0] + grid[i][0];
	}
	// 第一行DP赋值
	for (int i = 1; i < n; i++) {
		dp[0][i] = dp[0][i - 1] + grid[0][i];
	}
	for (int i = 1; i < m; i++) {
		for (int j = 1; j < n; j++) {
			dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
		}
	}
	return dp[m - 1][n - 1];
	
	}
```
`还能不能优化了， 用一用C++，好久没写C++了`
```c++
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size();
        int n = grid[0].size();
        vector<int> pre(m, grid[0][0]);
        vector<int> cur(m, 0);
        for (int i = 1; i < m; i++)
            pre[i] = pre[i - 1] + grid[i][0];
        for (int j = 1; j < n; j++) { 
            cur[0] = pre[0] + grid[0][j]; 
            for (int i = 1; i < m; i++)
                cur[i] = min(cur[i - 1], pre[i]) + grid[i][j];
            swap(pre, cur); 
        }
        return pre[m - 1];
    }
};
```
## 卡塔兰数的应用
Given n, how many structurally unique BST's (binary search trees) that store values 1...n?

For example,
Given n = 3, there are a total of 5 unique BST's.

	   1         3     3      2      1
	    \       /     /      / \      \
	     3     2     1      1   3      2
	    /     /       \                 \
	   2     1         2                 3


![参考大神的讲解](https://www.cnblogs.com/grandyang/p/4299608.html)

```java
public static int numTrees(int n) {
		if (n <= 0) {
			return 0;
		}
	    int[] dp = new int[n + 1];
	    dp[0] = 1; // 空BST也算是 1 
	    dp[1] = 1; //
	    for (int i = 2; i <= n; i++) {
	    	for (int j = 0; j < i; j++) {
	    		dp[i] += (dp[j]*dp[i - j - 1]);
	    	}
	    }
	    return dp[n];
	}
```

## Word Break

 Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, determine if s can be segmented into a space-separated sequence of one or more dictionary words. You may assume the dictionary does not contain duplicate words.

## For example, given
s = `"leetcode"`,
dict = `["leet", "code"]`.

Return true because `"leetcode"` can be segmented as `"leet code"`. 
如何抽象这个问题呢？  
dp[0] = true   
从头到尾遍历s, 遇到一个符合的子串时， set dp[i] = true  
下次转移条件，从dp[i] = true开始，按着这个想法，写出代码:   
```java
public boolean wordBreak(String s, List<String> wordDict) {
		int[] dp = new int[s.length() + 1];
		dp[0] = 1;
		for (int i = 1; i <= s.length(); i++) {
			for (int j = 0; j < i; j++) {
				// 转移的条件
				if (dp[j] == 1 && wordDict.contains(s.substring(j, i))) {
					dp[i] = 1;
				}
			}
		}
		return dp[s.length()] == 1;
	}
```
##  leetcode152. Maximum Product Subarray
 Find the contiguous subarray within an array (containing at least one number) which has the largest product.

For example, given the array `[2,3,-2,4]`,
the contiguous subarray `[2,3]` has the largest product = `6`. 

大家应该都会想到算法设计分析书里的那道，求子数组的最大值，或者子序列的，最大值。  
OK，但是
