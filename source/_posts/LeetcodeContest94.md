---
title: LeetcodeContest94
date: 2018-7-22 23:43:00
categories:
	- leetcode
tags:
	- Algorithm
---
摘要: 做的最简单的一次周比赛

<!-- more -->

## 872 Leaf-Similar Trees左相似的树

Consider all the leaves of a binary tree.  From left to right order, the values of those leaves form a leaf value sequence.
![](/images/tree5.png)
For example, in the given tree above, the `leaf value sequence` is `(6, 7, 4, 9, 8)`.

Two binary trees are considered leaf-similar if their leaf value sequence is the same.

Return true `if` and `only if` the two given trees with head nodes `root1` and `root2` are `leaf-similar`.

左相似的定义是只要叶子节点序列，从左往右是一样的就符合了。  
那么，我们就可以中序遍历访问整棵树，把所有的叶子节点保存起来，最后判断是否相同就OK了。
```java
public class leetcode872_Leaf_Similar_trees {

	public static List<Integer> list1 = new ArrayList<>();
	public static List<Integer> list2 = new ArrayList<>();

	public static boolean leafSimilar(TreeNode root1, TreeNode root2) {
		bottom_sequence(root1, list1);
		bottom_sequence(root2, list2);
		if (list1.size() == list2.size()) {
			for (int i = 0; i < list1.size(); i++) {
				if (list1.get(i) != list2.get(i)) {
					return false;
				}
			}
			return true;
		}
		return false;
		
    }
	public static void bottom_sequence(TreeNode root, List<Integer> list) {
		if (root == null) {
			return;
		}
		if (root.left == null && root.right == null) {
			list.add(root.val);
		}
		bottom_sequence(root.left, list);
		bottom_sequence(root.right, list);
	}
```
## 874Walking Robot Simulation 模拟机器人行走路线
A robot on an infinite grid starts at point (0, 0) and faces north.  The robot can receive one of three possible types of commands:

    `-2`: turn left 90 degrees
    `-1`: turn right 90 degrees
    `1 <= x <= 9`: move forward x units

Some of the grid squares are obstacles. 

The `i-th` obstacle is at grid point (`obstacles[i][0]`, obstacles[i][1])

If the robot would try to move onto them, the robot stays on the previous grid square instead (but still continues following the rest of the route.)

Return the square of the maximum Euclidean distance that the robot will be from the origin.
问题已经很明确，  
给定行走的路线一维数组，和障碍路二维数组。
对于commands:  
* -1 右旋转90度路线
* -2 左旋转90度路线
* 1-9 就按照机器人方向行走
但是在遇到障碍物的时候，立即停止，直到下一个可执行的command到来。最后求，机器人所能到达的最远的 `max = i * i + j * j`

我在这里用了两个map来保存机器人的方向前进 度量dir, 和转向的map turn  
再把所有的obstacle用字符串拼接的方式保存起来，转向的时候，就正常转向，每次前进一个度量的时候，判断是否会遇到 障碍物，如果遇到了就停止command的循环，否则继续前进。max一直记录，所能到达的最远记录即可。

```java
class Solution {
    public int robotSim(int[] commands, int[][] obstacles) {
      	int max = 0;
        Map<Integer, int[]> dir = new HashMap<>();
        Map<Integer, int[]> turn = new HashMap<>();
        dir.put(0, new int[]{0,1});
        dir.put(1, new int[]{1,0});
        dir.put(2, new int[]{0,-1});
        dir.put(3, new int[]{-1,0});
        turn.put(0, new int[]{3,1});
        turn.put(1, new int[]{0,2});
        turn.put(2, new int[]{1,3});
        turn.put(3, new int[]{2,0});
        Set<String> set = new HashSet<>();
        for (int[] o : obstacles) {
        	set.add(o[0] + "+" + o[1]);
        }
        int face = 0;
        int[] start = new int[]{0,0};
        for (int command : commands) {
        	if (command == -2) {
        		face = turn.get(face)[0];
        	}
        	else if (command == -1) {
        		face = turn.get(face)[1];
        	}
        	else {
        		int len = command;
        		while (len > 0) {
        			int x = start[0] + dir.get(face)[0]; 
        			int y = start[1] + dir.get(face)[1];
        			if (!set.contains(x + "+" + y)) {
        				start[0] += dir.get(face)[0];
        				start[1] += dir.get(face)[1];
        				max = Math.max(max, start[0]*start[0] + start[1]*start[1]);
        			}
        			else {
        				break;
        			}
        			len--;
        		}
        	}
        }
        return max;
    }
}
```
## 875. Koko Eating Bananas 珂珂吃香蕉
Koko loves to eat bananas.  There are `N` piles of bananas, the `i-th` pile has `piles[i]` bananas.  The guards have gone and will come back in `H `hours.

