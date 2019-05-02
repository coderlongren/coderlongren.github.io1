---
title: LeetcodeContest-84
date: 2018-5-13 23:18:40
categories:
	- Algorithm
tags:
	- Algorithm
	- leetcode
---
摘要：Leetcode-84-Contest,一次愉快的比赛
<!-- more -->

## 832. Flipping an Image
Given a binary matrix `A`, we want to flip the image horizontally, then invert it, and return the resulting image.

To flip an image horizontally means that each row of the image is reversed.  For example, flipping `[1, 1, 0]` horizontally results in `[0, 1, 1]`.

To invert an image means that each 0 is replaced by 1, and each 1 is replaced by 0. For example, inverting `[0, 1, 1]` results in`[1, 0, 0]`.

* Example 1: * 

Input: `[[1,1,0],[1,0,1],[0,0,0]]`
Output: `[[1,0,0],[0,1,0],[1,1,1]]`
Explanation: First reverse each row: [[0,1,1],[1,0,1],[0,0,0]].
Then, invert the image: [[1,0,0],[0,1,0],[1,1,1]]

* Example 2:* 

Input: [[1,1,0,0],[1,0,0,1],[0,1,1,1],[1,0,1,0]]
Output: [[1,1,0,0],[0,1,1,0],[0,0,0,1],[1,0,1,0]]
Explanation: First reverse each row: [[0,0,1,1],[1,0,0,1],[1,1,1,0],[0,1,0,1]].
Then invert the image: [[1,1,0,0],[0,1,1,0],[0,0,0,1],[1,0,1,0]]

```python
class Solution(object):
    def flipAndInvertImage(self, A):
        # First reverse each row
        for i, row in enumerate(A):
            A[i] = row[::-1]
            # Then invert the image
            for j, item in enumerate(A[i]): # enumerate() is unnecessary here
                A[i][j] = 1 - A[i][j]
								
        return A
```

## 833. Find And Replace in String

To some string S, we will perform some replacement operations that replace groups of letters with new ones (not necessarily the same size).
Each replacement operation has 3 parameters: a starting index i, a source word x and a target word y.  The rule is that if x starts at position i in the original string S, then we will replace that occurrence of x with y.  If not, we do nothing.
For example, if we have S = "abcd" and we have some replacement operation i = 2, x = "cd", y = "ffff", then because "cd" starts at position 2 in the original string S, we will replace it with "ffff".
Using another example on S = "abcd", if we have both the replacement operation i = 0, x = "ab", y = "eee", as well as another replacement operation i = 2, x = "ec", y = "ffff", this second operation does nothing because in the original string S[2] = 'c', which doesn't match `x[0] = 'e'.`

All these operations occur simultaneously.  It's guaranteed that there won't be any overlap in replacement: for example, S = "abc", indexes = `[0, 1]`, sources = `["ab","bc"]` is not a valid test case.

* Example 1:* 

Input: S = "abcd", indexes = [0,2], sources = `["a","cd"]`, targets = `["eee","ffff"]`
Output: "eeebffff"
Explanation: "a" starts at index 0 in S, so it's replaced by "eee".
"cd" starts at index 2 in S, so it's replaced by "ffff".

* Example 2:*

Input: S = "abcd", indexes = `[0,2]`, sources = `["ab","ec"]`, targets =` ["eee","ffff"]`
Output: "eeecd"
Explanation: "ab" starts at index 0 in S, so it's replaced by "eee". 
"ec" doesn't starts at index 2 in the original S, so we do nothing.

```python
def findReplaceString(self, S, indexes, sources, targets):
        r = S
        offset = {}
        for i, ind in enumerate(indexes):
            if S[ind : ind + len(sources[i])] == sources[i]:
                off = 0
                for k in offset:
                    if ind >= k:
                        off += offset[k]
                r = r[:ind+off] + targets[i] +  r[ind+off+len(sources[i]):]
                offset[ind + len(sources[i])] = len(targets[i]) - len(sources[i])
        return r
```
---

