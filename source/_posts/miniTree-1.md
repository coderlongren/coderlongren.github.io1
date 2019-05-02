---
title: miniTree
date: 2018-01-05 22:32:55
tags: java 
---


![](/images/suanfa1.jpg)
<!-- more -->
正文：

# 求一棵树的最小深度 
我在这里给出了三种语言的解法。
`Java version`
```
public int minDepth(TreeNode root) {
    if (root == null) return 0;
    int L = minDepth(root.left), R = minDepth(root.right);
    return 1 + (Math.min(L, R) > 0 ? Math.min(L, R) : Math.max(L, R));
}

public int minDepth(TreeNode root) {
    if (root == null) return 0;
    int L = minDepth(root.left), R = minDepth(root.right), m = Math.min(L, R);
    return 1 + (m > 0 ? m : Math.max(L, R));
}

public int minDepth(TreeNode root) {
    if (root == null) return 0;
    int L = minDepth(root.left), R = minDepth(root.right);
    return L<R && L>0 || R<1 ? 1+L : 1+R;
}
```
`Python version`
```
def minDepth(self, root):
    if not root: return 0
    d = map(self.minDepth, (root.left, root.right))
    return 1 + (min(d) or max(d))

def minDepth(self, root):
    if not root: return 0
    d, D = sorted(map(self.minDepth, (root.left, root.right)))
    return 1 + (d or D)
```
`c++ version`
```
int minDepth(TreeNode* root) {
    if (!root) return 0;
    int L = minDepth(root->left), R = minDepth(root->right);
    return 1 + (min(L, R) ? min(L, R) : max(L, R));
}

int minDepth(TreeNode* root) {
    if (!root) return 0;
    int L = minDepth(root->left), R = minDepth(root->right);
    return 1 + (L && R ? min(L, R) : max(L, R));
}

int minDepth(TreeNode* root) {
    if (!root) return 0;
    int L = minDepth(root->left), R = minDepth(root->right);
    return 1 + (!L-!R ? max(L, R) : min(L, R));
}

int minDepth(TreeNode* root) {
    if (!root) return 0;
    int L = minDepth(root->left), R = minDepth(root->right);
    return L<R && L || !R ? 1+L : 1+R;
}
```
