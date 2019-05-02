---
title: LeetcodeContest89
date: 2018-6-16 12:43:00
categories:
	- leetcode
tags:
	- Algorithm
---
摘要: 排名，第785
<!-- more -->

## 1 山脉顶峰索引
我们把符合下列属性的数组 A 称作山脉：
    `A.length >= 3`
    存在 `0 < i < A.length - 1 `使得`A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1]`

给定一个确定为山脉的数组，返回任何满足 `A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1] `的` i `的值。

It is a joke.
```java
class Solution {
    public int peakIndexInMountainArray(int[] A) {
        Map<Integer, Integer> map = new HashMap<>();
        int max = Integer.MIN_VALUE;
        for (int i = 0; i < A.length; i++) {
        	map.put(A[i], i);
        	max = Math.max(max, A[i]);
        }
        return map.get(max);
    }
}
```
## 2 车队
`N`  辆车沿着一条车道驶向位于 target 英里之外的共同目的地。

每辆车 `i `以恒定的速度 `speed[i] `（英里/小时），从初始位置` position[i]` （英里） 沿车道驶向目的地。

一辆车永远不会超过前面的另一辆车，但它可以追上去，并与前车以相同的速度紧接着行驶。

此时，我们会忽略这两辆车之间的距离，也就是说，它们被假定处于相同的位置。

车队 是一些由行驶在相同位置、具有相同速度的车组成的非空集合。注意，一辆车也可以是一个车队。

即便一辆车在目的地才赶上了一个车队，它们仍然会被视作是同一个车队。
会有多少车队到达目的地?
`示例：`

输入：`target = 12, position = [10,8,0,5,3], speed = [2,4,1,1,3]`
输出：`3`
解释：
从 `10` 和` 8 `开始的车会组成一个车队，它们在 `12 `处相遇。
从` 0 `处开始的车无法追上其它车，所以它自己就是一个车队。
从 `5 `和 `3 `开始的车会组成一个车队，它们在 `6` 处相遇。
请注意，在到达目的地之前没有其它车会遇到这些车队，所以答案是 3。

### 解释
1. 我们可以先把位置position[i], speed[i]做一次hash映射。
2. position数组排序，排序之后从尾部开始计算车队数目。
3. 如何判断其他的车辆能够追上第一辆top车呢? (target - position[top])*1.0/top_speed*upSpeed >= (float)(position[top] - position[i])) 
能够一起到达的车辆归于一个车队，设置标志位visit = true
4. 尾部—》 头部遍历，遍历标志位false的车辆，计算出所有的车队。
`java`代码
```java
class Solution {
    public int carFleet(int target, int[] position, int[] speed) {
      Map<Integer, Integer> map = new HashMap<>();
		for (int i = 0; i < position.length; i++) {
			map.put(position[i], speed[i]);
		}
		int size_left = position.length;
		boolean[] visit = new boolean[size_left];
		Arrays.sort(position);
		int count = 0;
		while (size_left > 0) {
			int top = Integer.MAX_VALUE;
			for (int i = position.length - 1; i >= 0; i--) {
				if (!visit[i]) {
					top = i;
					size_left--;
					break;
				}
			}
			visit[top] = true; //
			int top_speed = map.get(position[top]);
			for (int i = top - 1; i >= 0; i--) {
				if (!visit[i] && map.get(position[i]) > top_speed) {
					int left = target - position[top];
					int upSpeed = map.get(position[i]) - top_speed;
					if ((target - position[top])*1.0/top_speed*upSpeed >= (float)(position[top] - position[i])) {
						visit[i] = true;
						size_left--;
					}
				}
			}
			count++;
		}
		return count;
		
    }
}
```
## 考场就坐， 不会写
855. 考场就座
    用户通过次数 7
    用户尝试次数 22
    通过次数 7
    提交次数 43
    题目难度 Medium

在考场里，一排有 N 个座位，分别编号为 0, 1, 2, ..., N-1 。

