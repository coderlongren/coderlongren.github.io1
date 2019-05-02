---
title: DP的一点总结
date: 2018-3-27 10:18:40
categories:
	- Algorithm
tags:
	- Algorithm
---
摘要：三言两语之DP
<!-- more -->
## 动态规划算法适用于解最优化问题。通常可按以下4个步骤设计：
1. 找出最优解的性质，并刻画其结构特征；
2. 递归地定义最优值；
3. 以自底向上的方式计算出最优值；
4. 根据计算最优值时得到的信息，构造最优解。
## 动态规划算法的基本要素:
1. 最优子结构
当问题的最优解包含了其子问题的最优解时，称该问题具有最优子结构性质。问题的最优子结构性质提供了该问题可用动态规划算法求解的重要线索。
2. 重叠子问题
可用动态规划算法求解的问题应具备的另一个基本要素是子问题的重叠性质。在用递归算法自顶向下解此问题时，每次产生的子问题并不总是新问题，有些子问题被反复计算多次。动态规划算法正是利用了这种子问题的重叠性质，对每一个子问题只解一次，而后将其解保存在一个表格里中，当再次需要解此子问题时，只是简单地用常数时间查看一下结果。通常，不同的子问题个数随问题的大小呈多项式增长。因此，用动态规划算法只需要多项式时间，从而获得较高的解题效率。

`层数塔` `记忆数组方法` `DP方法`
```java
public class POJ121_Trangle_Max {
	// 存储数组
	static int[][] D = new int[200][200];
	// 记忆数组
	static int[][] max = new int[200][200];
	static int n = 0;
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Scanner scanner = new Scanner(System.in);
		n = scanner.nextInt();
		for (int i = 0; i < n; i++) {
			for (int j = 0; j <= i; j++) {
				D[i][j] = scanner.nextInt();
			}
		}
		System.out.println(maxNum(0, 0));
	}
	public static int maxNum (int i, int j) {
		if (max[i][j] != 0) {
			return max[i][j];
		}
		// 边界条件， 到达了最后一层， 就直接赋值了。
		if (i == n - 1) {
			max[i][j] = D[i][j];
		}
		else {
			int x = maxNum(i + 1, j);
			int y = maxNum(i + 1, j + 1);
			max[i][j] = Math.max(x, y) + D[i][j];
		}
		return max[i][j];
	}

}
```
Dp
```java
public class POJ121_Trangle_Max2 {
	static int[][] D = new int[200][200];
	static int[][] dp = new int[200][200];
	static int n = 0;
	public static void main(String[] args) {
			Scanner scanner = new Scanner(System.in);
			n = scanner.nextInt();
			for (int i = 0; i < n; i++) {
				for (int j = 0; j <= i; j++) {
					D[i][j] = scanner.nextInt();
				}
			}
			for (int i = 0; i < n; i++) {
				dp[n - 1][i] = D[n - 1][i]; //最后一层 手动赋值
			}
			System.out.println(maxNum());
		}
	// 列出Dp方程， dp[i][j] = dp[i + 1][j] + dp[i + 1][j + 1] 
		public static int maxNum () {
			for (int i = n - 2; i >= 0; i--) {
				for (int j = 0; j <= i; j++) {
					dp[i][j] = Math.max(dp[i + 1][j], dp[i + 1][j + 1]) + D[i][j];
				}
			}
			return dp[0][0];
		}
}
```
---

`最大字段和的 贪心算法，和 DP算法`
```java

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int[] nums = {-1,-2,5,6,-20,100};
		System.out.println(maxSum2(nums));
	}
	// Greedy
	public static int maxSum(int[] nums){
		int sum = 0;
		int b = Integer.MIN_VALUE;
		for (int i = 0; i < nums.length; i++){
			if (b > 0){
				b += nums[i];
			}
			else {
				b = nums[i];
			}
			if (b > sum){
				sum  = b;
			}
		}
		return sum;
	}
	// Dp 
	public static int maxSum2(int[] nums){
		int n = nums.length;
		int[] DP = new int[n];
		DP[0] = nums[0];
		int max = nums[0];
		
		for (int i = 1; i < n; i++) {
			DP[i] = nums[i]  + (DP[i - 1] > 0 ? DP[i - 1] : 0);
			max = Math.max(max, DP[i]);
		}
		return max;
	}
```

