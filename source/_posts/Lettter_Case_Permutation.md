---
title:  784. Letter Case Permutation
date: 2018-3-4 21:18:40
categories:
	- Algorithm
tags:
	- Algorithm

---
Given a string S, we can transform every letter individually to be lowercase or uppercase to create another string.  Return a list of all possible strings we could create.
`
Examples:
Input: S = "a1b2"
Output: ["a1b2", "a1B2", "A1b2", "A1B2"]

Input: S = "3z4"
Output: ["3z4", "3Z4"]

Input: S = "12345"
Output: ["12345"]
`
abc  
abc Abc 0  
abc aBc Abc ABc 1  
abc abC aBc aBC Abc AbC ABc ABC 2  
---

Let's go!
```java
// BFS solution
	public static List<String> letterCasePermutation(String S) {
//		Set<String> set = new HashSet<>();
		LinkedList<String> list = new LinkedList<>();
		char[] chars = S.toCharArray();
		list.add(S);
		for (int i = 0; i < S.length(); i++) {
			if (Character.isDigit(S.charAt(i))) {
				continue;
			}
			int size = list.size();
			for (int j = 0; j < size; j++) {
				String cur = list.get(j);
				char[] temp = cur.toCharArray();
				temp[i] = Character.toLowerCase(temp[i]);
				list.addLast(String.valueOf(temp));
				temp[i] = Character.toUpperCase(temp[i]);
				list.addLast(String.valueOf(temp));
			}
		}
		Set<String> set = new HashSet<>(list);
		return new LinkedList<>(set);
	}
```


```java
	// DFS solution
	  public static List<String> letterCasePermutation2(String S) {
		  if (S == null) {
			  return new LinkedList<>();
		  }
		  List<String> res = new LinkedList<>();
		  helper(S,res,0);
		  return res;
	  }
	
	private static void helper(String s, List<String> res, int pos) {
		if (pos == s.length()) {
			res.add(s);
			return;
		}
		// 是数字
		if (s.charAt(pos) >= '0' && s.charAt(pos) <= '9') {
			helper(s, res, pos + 1);
			return;
		}
		char[] chs = s.toCharArray();
        chs[pos] = Character.toLowerCase(chs[pos]);
        helper(String.valueOf(chs), res, pos + 1);
        
        chs[pos] = Character.toUpperCase(chs[pos]);
        helper(String.valueOf(chs), res, pos + 1);
	}
```
