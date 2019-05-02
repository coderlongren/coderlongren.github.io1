---
title: Longest Substring Without Repeating Characters
date: 2017-10-24 22:58:49
categories:
	- Algorithm
tags:
	- Algorithm
---
摘要：Brute Force
<!-- More -->
正文:
![Sahara](/images/sahala.jpeg)
## Approach #1 Brute Force [Time Limit Exceeded]
第一眼看到 求最长的非重复子串，就是三个for循环 求解，去掉非法的判断，花了十分钟得到了一个不得使用“蛮力”，时间复杂度O（n*3）可是明明用了hashSet的Contain（）方法了，Exception代码如下:

	class Solution {
	        public static boolean allUnique(String string,int start,int end){
			
			Set<Character> set =new HashSet<Character>();
			for (int i = start; i < end; i++){
				if (set.contains(string.toCharArray()[i])){
					return false;
				}
				set.add(string.toCharArray()[i]);
			}
			
			return true;
		}
	    public int lengthOfLongestSubstring(String s) {
	        int length = s.length();
			int ans = 0;
			for (int i = 0; i < length; i++){
				for (int j = i + 1; j <= length; j++){
					if (allUnique(s, i, j)){
						ans = Math.max(ans, j - i);
					}
				}
			}
			return ans;
	    }
	}
---

看了别人通过的代码才知道我的暴力破除的确不够优雅啊 ，奉上正确的代码，很简单就不注释了

	public int lengthOfLongestSubstring(String s) {
	    if (s.length()==0) return 0;
	    HashMap<Character, Integer> map = new HashMap<Character, Integer>();
	    int max=0;
	    for (int i=0, j=0; i<s.length(); ++i){
	        if (map.containsKey(s.charAt(i))){
	            j = Math.max(j,map.get(s.charAt(i))+1);
	        }
	        map.put(s.charAt(i),i);
	        max = Math.max(max,i-j+1);
	    }
	    return max;
	}

