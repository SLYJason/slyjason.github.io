---
layout: post
title: "Lowest Common Ancestor of Binary Tree"
subtitle: "LCA"
date: 2021-02-01
author: "Luyi"
header-img: "img/in-post/2021-02-01-lca-of-binary-tree/post-bg.jpg"
tags:
    - Algorithms
---

### What is the Lowest Ancestor (LCA) of Binary Tree?
The **lowest common ancestor** is the lowest node in the tree that has both `p` and `q` as descendants. Hence, the LCA of a binary tree with nodes `p` and `q` is the shared ancestor of `p` and `q` that is located **farthest** from the root. Looking the following example:
![Figure_1](/img/in-post/2021-02-01-lca-of-binary-tree/Figure_1.svg)
From the above figure, given node `6` and `4` as colored by `yellow`, the lowest common ancestor should be `5` as colored by `green`. Note node `3` which is the root node is not LCA, since LCA means the common ancestor is located **farthest** from the root.

### Case 1: LCA of Binary Search Tree
Let's first look at an easy case, what is the LCA of a binary search tree (BST). From [wikipedia](https://en.wikipedia.org/wiki/Binary_search_tree): BST is a rooted binary tree data structure with the key of each internal node being greater than all the keys in the respective node's left subtree and less than the ones in its right subtree.
An intuitive approach would be using recursive to compare the current node with `p` and `q`:
1. If the current node is a separation node that node `p` and `q` lives in different left subtree or right subtree, then the current node is LCA.
2. If both `p` and `q`  smaller than the current node, then LCA must reside in left subtree, thus continuously to search LCA in the left subtree.
3. If both `p` and `q`  bigger than the current node, then LCA must reside in right subtree, thus continuously to search LCA in the right subtree.
```java
public TreeNode LCA(TreeNode root, TreeNode p, TreeNode q) {
        if (p.val <= root.val && q.val >= root.val || q.val <= root.val && p.val >= root.val) return root; // current node is LCA.
        if (p.val < root.val && q.val < root.val) {
            return lowestCommonAncestor(root.left, p, q); // search LCA in left subtree.
        } else {
            return lowestCommonAncestor(root.right, p, q); // search LCA in right subtree.
        }
}
```
Similarly, using an iterative approach:
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        TreeNode parent = root;
        while (parent != null) {
            if (p.val <= parent.val && q.val <= parent.val) {
                parent = parent.left; // search LCA in left subtree.
            } else if (p.val > parent.val && q.val > parent.val) {
                parent = parent.right; // search LCA in right subtree.
            } else {
                return parent; // found LCA.
            }
        }
        return null;
}
```

### Case 2: LCA of General Binary Tree
Now let's look at a more general case, what is the LCA of general binary tree. The idea is still same if using recursive approach, keep searching node `p` and `q` in the current node left subtree and right subtree:
1. If current node is equal `p` or `q` then return the current node.
2. If the returned LCA from both left subtree and right subtree not null, then the current node is LCA.
3. If the returned LCA from both left subtree and right subtree are null, then the current node is not LCA.
4. Then must either LCA returned from left subtree or LCA returned from right subtree is not null, so return the not null one.
```java
public TreeNode LCA(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null) return null;
        if (root == p || root == q) return root;
        
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        
        if (left != null && right != null) return root; // LCA found which is root.
        if (left == null && right == null) return null; // Both p and q not found.
        return left != null ? left : right; // either p or q found.
}
```
Similarly, we can also use an iterative approach:
```java
public TreeNode LCA(TreeNode root, TreeNode p, TreeNode q) {
        Map<TreeNode, TreeNode> parent = new HashMap<>(); // key: left or right child node, value: root node.
        parent.put(root, null);
        Deque<TreeNode> stack = new ArrayDeque<>();
        stack.push(root);

        // step 1: build parent, find p and q.
        while (!parent.containsKey(p) || !parent.containsKey(q)) {
            TreeNode node = stack.pop();
            if (node.left != null) {
                parent.put(node.left, node);
                stack.push(node.left);
            }
            if (node.right != null) {
                parent.put(node.right, node);
                stack.push(node.right);
            }
        }
        
        // step 2: find all ancestors of node p, the LCA must exist in one of these ancestors.
        Set<TreeNode> ancestors = new HashSet<>();
        while (p != null) {
            ancestors.add(p);
            p = parent.get(p);
        }
        // step 3: go through all the ancestors of node q, if the ancestors of node p contains any first ancestor of q, then the LCA found.
        while (!ancestors.contains(q)) {
            q = parent.get(q);
        }
        return q;
    }
```
### Conclusion
Finding LCA in binary tree and graph is a very common problem, sometimes if we think binary tree is a special type of graph more like a directed acyclic graph (DAG) in this case, the problem will be much easier. We can either apply recursive or iterative approach to solve the problem.
