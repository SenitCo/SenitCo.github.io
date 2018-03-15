---
title: 二叉树问题集锦（二）
date: 2018-02-27
categories: Data Structure
tags: [code, binary tree, dfs, bfs]
---

数据结构中二叉树问题的总结归纳。
<!--more-->

二叉树结点定义：
```cpp
struct TreeNode 
{
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};
```

### Minimum Depth of Binary Tree
题目描述：Given a binary tree, find its minimum depth. The minimum depth is the number of nodes along the shortest path from the root node down to the nearest leaf node.[LeetCode](https://leetcode.com/problems/minimum-depth-of-binary-tree/description/)
```cpp

//中序遍历
class Solution {
public:
    int minDepth(TreeNode* root) {
        int depth = 1, minDepth = INT_MAX;
        if(!root)   return 0;
        traverse(root, depth, minDepth);
        return minDepth;
    }
private:
    void traverse(TreeNode* node, int depth, int& minDepth)
    {
        if(node == NULL)
            return;
        if(!node->left && !node->right)
        {
            if(depth < minDepth)
                minDepth = depth;
            return;
        }   
        traverse(node->left, depth + 1, minDepth);
        traverse(node->right, depth + 1, minDepth);  
    }
};

//递归遍历
class Solution {
public:
    int minDepth(TreeNode* root) {
        if(!root)
            return 0;
        int l = minDepth(root->left);
        int r = minDepth(root->right);
        if(l == 0 || r == 0)
            return 1 + l + r;
        return 1 + min(l, r);    
    }
};

/**
广度优先遍历（层序遍历），找到的第一个叶节点的深度即是最浅，不需要遍历所有结点
*/
class Solution {
public:
    int minDepth(TreeNode* root) {
        if(!root) return 0;
        queue<TreeNode*> qu;
        TreeNode *last, *now;
        int level, size;
        last = now = root;
        level = 1;
        qu.push(root);
        while(qu.size())
        {
            now = qu.front();
            qu.pop();
            size = qu.size();
            if(now->left)   qu.push(now->left);
            if(now->right)  qu.push(now->right);
            if(qu.size() == size)   break;
            if(last == now)
            {
                level++;
                if(qu.size())
                    last = qu.back();
            }
        }
        return level;      
    }
};
```

### Maximum Depth of Binary Tree
题目描述：Given a binary tree, find its maximum depth. The maximum depth is the number of nodes along the longest path from the root node down to the farthest leaf node.[LeetCode](https://leetcode.com/problems/maximum-depth-of-binary-tree/description/)
```cpp
/**
For example: Given binary tree [3,9,20,null,null,15,7],
    3
   / \
  9  20
    /  \
   15   7
return its depth = 3.
*/

//队列实现层序遍历（广度优先遍历）
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(root == NULL)    return 0;
        queue<TreeNode*> toVisit;
        toVisit.push(root);
        int depth = 0;
        while(!toVisit.empty())
        {
            int size = toVisit.size();
            for(int i = 0; i < size; i++)
            {
                TreeNode* curr = toVisit.front();
                toVisit.pop();
                if(curr->left)  toVisit.push(curr->left);
                if(curr->right) toVisit.push(curr->right);
            }
            depth++;
        }
        return depth;
    }
};

//中序遍历
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(!root)   return 0;
        int depth = 1, maxDepth = 0;
        traverse(root, depth, maxDepth);
        return maxDepth;
    }
private:
    void traverse(TreeNode* node, int depth, int& maxDepth)
    {
        if(node == NULL)
            return;
        if(!node->left && !node->right)
        {
            if(depth > maxDepth)
                maxDepth = depth;
            return;
        }   
        traverse(node->left, depth + 1, maxDepth);
        traverse(node->right, depth + 1, maxDepth);  
    }
};

//递归遍历（深度优先遍历）
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(root == NULL)  return 0;
        return 1 + max(maxDepth(root->left), maxDepth(root->right));
    }
};
```

### Balanced Binary Tree
题目描述：Given a binary tree, determine if it is height-balanced. For this problem, a height-balanced binary tree is defined as: a binary tree in which the depth of the two subtrees of every node never differ by more than 1.[LeetCode](https://leetcode.com/problems/balanced-binary-tree/description/)
```cpp
/**
Example 1: Given the following tree [3,9,20,null,null,15,7]:
    3
   / \
  9  20
    /  \
   15   7
Return true.

Example 2: Given the following tree [1,2,2,3,3,null,null,4,4]:
       1
      / \
     2   2
    / \
   3   3
  / \
 4   4
Return false.
*/

/**
自顶向下：判断每个结点左右子树的深度，如果深度差大于1，则返回false，否则继续考察其子结点是否为平衡树
*/
class Solution {
public:
    bool isBalanced(TreeNode* root) {
        if(root == NULL)    return true;
        int left = depth(root->left);
        int right = depth(root->right);
        if(abs(left - right) > 1)   return false;
        return isBalanced(root->left) && isBalanced(root->right);
    }
private:
    int depth(TreeNode* node)
    {
        if(node == NULL)    return 0;
        return max(depth(node->left), depth(node->right)) + 1;
    }
};

/**
自底向上，后序遍历（左-右-中），从叶子结点开始计算每个结点的高度，并判断左右子树的高度差是否大于1，
若大于1则递归返回1，否则进一步判断其他结点是否为平衡树。（相比于自顶向下，该方法更高效）
*/
class Solution {
public:
    bool isBalanced(TreeNode* root) {
        if(root == NULL)    return true;
        return dfsHeight(root) != -1;
    }
private:
    int dfsHeight(TreeNode* node)
    {
        if(node == NULL)    return 0;
        int left = dfsHeight(node->left);
        if(left == -1)  return -1;
        int right = dfsHeight(node->right);
        if(right == -1) return -1;
        if(abs(left - right) > 1)   return -1;
        return max(left, right) + 1;
    }
};
```