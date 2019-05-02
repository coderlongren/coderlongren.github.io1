---
title: Sliding Window Algorithm
date: 2018-1-14 15:18:40
categories:
    - Algorithm
tags:
	- Algorithm
	- java
	- 经典算法
---

## Partition Labels
A string S of lowercase letters is given. We want to partition this string into as many parts as possible so that each letter appears in at most one part, and return a list of 
integers representing the size of these parts. 
*Example 1:*  

```
Input: S = "ababcbacadefegdehijhklij"
Output: [9,7,8]
Explanation:
The partition is "ababcbaca", "defegde", "hijhklij".
This is a partition so that each letter appears in at most one part.
A partition like "ababcbacadefegde", "hijhklij" is incorrect, because it splits S into less parts.

```

*Note:*  
	S will have length in range [1, 500].  
	S will consist of lowercase letters ('a' to 'z') only.
把一个字符串分割成，几部分，要求：分割的个数尽可能多，每部分都是独立的  
`独立的`:
除了这个部分外，字符串其他部分不包含这个窗口里面的字符。
```
public static void main(String[] args) {
		// TODO Auto-generated method stub
		String S = "ababcbacadefegdehijhklij";
		System.out.println(partitionLabels(S));
	}
	 public static List<Integer> partitionLabels(String S) {
	        List<Integer> res = new ArrayList<>();
	        
	        int[] table = new int[26];
	        char[] stc = S.toCharArray();
	        for(char c:stc)// 记录C 在整个String里面出现的次数
	            table[c-'a']++;
	        
	        int i = 0, j = 0, l = S.length(), counter = 0;
	        HashSet<Character> hs = new HashSet<>();
	        while(j < l){
	            char c = stc[j];
	            if(!hs.contains(c)){//  count记录，   除了窗口里面的字符，外面都不包含这个窗口里面的字符，也就是说这个窗口是独立于整个字符串的
	                hs.add(c);
	                counter++;
	            }
	            table[c-'a']--; // 
	            j++;
	            if(table[c-'a'] == 0){ // table[c - 'a'] == 0 也就意味着窗口已经包含字符串里面所有的这个字符 c
	                counter--;
	                hs.remove(c);
	            }
	            if(counter == 0){// 如果count，说明整个窗口是独立于整个字符串的 
	                res.add(j - i);// 把这个窗口的长度记录到res
	                i = j;// 重新选择一个窗口，也就是重置起点   i坐标
	            }            
	        } 
	        return res;
	    }
```