当学生进入考场后，他必须坐在能够使他与离他最近的人之间的距离达到最大化的座位上。如果有多个这样的座位，他会坐在编号最小的座位上。(另外，如果考场里没有人，那么学生就坐在 0 号座位上。)

返回 ExamRoom(int N) 类，它有两个公开的函数：其中，函数 ExamRoom.seat() 会返回一个 int （整型数据），代表学生坐的位置；函数 ExamRoom.leave(int p) 代表坐在座位 p 上的学生现在离开了考场。请确保每次调用 ExamRoom.leave(p) 时都有学生坐在座位 p 上。
示例：

输入：["ExamRoom","seat","seat","seat","seat","leave","seat"], [[10],[],[],[],[],[4],[]]
输出：[null,0,9,4,2,null,5]
解释：
ExamRoom(10) -> null
seat() -> 0，没有人在考场里，那么学生坐在 0 号座位上。
seat() -> 9，学生最后坐在 9 号座位上。
seat() -> 4，学生最后坐在 4 号座位上。
seat() -> 2，学生最后坐在 2 号座位上。
leave(4) -> null
seat() -> 5，学生最后坐在 5 号座位上。


## 4 给两个字符串求出 A字符串最小的交换次数，变为B字符串
`Example 1:`  
Input: A = "ab", B = "ba"
Output: 1

`Example 2:`  

Input: A = "abc", B = "bca"
Output: 2  

```java
package leetcodeContest89;
/**
* @author 作者 : coderlong
* @version 创建时间：2018年6月17日 下午1:27:11
* 类说明: 
*/
public class leetcode854_K_Similar_Strings {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		String A = "abac";
		String B = "baca";
		System.out.println(kSimilarity(A, B));
	}
	public static int kSimilarity(String A, String B) {
		char[] a = A.toCharArray();
		char[] b = B.toCharArray();
		return search(a, b, 0, 0); 
	}
	// 简单的搜索问题，也可以BFS 
	public static int search(char[] now, char[] target, int i, int swap) {
		// search 到了最后
		if (i >= target.length) {
			return swap;
		}
		if (now[i] == target[i]) {
			return search(now, target, i + 1, swap);
		}
		int min = Integer.MAX_VALUE;
		for (int j = i + 1; j < target.length; j++) {
			if (now[j] != target[i]) { // 
				continue; // 继续search
			}
			//在i 后找到了第一个 == target[i]的char
			swap(now, i, j);
			min = Math.min(min, search(now, target, i + 1, swap + 1)); // 交换了一次
			swap(now, j, i);
		}
		return min;
	}
	public static void swap(char[] a, int i, int j){
		char temp = a[i];
		a[i] = a[j];
		a[j] = temp;
	}

}
```
做一次广度优先搜索也可以得出答案。
```java
class Solution {
   public int kSimilarity(String A, String B) {
		HashSet<String> visited = new HashSet<>();

		Queue<String> queue = new LinkedList<>();
		queue.offer(A);
		visited.add(A);

		int steps = 0;
		while (!queue.isEmpty()) {
			int size = queue.size();
			for (int i = 0; i < size; i++) {
				String node = queue.poll();
				//visit(node)
				if (node.equals(B))
					return steps;

				//extend new node
				int j = 0, k = 0;
				for (j = 0; j < node.length() && node.charAt(j) == B.charAt(j); j++)
					;
				for (k = j + 1; k < node.length(); k++)
					if (node.charAt(k) != B.charAt(k) && B.charAt(j) == node.charAt(k)) {
						String nextNode = swap(node, j, k);
						if (visited.add(nextNode))
							queue.add(nextNode);
					}
			}
			steps++;
		}
		return 0;
	}

	private String swap(String s, int j, int k) {
		StringBuilder sb = new StringBuilder(s);
		char c = sb.charAt(j);
		sb.setCharAt(j, sb.charAt(k));
		sb.setCharAt(k, c);
		return sb.toString();
	}
```