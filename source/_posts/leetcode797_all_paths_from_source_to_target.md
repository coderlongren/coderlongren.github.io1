---
title: leetcode797_all_paths_from_source_to_target
date: 2018-1-11 14:48:38
categories:
	- leetcode
tags:
	- leetcode
---

Given a directed, acyclic graph of `N` nodes.  Find all possible paths from node `0` to node `N-1`, and return them in any order.

The graph is given as follows:  the nodes are 0, 1, ..., graph.length - 1.  graph[i] is a list of all nodes j for which the edge (i, j) exists.

	Example:
	Input: [[1,2], [3], [3], []] 
	Output: [[0,1,3],[0,2,3]] 
	Explanation: The graph looks like this:
	0--->1
	|    |
	v    v
	2--->3
	There are two paths: 0 -> 1 -> 3 and 0 -> 2 -> 3


刚开始根本没有思路，因为那个二维数组来表示图就没看懂，
告诉自己，还是要学好英文啊！！！

`graph[i] is a list of all nodes j for which the edge (i, j) exists`
每个list代表的是edge(i,j)存在， 那就是有一条路径i -- > j喽
参考二叉树路径，回溯解决
```java
public class leetcode797_all_paths_from_source_to_target {
	 // 让我们用回溯法 来解决此题
	 public static List<List<Integer>> allPathsSourceTarget(int[][] graph) {
	        List<List<Integer>> res = new ArrayList<>();
	        if (graph == null) {
	        	return res;
	        }
	        ArrayList<Integer> list = new ArrayList<>();
	        backtrack(res, list, 0, graph.length - 1,graph);
	        return res;
	  }
	 public static void backtrack(List<List<Integer>> res, ArrayList<Integer> list, int start, int end,int[][] graph) {
		 // 找打了一条到达终点的路径
		 if (start == end) {
			 list.add(start);
			 res.add(new ArrayList<>(list));
			 list.remove(list.size() - 1); 
			 return;
		 }
		 for (int next:graph[start]) {
			 list.add(start);
			 backtrack(res, list, next, end, graph);
			 list.remove(list.size() - 1);
		 }
	 }
	 public static void main(String[] args) {
		int[][] graph = {{1,2},{3},{3},{}};
		System.out.println(allPathsSourceTarget(graph));
		//[[0, 1, 3], [0, 2, 3]]
	}
}

```

`Python`代码

```python
class Solution(object):
    def allPathsSourceTarget(self, graph):
        """
        :type graph: List[List[int]]
        :rtype: List[List[int]]
        """
        
        res = []
        dic = collections.defaultdict(list)
        for index in range(len(graph)-1):
            if len(graph[index]) > 1:
                for each in graph[index]:
                    #print each
                    dic[each].append(index)
            else:
                dic[graph[index][0]].append(index)
                
        def dfs(node, temp):
            if temp and temp[-1] == 0:
                res.append((list(temp[::-1])))
                return
            for n in dic[node]:
                temp.append(n)
                dfs(n,temp)
                temp.pop()
        dfs(len(graph)-1, [len(graph)-1])
        return res
```

