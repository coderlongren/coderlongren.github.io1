---
title: convert tree to greater
date: 2018-01-05 22:33:11
tags: Java 
---

![](/images/tree1.png)
<!-- more -->
正文：

# Convert BST to Greater Tree
`题目`
Given a Binary Search Tree (BST), convert it to a Greater Tree such that every key of the original BST is changed to the original key plus sum of all keys greater than the original key in BST.  
**Example** :
```
Input: The root of a Binary Search Tree like this:
              5
            /   \
           2     13

Output: The root of a Greater Tree like this:
             18
            /   \
          20     13
```
*解析* 这道题利用二叉树的后序遍历是最巧妙地，第一次commit, 传值没有通过，最后利用传递A对象的Val，AC
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
   
    public TreeNode convertBST(TreeNode root) {
        A a = new A();
		a.val = Integer.MIN_VALUE;
        search(root,a);
        return root;
    }
  public int search(TreeNode root,A a){
		if (root == null){
			return 0;
		}
		search(root.right, a);
		if (a.val != Integer.MIN_VALUE){
			root.val = root.val + a.val;
		}
		a.val = root.val;
		search(root.left, a);
		return a.val;
	}
	class A{
		int val;
	}
}
```



