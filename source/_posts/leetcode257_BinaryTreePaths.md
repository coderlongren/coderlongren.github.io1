---
title: leetcode257_BinaryTreePaths
date: 2018-3-11 14:18:40
categories:
	- leetcode
tags:
	- leetcode
---
 Given a binary tree, return all root-to-leaf paths.

For example, given the following binary tree: 
	  1
	 /   \
	2     3
	 \
	  5

 All root-to-leaf paths are:

	`["1->2->5", "1->3"]`

```java
	 public List<String> binaryTreePaths(TreeNode root) {
		 List<String> paths = new LinkedList<String>();
		 if (root == null){
			 return paths;
		 }
		 searchBT(root, "", paths);
		return paths;
	  }
	  // 简单的递归实现
	 public static void searchBT(TreeNode root,String path,List<String> answer){
		 
		 if (root.left == null && root.right == null){
			 answer.add(path + root.val);
		 }
		 if (root.left != null){
			 searchBT(root.left, path + root.val + "->", answer);
		 }
		 if (root.right != null){
			 searchBT(root.right, path + root.val + "->", answer);

		 }
		 
		 
	 }
```