`Java版本` 双Map就可以解决
```java
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		String S = "mhnbzxkwzxtaanmhtoirxheyanoplbvjrovzudznmetkkxrdmr";
//		"mhnbzxkwzxtaanmhtoirxheyanoplbvjrovzudznmetkkxrdmr"
//		[46,29,2,44,31,26,42,9,38,23,36,12,16,7,33,18]
//		["rym","kv","nbzxu","vx","js","tp","tc","jta","zqm","ya","uz","avm","tz","wn","yv","ird"]
//		["gfhc","uq","dntkw","wql","s","dmp","jqi","fp","hs","aqz","ix","jag","n","l","y","zww"]
		int[] indexes = {46,29,2,44,31,26,42,9,38,23,36,12,16,7,33,18};
		String[] sources = {"rym","kv","nbzxu","vx","js","tp","tc","jta","zqm","ya","uz","avm","tz","wn","yv","ird"};
		String[] targets = {"gfhc","uq","dntkw","wql","s","dmp","jqi","fp","hs","aqz","ix","jag","n","l","y","zww"};
		System.out.println(findReplaceString(S, indexes, sources, targets));
	}
	public static String findReplaceString(String S, int[] indexes, String[] sources, String[] targets) {
        if (S == null || S.length() == 0) {
        	return S;
        }
        StringBuilder res = new StringBuilder();
        int j = 0;
        int i = 0;
        if (indexes == null || indexes.length == 0) {
        	res = new StringBuilder(S);
        	return S;
        }
        Map<Integer, String> map1 = new HashMap<>();
        Map<Integer, String> map2 = new HashMap<>();
        
        for (int k = 0; k < indexes.length; k++) {
        	map1.put(indexes[k],sources[k]);
        	map2.put(indexes[k], targets[k]);
        }
        Arrays.sort(indexes);
        for (int k = 0; k < indexes.length; k++) {
        	sources[k] = map1.get(indexes[k]);
        	targets[k] = map2.get(indexes[k]);
        }
        
        
        while (i < S.length()) {
        	if (j < indexes.length && i == indexes[j] && S.startsWith(sources[j],i)) {
        		res.append(targets[j]);
        		i += sources[j].length();
        		j++;
        	}
        	else {
        		res.append(S.charAt(i));
        		i++;
        	}
        	
        }
		return res.toString();
    }

```
## 835 Image Overlap
Two images A and B are given, represented as binary, square matrices of the same size.  (A binary matrix has only 0s and 1s as values.)

We translate one image however we choose (sliding it left, right, up, or down any number of units), and place it on top of the other image.  After, the overlap of this translation is the number of positions that have a 1 in both images.

(Note also that a translation does not include any kind of rotation.)

What is the largest possible overlap?

Example 1:

Input: 
			A = [[1,1,0],
	            [0,1,0],
	            [0,1,0]]
	       B = [[0,0,0],
	            [0,1,1],
	            [0,0,1]]
Output: 3
Explanation: We slide A to right by 1 unit and down by 1 unit.

`python代码`
```python
def largestOverlap(self, A, B):
    N = len(A)
    LA = [i / N * 100 + i % N for i in xrange(N * N) if A[i / N][i % N]]
    LB = [i / N * 100 + i % N for i in xrange(N * N) if B[i / N][i % N]]
    c = collections.Counter(i - j for i in LA for j in LB)
    return max(c.values() or [0])
```
照例Java代码
```java
public int largestOverlap(int[][] A, int[][] B) {
    int N = A.length;
    List<Integer> LA = new ArrayList<>();
    List<Integer> LB = new ArrayList<>();
    HashMap<Integer, Integer> count = new HashMap<>();
    for (int i = 0; i < N * N; ++i) if (A[i / N][i % N] == 1) LA.add(i / N * 100 + i % N);
    for (int i = 0; i < N * N; ++i) if (B[i / N][i % N] == 1) LB.add(i / N * 100 + i % N);
    for (int i : LA) for (int j : LB)
            count.put(i - j, count.getOrDefault(i - j, 0) + 1);
    int res = 0;
    for (int i : count.values()) res = Math.max(res, i);
    return res;
 }
```