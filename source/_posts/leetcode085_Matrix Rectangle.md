---
title: leetcode 85. Maximal Rectangle
date: 2018-8-14 10:18:40
categories:
    - Algorithm
tags:
    - Algorithm
---
Given a 2D binary matrix filled with 0's and 1's, find the largest rectangle containing only 1's and return its area.

`Example`:

Input:
`
[
  ["1","0","1","0","0"],
  ["1","0","1","1","1"],
  ["1","1","1","1","1"],
  ["1","0","0","1","0"]
]
`
Output: 6
初次见你是在keep的笔试中，让我绞尽脑汁，一个小时都没AC掉，二次见你是在CodeM比赛中，还是不会，为什么每次写这道题时，总想用do[i][j] = max(dp[i - 1][j], dp[i][j - 1]) 和前面的三块比较来求转移方程。这是受了那道题目影响。  
今日把你AC掉，以报昔日之仇。  
具体方法是，在每一行的基础上，去求“1”组成的柱状图的最大面积， 可以参考LeetCode84 max rectangleArea in history中，求最大柱状图的算法，使用 left, right数组维持当前坐标,左边最大值坐标，右边最大值坐标。最后求积。

```java
class Solution {
    public int maximalRectangle(char[][] matrix) {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
    		return 0;
    	}
    	int[] height = new int[matrix[0].length];
    	int max = 0;
    	for (int i = 0; i < matrix.length; i++) {
    		// 这里的循环是最精妙之处，相当于求出以当前行为地面，求面积最大的矩形。
    		// height 一直维持着 每座山峰的高度
    		for (int j = 0; j < height.length; j++) {
    			if (matrix[i][j] == '1') {
    				height[j] += 1;
    			}
    			else {
    				height[j] = 0;
    			}
    		}
    		max = Math.max(max, maxarea(matrix, height));
    	}
    	return max;
    }
    public  int maxarea(char[][] matrix, int[] height) {
    	int n = height.length;
    	int[] left = new int[n];
    	int[] right = new int[n];
    	left[0] = -1; // left记录 每座山峰的最左边 >= 自己的坐标
    	right[n - 1] = n; // right 记录每座山峰最右边 >= 自己的坐标
    	for (int i = 1; i < n; i++) {
    		int p = i - 1;
    		while (p >= 0 && height[p] >= height[i]) {
    			p = left[p];
    		}
    		left[i] = p;
    	}
    	for (int i = n - 2; i >= 0; i--) {
    		int p = i + 1;
    		while (p < n && height[p] >= height[i]) {
    			p = right[p];
    		}
    		right[i] = p;
    	}
    	int max = 0;
    	for (int i = 0; i < n; i++) {
    		max = Math.max(max, height[i] * (right[i] - left[i] - 1));
    	}
    	return max;
    }
}
```

顺便附上，一个N皇后问题的，回溯法，最近在练习一些经典算法，怕了怕了，面试。
```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
/**
* @author 作者 : coderlong
* @version 创建时间：2018年8月24日 上午11:12:01
* 类说明: 
*/
public class N_Queue {
	public static void main(String[] args) {
		List<List<String>> res = getSolutionNQueue(8);
		for (List<String> item : res) {
			for (String string : item) {
				System.out.println(string);
			}
		}
		System.out.println(res.size() + "解决方案");
	}
	public static List<List<String>> getSolutionNQueue(int n) {
		int rowIndex = 0;
		List<List<String>> res = new ArrayList<>();
		boolean[] row = new boolean[n]; // row 记忆
		boolean[] left = new boolean[2 * n]; // 左下角 记忆
		boolean[] right = new boolean[2 * n]; // 右下角 记忆
		backtrack(res, new ArrayList<>(), row, left, right, rowIndex, n);
		return res;
	}
	public static void backtrack(List<List<String>> res, ArrayList<String> temp, boolean[] row, boolean[] left, boolean[] right, int rowIndex, int n) {
		if (rowIndex == n) {
			res.add(new ArrayList<>(temp));
			return;
		}
		for (int i = 0; i < n; i++) {
			if (row[i] || left[i + rowIndex] || right[rowIndex - i + n - 1]) {
				continue;
			}
			
			row[i] = true;
			left[i + rowIndex] = true;
			right[rowIndex - i + n - 1] = true;
			
			char[] tempString = new char[n];
			Arrays.fill(tempString, '.');
			tempString[i] = '*';
			String string = "";
			for (char c : tempString) {
				string += c + "";
			}
			temp.add(string);
			backtrack(res, temp, row, left, right, rowIndex + 1, n);
			row[i] = false;
			left[i + rowIndex] = false;
			right[rowIndex - i + n - 1] = false;
			temp.remove(temp.size() - 1);
		}
	}
}

```