---
title: 363.Max Sum of Rectangle No Larger Than K
date: 2018-2-7 21:48:38
categories:
	- Algorithm
tags:
	- Algorithm
	- java
	- leetcode
---
Given a non-empty 2D matrix matrix and an integer k, find the max sum of a rectangle in the matrix such that its sum is no larger than k.

*Example:*
	Given matrix = [
	  [1,  0, 1],
	  [0, -2, 3]
	]
	k = 2

The answer is **2**. Because the sum of rectangle **[[0, 1], [-2, 3]]** is 2 and 2 is the max number no larger than k (k = 2).  

Note:
    The rectangle inside the matrix must have an area > 0.
    What if the number of rows is much larger than the number of columns?

按照`[LeetCode] Range Sum Query 2D - Immutable 二维区域和检索 - 不可变` 一样的方法，使用O(n^4)复杂度得出答案。

```java
public static int maxSumSubmatrix(int[][] matrix, int k) {
			if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
				return -1;
			}
			// DP数组保存的是右下角和左上角之间区域的和
	        int[][] dp = new int[matrix.length + 1][matrix[0].length + 1];
//	        dp[0][0] = matrix[0][0];
	        for (int i = 1; i <= matrix.length; i++) {
	        	for (int j = 1; j <= matrix[0].length; j++) {
	        		dp[i][j] = dp[i - 1][j] + dp[i][j - 1] - dp[i - 1][j - 1] + matrix[i - 1][j - 1];
	        	}
	        }
	        int res = Integer.MIN_VALUE;
	        // 这里O(n^4)应该是可以改善的，，，，，，，,暂时还没注意。
	        for (int i = 1; i <= matrix.length; i++) {
	        	for (int j = 1; j <= matrix[0].length; j++) {
	        		for (int h = i; h <= matrix.length; h++) {
	        			for (int s = j; s <= matrix[0].length; s++) {
	        				if (sumRegion(i, j, h, s, dp) == k) {
	        					return k;
	        				}
	        				if (sumRegion(i, j, h, s, dp) < k) {
	        					res = Math.max(res, sumRegion(i, j, h, s, dp));
	        				}
	        			}
	        		}
	        	}
	        }
	        // 没有得到小于k的res
	        if (res == Integer.MIN_VALUE) {
	        	return -1;
	        }
	        return res;
	 }
	 // 求区域和的函数
	 public static int sumRegion(int row1, int col1, int row2, int col2, int[][] dp) {
	    	
			return dp[row2][col2] - dp[row2][col1 - 1] - dp[row1 - 1][col2] + dp[row1 - 1][col1 - 1];
	        
	 }
```

---

 Range Sum Query 2D - Immutable 和上面那个题目是相同的思想 

Given a 2D matrix matrix, find the sum of the elements inside the rectangle defined by its upper left corner (row1, col1) and lower right corner (row2, col2).

Range Sum Query 2D
The above rectangle (with the red border) is defined by (row1, col1) = (2, 1) and (row2, col2) = (4, 3), which contains sum = 8.

Example:

Given matrix = [
  [3, 0, 1, 4, 2],
  [5, 6, 3, 2, 1],
  [1, 2, 0, 1, 5],
  [4, 1, 0, 1, 7],
  [1, 0, 3, 0, 5]
]

sumRegion(2, 1, 4, 3) -> 8
sumRegion(1, 1, 2, 2) -> 11
sumRegion(1, 2, 2, 4) -> 12

Note:

    You may assume that the matrix does not change.
    There are many calls to sumRegion function.
    You may assume that row1 ≤ row2 and col1 ≤ col2.

```java
public class NumMatrix {
	private int[][] dp;
	public NumMatrix(int[][] matrix) {
		if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
			return ;
		}
        int rows = matrix.length;
        int cols = matrix[0].length;
        dp = new int[rows + 1][cols + 1];
        dp[0][0] = matrix[0][0];
//        for (int i = 1; i < rows; i++) {
//        	dp[i][0] = matrix[i][0] + dp[i - 1][0];
//        }
//        for (int i = 1; i < cols; i++) {
//        	dp[0][i] = matrix[0][i] + dp[0][i - 1];
//        }
        
        for (int i = 1; i <= rows; i++) {
        	for (int j = 1; j <= cols; j++) {
        		dp[i][j] = dp[i - 1][j] + dp[i][j - 1] - dp[i - 1][j - 1] + matrix[i - 1][j - 1];
        	}
        }
    }
    
    public int sumRegion(int row1, int col1, int row2, int col2) {
    	
		return dp[row2 + 1][col2 + 1] - dp[row2 + 1][col1] - dp[row1][col2 + 1] + dp[row1][col1];
        
    }

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int[][] matrix = {
		                {3, 0, 1, 4, 2},
		                {5, 6, 3, 2, 1},
		                {1, 2, 0, 1, 5},
		                {4, 1, 0, 1, 7},
		                {1, 0, 3, 0, 5}
		};
		NumMatrix matrix2 = new NumMatrix(matrix);
		System.out.println(matrix2.sumRegion(1,2,2,4));
	}

}
```

