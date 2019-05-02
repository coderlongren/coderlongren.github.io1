---
title: leetcode300. Longest Increasing Subsequence
date: 2018-4-4 22:18:40
categories:
	- Algorithm
tags:
	- Algorithm
---
Given an unsorted array of integers, find the length of longest increasing subsequence.

*For example*,
Given `[10, 9, 2, 5, 3, 7, 101, 18]`,
The longest increasing subsequence is `[2, 3, 7, 101]`, therefore the length is 4. Note that there may be more than one LIS combination, it is only necessary for you to return the length.

Your algorithm should run in O(n2) complexity.

*Follow up*: Could you improve it to O(n log n) time complexity? 
```java
// 如果只是，求连续的递增子序列，下面的DP函数，就可以求出
public static int lengthOfLIS(int[] nums) {
	if (nums == null || nums.length == 0) {
		return 0;
	}
	if (nums.length == 1) {
		return 1;
	}
	int[] dp = new int[nums.length  + 1];
	dp[0] = 1;
	int res = Integer.MIN_VALUE;
	for (int i = 1; i < nums.length; i++) {
		dp[i] = nums[i] > nums[i - 1] ? dp[i - 1] + 1 : 1;
		res = Math.max(res, dp[i]);
	}
	return res;
 }
```

```java
// 非连续的递增子序列, 控制在O(n^2)
public static int lengthOfLIS2(int[] nums) {
	if (nums == null || nums.length == 0) {
		return 0;
	}
	if (nums.length == 1) {
		return 1;
	}
	int[] dp = new int[nums.length  + 1];
	dp[0] = 1;
	int res = Integer.MIN_VALUE;
	for (int i = 1; i < nums.length; i++) {
		int temp = Integer.MIN_VALUE;
		for (int j = 0; j < i; j++) {
			if (nums[i] > nums[j]) {
				temp = Math.max(temp, dp[j]);
			}
		}
		dp[i] = temp == Integer.MIN_VALUE ? 1 : temp + 1;
		res = Math.max(res, dp[i]);
	}
	return res;
}
```