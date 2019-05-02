---
title: leetcode801_Mininum_SwapMakesSequencesIncreasing
date: 2018-3-20 14:18:40
categories:
	- leetcode
tags:
	- leetcode
---
We have two integer sequences A and B of the same non-zero length.

We are allowed to swap elements A[i] and B[i].  Note that both elements are in the same index position in their respective sequences.

At the end of some number of swaps, A and B are both strictly increasing.  (A sequence is strictly increasing if and only if A[0] < A[1] < A[2] < ... < A[A.length - 1].)

Given A and B, return the minimum number of swaps to make both sequences strictly increasing.  It is guaranteed that the given input always makes it possible.

	Example:
	Input: A = [1,3,5,4], B = [1,2,3,7]
	Output: 1
	Explanation: 
	Swap A[3] and B[3].  Then the sequences are:
	A = [1, 3, 5, 7] and B = [1, 2, 3, 4]
	which are both strictly increasing.
意思就是说， 给两个长度相等的数组，可以交换相同index位置的元素，使两个数组都递增有序，求最小的交换数。 DP是毫无疑问的。

1. 我们假定`n1`是 `n-1`处，没有交换就有序的花费,`s1`是`n-1`处交换之后的花费，我们现在要依靠 `n1`,`s1`来推测 `n2`, `s2`的花费。
我们可以假设，`a1 = A[i-1], b1 = B[i-1]` ||`a2 = A[i], b2 = B[i]`.
如果 `a1 < a2` && `b1 < b2` 很显然，`n2 = min(n1,n2),s2 = min(s2, s1 + 1)`, 因为 s2 总是代表交换之后的花费。  
2. `a1 < b2 &&  b1 < a2` `n2 = min(n2,s1),s2 = min(s2, n1 + 1)`, 写出算法如下：

---

```java
public static int minSwap(int[] A, int[] B) {
	int n1 = 0; //  ni表示  n(i-1)不需要交换就已经是有序的了
	int s1 = 1; // si 代表s(i-1) 在i - 1处交换一次就有序了
	// 那么 以上推测， n1 = 0 s1 = 1 很明显，index 0处肯定是有序的
	// index o 处 交换一次肯定也是有序的， 因为他们是第一个元素嘛
	
	for (int i = 1; i < A.length; i++) {
		// 以下是推测 n2 s2的过程
		int n2 = Integer.MAX_VALUE;
		int s2 = Integer.MAX_VALUE;
		// 
		if (A[i - 1] < A[i] && B[i - 1] < B[i]) {
			n2 = Math.min(n2, n1);
			s2 = Math.min(s2, s1 + 1);
		}
		if (A[i - 1] < B[i] && B[i - 1] < A[i]) {
			n2 = Math.min(n2, s1);
			s2 = Math.min(s2, n1 + 1);
		}
		
		n1 = n2;
		s1 = s2;
	}
	return Math.min(n1, s1);
}
```
`python`

```python
def minSwap(self, A, B):
    """
    :type A: List[int]
    :type B: List[int]
    :rtype: int
    """
    n = len(A)
    pre = [0, 1]
    for i in range(1, n):
        cur = [sys.maxsize, sys.maxsize]
        if A[i]>A[i-1] and B[i]>B[i-1]:
            cur[0] = min(cur[0], pre[0])
            cur[1] = min(cur[1], pre[1]+1)
        if A[i]>B[i-1] and B[i]>A[i-1]:
            cur[0] = min(cur[0], pre[1])
            cur[1] = min(cur[1], pre[0]+1)
        pre = cur
        return min(pre)
```