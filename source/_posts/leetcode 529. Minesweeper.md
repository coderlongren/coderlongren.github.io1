---
title: LeetCode529. Minesweeper
date: 2018-05-05 20:01:47
categories:
	- Algorithm
tags:
	- Algorithm
	- java
---
DFS && BFS 
<!-- more -->

[原题目连接](https://leetcode.com/problems/minesweeper/description/)
Let's play the minesweeper game (Wikipedia, online game)!

You are given a 2D char matrix representing the game board. 'M' represents an unrevealed mine, 'E' represents an unrevealed empty square, 'B' represents a revealed blank square that has no adjacent (above, below, left, right, and all 4 diagonals) mines, digit ('1' to '8') represents how many mines are adjacent to this revealed square, and finally 'X' represents a revealed mine.

Now given the next click position (row and column indices) among all the unrevealed squares ('M' or 'E'), return the board after revealing this position according to the following rules:

    If a mine ('M') is revealed, then the game is over - change it to 'X'.
    If an empty square ('E') with no adjacent mines is revealed, then change it to revealed blank ('B') and all of its adjacent unrevealed squares should be revealed recursively.
    If an empty square ('E') with at least one adjacent mine is revealed, then change it to a digit ('1' to '8') representing the number of adjacent mines.
    Return the board when no more squares will be revealed.

Example 1:

Input: 

	[['E', 'E', 'E', 'E', 'E'],
	 ['E', 'E', 'M', 'E', 'E'],
	 ['E', 'E', 'E', 'E', 'E'],
	 ['E', 'E', 'E', 'E', 'E']]

Click :` [3,0]`

Output: 

	[['B', '1', 'E', '1', 'B'],
	 ['B', '1', 'M', '1', 'B'],
	 ['B', '1', '1', '1', 'B'],
	 ['B', 'B', 'B', 'B', 'B']]

这是经典游戏扫雷的变形版，游戏初始化： `M`代表未揭开的雷，`E`代表没有揭开的空地 `EmptySquare`， `B`代表揭开后为空地 （board[x][y]四周都没有雷），数字 `1-8` 代表揭开的坐标处，四周的雷数目，M雷揭开后变为X(扫中雷)，否则什么也不做。
这里我给出了DFS，BFS版本的答案，其实都差不多了，这段时间搜索的题目做的好多了。基本都是只要剪枝操作考虑清楚，这种题目一般很容易做出来.

`Java版本`
```java
public int[][] dics = {{0,1}, {0,-1}, {1,0}, {-1,0}, {1,1}, {1,-1}, {-1,1}, {-1,-1}};
public char[][] updateBoard(char[][] board, int[] click) {
	if (board == null || board.length == 0 || board[0].length == 0) {
		return board; 
	}
	int rows = board.length;
	int cols = board[0].length;
	boolean[][] visit = new boolean[rows][cols];
	dfs(board, visit, click[0], click[1]);
	return board;
}
public void dfs(char[][] board, boolean[][] visit, int x, int y) {
	int rows = board.length;
	int cols = board[0].length;
	// 探测到了没有揭开的  雷
	if (board[x][y] == 'M') {
		board[x][y] = 'X';
		return;
	}
	// 已经被揭开了 非 M 非 E, 直接返回
	if (board[x][y] != 'E') {
		return;
	}
	visit[x][y] = true;
	// 揭开了一个 不是雷的空地， 向四周搜索
	int mines = 0; // 四周的雷数
	for (int[] dic : dics) {
		int x1 = x + dic[0];
		int y1 = y + dic[1];
		if (x1 < 0 || x1 >= rows || y1 < 0 || y1 >= cols) {
			continue; // 越界
		}
		if (board[x1][y1] == 'M') {
			mines++;
		}
	}
	if (mines == 0) {
		board[x][y] = 'B';
		// x, y 周围没有雷
		for (int[] dic : dics) {
			int x1 = x + dic[0];
			int y1 = y + dic[1];
			if (x1 < 0 || x1 >= rows || y1 < 0 || y1 >= cols || visit[x1][y1]) {
				continue; // 越界
			}
			dfs(board, visit, x1, y1);
		}
	}
	else {
		board[x][y] = (mines + "").charAt(0);
	}
}
```

---

`BFS版本`
```java
// 广度优先搜索
public char[][] updateBoard2(char[][] board, int[] click) {
	if (board == null || board.length == 0 || board[0].length == 0) {
		return board; 
	}
	int rows = board.length;
	int cols = board[0].length;
	boolean[][] visit = new boolean[rows][cols];
	Queue<int[]> queue = new LinkedList<>();
	queue.offer(new int[]{click[0], click[1]});
	while (!queue.isEmpty()) {
		int[] poll = queue.poll();
		int x = poll[0];
		int y = poll[1];
		// 探测到了没有揭开的  雷
		if (board[x][y] == 'M') {
			board[x][y] = 'X';
			continue;
		}
		// 已经被揭开了 非 M 非 E, 直接返回
		if (board[x][y] != 'E') {
			continue;
		}
		visit[x][y] = true;
		int mines = 0; // 雷数目
		for (int[] dic : dics) {
			int x1 = x + dic[0];
			int y1 = y + dic[1];
			if (x1 < 0 || x1 >= rows || y1 < 0 || y1 >= cols) {
				continue; // 越界
			}
			if (board[x1][y1] == 'M') {
				mines++;
			}
		}
		if (mines == 0) {
			board[x][y] = 'B';
			// x, y 周围没有雷
			for (int[] dic : dics) {
				int x1 = x + dic[0];
				int y1 = y + dic[1];
				if (x1 < 0 || x1 >= rows || y1 < 0 || y1 >= cols || visit[x1][y1]) {
					continue; // 越界
				}
				// 
				queue.offer(new int[]{x1,y1});
			}
		}
		else {
			board[x][y] = (mines + "").charAt(0);
		}
		
	}
	return board;
}
```
DFS打败了99.2% BFS只击败了 7.5%的 Submition