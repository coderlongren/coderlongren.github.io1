---
title: leetcode207_Course Schedule
date: 2018-4-13 21:18:40
categories:
	- Algorithm
tags:
	- Algorithm
---
原题：  
 There are a total of n courses you have to take, labeled from `0` to `n - 1`.

Some courses may have prerequisites, for example to take course 0 you have to first take course 1, which is expressed as a pair: `[0,1]`

Given the total number of courses and a list of prerequisite pairs, is it possible for you to finish all courses?

For example:
`2, [[1,0]]`  
There are a total of 2 courses to take. To take course 1 you should have finished course 0. So it is possible.

`2, [[1,0],[0,1]]`  

There are a total of 2 courses to take. To take course 1 you should have finished course 0, and to take course 0 you should also have finished course 1. So it is impossible.

---
给多个课程， 还有一个课程的学习数组，数组中为二维， [i,j]代表学习i课程之前，必须先学习j课程，即在图中j为i的前继节点。
  
1. 这道题可以等价于有向图中是否存在环路问题，如果没有环路，那么就有可能学习完所有的课程.  
2. 也可以抽象成一个拓扑排序问题，
3. 使用DFS
4. 使用BFS

我们可以使用一个数组， 来记录每门课程一共多少们前继课程，然后使用一个队列，把那些没有前继课程的课程号，入队，遍历队列，循环检查课程二维数组中每个数组，前继节点是否是队列中的元素，如果是的话，课程数组数减一。  
当然，如果在遍历中，一个课程的前继课程数为0了的话，就把这个课程再次加入到队列中去。

Java代码如下：  
```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
	 if (prerequisites == null) {
		 return false;
	 }
     // 至少知道如何利用这个 课程数量 numCourses
	 // 记录每个课程的，preRequires
	 int[] courses = new int[numCourses];
	 int len = prerequisites.length;
	 if  (numCourses == 0 || len == 0) {
		 return true;
	 }
	 for (int i = 0; i < len; i++) {
		 // 假设 prerequisites[1][0] 就说明 1 号课程需要有前继课程, 
		 courses[prerequisites[i][0]]++; 
	 }
	 Queue<Integer> queue = new LinkedList<>(); 
	 for (int i = 0; i < numCourses; i++) {
		 // 说明 i 号课程 没有前继课程 i课程可以直接学习
		 if (courses[i] == 0) {
			 queue.offer(i);
		 }
	 }
	 
	 // 下面在一一取出队列中，可以直接学习的课程 对prerequisites 判断
	 int numNoPre = queue.size();
	 while (!queue.isEmpty()) {
		 int oneNoPrecourse = queue.poll();
		 for (int i = 0; i < len; i++) {
			 // 这个课程如果是 某个课程的前继课程
			 if (prerequisites[i][1] == oneNoPrecourse) {
				 courses[prerequisites[i][0]]--;
				 if (courses[prerequisites[i][0]] == 0) {
					 numNoPre++;
					 queue.offer(prerequisites[i][0]);
				 }
			 }
		 }
	 }
	 // 如果没有前继节点的课程数 == 所有课程数
	 if (numNoPre == numCourses) {
		 return true;
	 }
	 else {
		 return false;
	 }
 }
```

--- 
再前进一步，我们进入leetcode210. Course Schedule II
There are a total of n courses you have to take, labeled from `0` to` n - 1`.

Some courses may have prerequisites, for example to take course 0 you have to first take course 1, which is expressed as a pair: `[0,1]`

Given the total number of courses and a list of prerequisite pairs, return the ordering of courses you should take to finish all courses.

There may be multiple correct orders, you just need to return one of them. If it is impossible to finish all courses, return an empty array.

For example:

`2, [[1,0]]`  

There are a total of 2 courses to take. To take course 1 you should have finished course 0. So the correct course order is [0,1]

`4, [[1,0],[2,0],[3,1],[3,2]]`  

There are a total of 4 courses to take. To take course 3 you should have finished both courses 1 and 2. Both courses 1 and 2 should be taken after you finished course 0. So one correct course order is `[0,1,2,3]`. Another correct ordering is`[0,2,1,3]`. 
原题意思是差不多的，只不过要求的结果不一样了，这次是要求出能够完成所有课程学习的序列。那么这个问题就比上一个问题还复杂一些了，







