---
title: 二叉搜索树
date: 2018-03-18
categories: Data Structure
tags: [code, binary tree, binary search tree, dp, recursive]
---

数据结构中二叉搜索树问题总结归纳。
<!--more-->

### Unique Binary Search Trees
[Description](https://leetcode.com/problems/unique-binary-search-trees/description/): Given n, how many structurally unique BST's (binary search trees) that store values 1...n?
```cpp
/*
For example, Given n = 3, there are a total of 5 unique BST's.
   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
*/
/**DP方法
See https://leetcode.com/problems/unique-binary-search-trees/discuss/31666
令G(n)表示长度为n的序列的二叉搜索数的数量，F(i, n)表示在长度为 n 的序列中，以 i 为根结点的
二叉搜索数的数量，则有 G(n) = F(1, n) + F(2, n) + ... + F(n, n)，且 G(0) = G(1) = 1
对于F(i, n)，有 F(i, n) = G(i - 1) * G(n - i)
*/
int numTrees(int n) 
{
    vector<int> dp(n + 1, 0);
    dp[0] = dp[1] = 1;
    for(int i = 2; i < n + 1; i++)
    {
        for(int j = 1; j <= i; j++)
        {
            dp[i] += dp[j - 1] * dp[i - j];
        }
    }
    return dp[n];
}
```

### Unique Binary Search Trees II
[Description](https://leetcode.com/problems/unique-binary-search-trees-ii/discuss/): Given an integer n, generate all structurally unique BST's (binary search trees) that store values 1...n.
```cpp
/*
For example, Given n = 3, your program should return all 5 unique BST's shown below.

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
*/
//Definition for a binary tree node.
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

/**
递归解法
*/
class Solution {
public:
    vector<TreeNode*> generateTrees(int n) {
        if(n == 0)  return vector<TreeNode*>();
        return generateBST(1, n);
    }
private:
    vector<TreeNode*> generateBST(int start, int end)
    {
        vector<TreeNode*> result;
        if(start > end)
            result.push_back(NULL);
        vector<TreeNode*> left, right;
        for(int i = start; i <= end; i++)
        {
            left = generateBST(start, i - 1);
            right = generateBST(i + 1, end);
            for(auto lnode : left)
            {
                for(auto rnode: right)
                {
                    TreeNode* root = new TreeNode(i);
                    root->left = lnode;
                    root->right = rnode;
                    result.push_back(root);
                }
            }
        }
        return result;
    }
};

```