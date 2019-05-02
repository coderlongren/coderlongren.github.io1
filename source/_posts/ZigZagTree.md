---
title: ZigZagTree
date: 2017-11-13 19:33:49
categories:
	- Algorithm
tags:
	- Java
	- Algorithm
---
摘要: Binary Tree Level Order Traversal
<!-- more -->
Given a binary tree, return the zigzag level order traversal of its nodes' values. (ie, from left to right, then right to left for the next level and alternate between).

For example:
Given binary tree {3,9,20,#,#,15,7},

    3
   / \
  9  20
    /  \
   15   7

 

return its zigzag level order traversal as:

[
  [3],
  [20,9],
  [15,7]
]
意思大概就是说，求一颗二叉树的ZigZag样式存储到list中，和之前的那一道深度遍历的题目是差不多的
我们可以维护两个栈，一个stack 是从左往右进栈,另一个是右往左进栈这样出栈的顺序就是我们想要的之
字形了。代码如下

	public static List<List<Integer>> zigzagLevelOrder(TreeNode root) {
	    List<List<Integer>> res = new ArrayList<List<Integer>>();
	    if (root == null){
	    	return res;
	    }
	    // 我们可以维护两个stack 一个stack 是从左往右进栈 另一个是 从右往左进栈
	    Stack<TreeNode> stack1 = new Stack<TreeNode>();
	    Stack<TreeNode> stack2 = new Stack<TreeNode>();
	    stack1.push(root);
	    
	    while(!stack1.isEmpty() || !stack2.isEmpty()){
	    	List<Integer> temp = new ArrayList<Integer>();
	    	while(!stack1.isEmpty()){
	    		TreeNode node = stack1.pop();
	    		temp.add(node.val);
	    		if (node.left != null){
	    			stack2.add(node.left);
	    		}
	    		if (node.right != null){
	    			stack2.add(node.right);
	    		}
	    	}
	    	if (temp.size() != 0){
	    		res.add(temp);	
	    	}
	    	
	    	//重新给temp赋值
	    	temp = new ArrayList<Integer>();
	    	while(!stack2.isEmpty()){
	    		TreeNode node = stack2.pop();
	    		
	    		temp.add(node.val);
	    		if (node.right != null){
	    			stack1.add(node.right);
	    		}
	    		if (node.left != null){
	    			stack1.add(node.left);
	    		}
	    		
	    	}
	    	if (temp.size() != 0){
	    		res.add(temp);	
	    	}
	    	
	    }
	    return res;
		
	}
---
结果令人失望，只击败了6%，使用DP，降低了时间复杂度
![Accepted](/images/beatme2.png)
---