Koko can decide her bananas-per-hour eating speed of `K`.  Each hour, she chooses some pile of bananas, and eats `K` bananas from that pile.  If the pile has less than `K` bananas, she eats all of them instead, and won't eat any more bananas during this hour.

Koko likes to eat `slowly`, but still wants to finish eating all the bananas before the guards come back.

Return the `minimum integer K `such that she can eat all the bananas within `H` hours.

```java
class Solution {
    public  int minEatingSpeed(int[] piles, int H) {
        int min = 1; // 最小，就是每小时吃一根把
        int max = 1; // 最大为， 每小时吃完最大的那堆香蕉
        for (int pile : piles) {
        	max = Math.max(max, pile);
        }
        while (min < max) {
        	int mid = (min + max)/2;
        	if (can_finished(piles, mid, H)) {
        		max = mid;
        	} else {
        		min = mid + 1;
        	}
        }
        return min;
    }
	// 每小时吃pir个香蕉，能在H小时内吃完所有的香蕉
	public  boolean can_finished(int[] piles, int pir, int H) {
		int sum = 0;
		int count = 0;
		for (int pile : piles) {
			if (pile <= pir) {
				count++;
			} 
			else {
				if (pile%pir == 0) {
					count += (pile/pir);
				}
				else {
					count += (pile/pir + 1);
				}
			}
			if (count > H) {
				return false;
			}
		}
		return count <= H;
	}
}
```
## 873. Length of Longest Fibonacci Subsequence 最长的fib序列

A sequence `X_1, X_2, ..., X_n `is fibonacci-like if:
    `n >= 3`
    `X_i + X_{i+1} = X_{i+2} for all i + 2 <= n`

Given a strictly increasing array A of positive integers forming a sequence, find the length of the longest fibonacci-like subsequence of A.  If one does not exist, return 0.

(Recall that a subsequence is derived from another sequence A by deleting any number of elements (including none) from A, without changing the order of the remaining elements.  For example, [3, 5, 8] is a subsequence of `[3, 4, 5, 6, 7, 8]`.)

 

Example 1:

Input: `[1,2,3,4,5,6,7,8]`
Output:` 5`
Explanation:
The longest subsequence that is fibonacci-like: `[1,2,3,5,8].`

Example 2:

Input: `[1,3,7,11,12,14,18]`
Output: `3`
Explanation:
The longest subsequence that is fibonacci-like:
`[1,11,12]`, `[3,11,14]` or` [7,11,18].`

simple brute force喽
```python
def lenLongestFibSubseq(self, A):
    """
    :type A: List[int]
    :rtype: int
    """
    max_ = 0
    
    for i in range(0, len(A)):
        if i > max_:
            return max_
        for j in range(i+1, len(A)):
            a = A[i]
            b = A[j]
            list_ = [a,b]
            for k in range(j+1, len(A)):
                c = a + b
                if c == A[k]:
                    a = b
                    b = c
                    list_.append(c)
                    max_ = max(max_, len(list_))
                if  A[k] > c:
                    break
    return max_
```

---
`Java Code `

```java
	public static int lenLongestFibSubseq(int[] A) {
        int max = 0;
//        Set<Integer> set = new HashSet<>();
//        for (int num : A) {
//        	set.add(num);
//        }
        for (int i = 0; i < A.length; i++) {
        	int start = A[i];
        	if (i > max) {
        		return max;
        	}
        	for (int j = i + 1; j < A.length; j++) {
        		List<Integer> list = new ArrayList<>();
        		int a = start;
        		int b = A[j];
        		list.add(a);
        		list.add(b);
        		for (int k = j + 1; k < A.length; k++) {
        			int c = a + b;
        			if (A[k] == c) {
        				list.add(c);
        				a = b;
        				b = c;
        				max = Math.max(max, list.size());
        			}
        		}
        	}
        }
        return max;
    }
```