## 链家网笔试， 字符串最小变换问题
```java
public class 字符串变换 {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println(editDistance("112", "012"));
	}
	public static int editDistance (String s1, String s2) {
		int len1 = s1.length();
		int len2 = s2.length();
		int[][] dp = new int[len1 + 1][len2 + 1];
		for (int i = 1; i <= len1; i++) {
			dp[i][0] = i; // 把长度为i的字符串变为空串
		}
		for (int j = 1; j <= len2; j++) {
			dp[0][j] = j; // 把空川 变为长度为j的字符串
		}
		
		for (int i = 1; i <= len1; i++) {
			for (int j = 1; j <= len2; j++) {
				// 边界条件, s1[i] == s2[j],
				if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
					dp[i][j] = dp[i - 1][j - 1];
				}
				else {
					dp[i][j] = Math.min(dp[i - 1][j - 1] + 1, Math.min(dp[i - 1][j] + 1, dp[i][j - 1] + 1));
					
				}
			}
		}
		return dp[len1][len2];
		
	}

}
```
## leetcode718_LongestSubstring， 其实和最长公共字符串是差不多的
	 public static  int findLength(int[] A, int[] B) {
		 //DP 思想 same idea of Long Common Substring
		 
		 int len1 = A.length;
		 int len2 = B.length;
		 int[][] dp = new int[len1 + 1][len2 + 1];
		 int max = 0;
		 for (int i = 1;  i<= A.length; i++){
			 for (int j = 1; j <= B.length; j++){
					if (A[i - 1] == B[j - 1]){
						dp[i][j] = dp[i - 1][j - 1] + 1;
						max = Math.max(max, dp[i][j]);
					}
				}
		 }
		return max;

	 }

## Unique Paths I 
给定一个二维数组， 从左上角走到右下角的路线数
穷举法的话，一般会超时
```java
 public  int res = 0;
 public  int uniquePaths(int m, int n) {
	 search(m, n, 1, 1);
	 return res;
 }
 // 这样搜索的话，一下子就超时了
 public  void search(int m, int n, int x, int y) {
	 if (x == m && y == n) {
		 res++;
	 }
	 if ((x + 1) <= m) {
		 search(m, n, x + 1, y);
	 }
	 if ((y + 1) <= n) {
		 search(m, n, x, y + 1);
	 }
 }
```
构造DP数组，`dp[i][j] = dp[i][j - 1] + dp[i - 1][j]`是显然的
```java
 public static int search2(int m, int n) {
	 int[][] dp = new int[m][n];
	 dp[0][0] = 1;
	 for (int i = 1; i < m; i++) {
		 dp[i][0] = 1;
	 }
	 for (int i = 1; i < n; i++) {
		 dp[0][i] = 1;
	 }
	 for (int i = 1; i < m; i++) {
		 for (int j = 1; j < n; j++) {
			 dp[i][j] = dp[i][j - 1] + dp[i - 1][j];
		 }
	 }
	 return dp[m - 1][n - 1];
 }
```
## Unique Paths II
和Path I相比，如果在二维数组的中间，放上一些障碍物的话，那么就不能再从这个点过了。抽象为`DP[i][j] = 0`,
```java
 public static int uniquePathsWithObstacles(int[][] obstacleGrid) {
	 int m = obstacleGrid.length;
	 int n = obstacleGrid[0].length;
	 int[][] dp = new int[m][n];
	 if (obstacleGrid[0][0] == 1) {
		 dp[0][0] = 0;
	 }
	 else {
		 dp[0][0] = 1;
	 }
//		 for (int i = 0; i < m; i++) {
//			 for (int j = 0; j < n; j++) {
//				 // 障碍物， 直接DP[i][j] = 0
//				 if (obstacleGrid[i][j] == 1) {
//					 dp[i][j] = 0;
//				 }
//			 }
//		 }
	 for (int i = 1; i < m; i++) {
		 if (obstacleGrid[i][0] == 1) {
			 dp[i][0] = 0;
		 }
		 else {
			 dp[i][0] = dp[i - 1][0];
		 }
	 }
	 for (int i = 1; i < n; i++) {
		 if (obstacleGrid[0][i] == 1) {
			 dp[0][i] = 0;
		 }
		 else {
			 dp[0][i] = dp[0][i - 1];
		 }
	 }
	 for (int i = 1; i < m; i++) {
		 for (int j = 1; j < n; j++) {
			 if (obstacleGrid[i][j] == 1) {
				 dp[i][j] = 0;
			 }
			 else {
				 dp[i][j] = dp[i][j - 1] + dp[i - 1][j];
			 }
		 }
	 }
	return dp[m - 1][n - 1];
 }
 ```

