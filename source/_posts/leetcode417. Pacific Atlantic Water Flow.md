---
title: 417. Pacific Atlantic Water Flow
date: 2018-4-27 16:18:40
categories:
	- Algorithm
tags:
	- Algorithm
	- leetcode
---

**原题目:**
Given an m x n matrix of non-negative integers representing the height of each unit cell in a continent, the "Pacific ocean" touches the left and top edges of the matrix and the "Atlantic ocean" touches the right and bottom edges.

Water can only flow in four directions (up, down, left, or right) from a cell to another one with height equal or lower.

Find the list of grid coordinates where water can flow to both the Pacific and Atlantic ocean.

**Note:**

    The order of returned grid coordinates does not matter.
    Both m and n are less than 150.

**Example:**

Given the following 5x5 matrix:

  Pacific ~   ~   ~   ~   ~ 
       ~  1   2   2   3  (5) *
       ~  3   2   3  (4) (4) *
       ~  2   4  (5)  3   1  *
       ~ (6) (7)  1   4   5  *
       ~ (5)  1   1   2   4  *
          *   *   *   *   * Atlantic

**Return:**

[[0, 4], [1, 3], [1, 4], [2, 2], [3, 0], [3, 1], [4, 0]] (positions with parentheses in above matrix).

题目说，在每个位置**matrix[x][y]** 代表这个点的高度， 每个点可以流向等于或者低于自己高度的点。
上面，左面代表 Pacific, 右面，下面代表Atlantic ，求这个matrix里面可以同时流向太平洋，大西洋的坐标集。
一般看到就会想到DFS的解法，我第一次理解错了题意，结果遍历matrix里的每个点， 看是否他能到达两个海洋。最后，超时，思考更完善的算法。
`Java BFS `
```java
public class Solution {
	int[][] dics = new int[][]{{1,0}, {0,1},{-1,0}, {0,-1}};
	public static void main(String[] args) {
		// TODO Auto-generated method stub

	}
	//  这道题明显是可以使用 BFS 或者DFS 来做的，
	// 第一种我才用 广搜
	public List<int[]> pacificAtlantic(int[][] matrix) {
		List<int[]> res = new ArrayList<>();
		if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
			return res;
		}
		int rows = matrix.length;
		int cols = matrix[0].length;
		boolean[][] pacific =  new boolean[rows][cols]; // 记录 可以到达 pacific
		boolean[][] atlantic = new boolean[rows][cols]; // 记录可以到达 atlantic
		Stack<int[]> visitPac = new Stack<>();
		Stack<int[]> visitAtlan = new Stack<>();
		for (int i = 0; i < rows; i++) {
			pacific[i][0] = true;
			visitPac.push(new int[]{i,0});
			atlantic[i][cols - 1] = true;
			visitAtlan.push(new int[]{i,cols - 1});
		}
		for (int j = 0; j < cols; j++) {
			pacific[0][j] = true;
			visitPac.push(new int[]{0,j});
			atlantic[rows - 1][j] = true;
			visitAtlan.push(new int[]{rows - 1, j});
		}
		bfs(matrix, visitPac, pacific);
		bfs(matrix, visitAtlan, atlantic);
		for (int i = 0; i < rows; i++) {
			for (int j = 0; j < cols; j++) {
				if (pacific[i][j] && atlantic[i][j]) {
					res.add(new int[]{i,j});
				}
			}
		}
		return res;
	}
	public void bfs(int[][] matrix, Stack<int[]> stack, boolean[][] visited) {
		int rows = matrix.length;
		int cols = matrix[0].length;
		while (!stack.isEmpty()) {
			int[] idx = stack.pop();
			for (int[] dic:dics) {
				int x = idx[0] + dic[0];
				int y = idx[1] + dic[1];
				if (x < 0 || x >= rows || y < 0 || y >= cols || visited[x][y] || matrix[x][y] < matrix[idx[0]][idx[1]]) {
					continue;
				}
				visited[x][y] = true;
				stack.push(new int[]{x,y});
			}
		}
	}
}
```
后来看到别人提交的DFS版本的答案，DFS 更加好理解。
```java
import java.util.ArrayList;
import java.util.List;

/**
* @author 作者 : coderlong
* @version 创建时间：2018年4月27日 下午2:59:28
* 类说明: 
*/
public class leetcode417_Pactic_Atlantic_water_Flow2 {
	public int[][] dics = {{1,0}, {-1,0}, {0,1}, {0,-1}};
	public static void main(String[] args) {
		// TODO Auto-generated method stub

	}
	public List<int[]> pacificAtlantic(int[][] matrix) {
		List<int[]> res = new ArrayList<>();
		if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
			return res;
		}
		int rows = matrix.length;
		int cols = matrix[0].length;
		
		boolean[][] pacific = new boolean[rows][cols];
		boolean[][] atlantic = new boolean[rows][cols];
		// 用Integer.MIN_VALUE 来访问 四周边界元素， 保证四周元素绝对可以流向visit海域
		for (int i = 0; i < rows; i++) {
			dfs(matrix, Integer.MIN_VALUE, i, 0, pacific);
			dfs(matrix, Integer.MIN_VALUE, i, cols - 1, atlantic);
		}
		for (int j = 0; j < cols; j++) {
			dfs(matrix, Integer.MIN_VALUE, 0, j, pacific);
			dfs(matrix, Integer.MIN_VALUE, rows - 1, j, atlantic);
		}
		for (int i = 0; i < rows; i++) {
			for (int j = 0; j < cols; j++) {
				if (pacific[i][j] && atlantic[i][j]) {
					res.add(new int[]{i,j});
				}
			}
		}
		return res;
	}
	public void dfs(int[][] matrix, int height, int x, int y, boolean[][]visit) {
		int rows = matrix.length;
		int cols = matrix[0].length;
		// x y 为坐标， 需先判断是否符合要求
		if (x <0 || x >= rows || y < 0 || y >= cols || visit[x][y] || matrix[x][y] < height) {
			return ;
		}
		visit[x][y] = true; // (x,y) 可以流向visit海域
		for (int[] cur : dics) {
			dfs(matrix, matrix[x][y], x + cur[0], y + cur[1], visit);
		}
	}

}
```

