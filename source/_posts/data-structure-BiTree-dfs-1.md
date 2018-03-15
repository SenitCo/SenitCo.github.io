---
title: 二叉树与深度优先遍历（一）
date: 2018-02-23
categories: Data Structure
tags: [code, dfs, binary tree]
---
数据结构与算法中利用深度优先遍历(DFS)借助二叉树问题。
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

### Sum Root to Leaf Numbers
题目描述：Given a binary tree containing digits from 0-9 only, each root-to-leaf path could represent a number. An example is the root-to-leaf path 1->2->3 which represents the number 123. Find the total sum of all root-to-leaf numbers.[LeetCode](https://leetcode.com/problems/sum-root-to-leaf-numbers/description/)
```cpp
For example,
    1
   / \
  2   3
The root-to-leaf path 1->2 represents the number 12.
The root-to-leaf path 1->3 represents the number 13.
Return the sum = 12 + 13 = 25.

//法一：栈实现
int sumNumbers(TreeNode* root) 
{
    if(!root)   return 0;
    int sum = 0;
    stack<TreeNode*> toVisit;
    toVisit.push(root);
    while(!toVisit.empty())
    {
        TreeNode* cur = toVisit.top();
        toVisit.pop();
        if(cur->right)
        {
            cur->right->val = cur->val * 10 + cur->right->val;
            toVisit.push(cur->right);
        }
        if(cur->left)
        {
            cur->left->val = cur->val * 10 + cur->left->val;
            toVisit.push(cur->left);
        }
        if(!cur->left && !cur->right)
        {
            sum += cur->val;
        }
    }
    return sum;
}

//法二：递归实现
class Solution {
public:
    int sumNumbers(TreeNode* root) {
        return sum(root, 0);
    }
private:
    int sum(TreeNode* node, int s)
    {
        if(!node)   return 0;
        if(!node->left && !node->right)
            return s * 10 + node->val;
        return sum(node->left, s * 10 + node->val) + sum(node->right, s * 10 + node->val);
    }
};

```

### Path Sum
题目描述：Given a binary tree and a sum, determine if the tree has a root-to-leaf path such that adding up all the values along the path equals the given sum.[LeetCode](https://leetcode.com/problems/path-sum/description/)
```cpp
For example: Given the below binary tree and sum = 22,
              5
             / \
            4   8
           /   / \
          11  13  4
         /  \      \
        7    2      1
return true, as there exist a root-to-leaf path 5->4->11->2 which sum is 22.

//深度优先遍历，迭代（借助栈）实现
bool hasPathSum(TreeNode* root, int sum) 
{
    if(!root)   return false;
    stack<TreeNode*> toVisit;
    toVisit.push(root);
    int pathSum = 0;
    while(!toVisit.empty())
    {
        TreeNode* cur = toVisit.top();
        toVisit.pop();
        if(cur->right)
        {
            cur->right->val += cur->val;
            toVisit.push(cur->right);
        }
        if(cur->left)
        {
            cur->left->val += cur->val;
            toVisit.push(cur->left);
        }
        if(!cur->left && !cur->right)
        {
            if(cur->val == sum)
                return true;
        }
    }
    return false;
}

//深度优先遍历，递归实现
bool hasPathSum(TreeNode* root, int sum) 
{
    if(!root)   return false;
    if(!root->left && !root->right && root->val == sum) return true;
    return hasPathSum(root->left, sum - root->val) || hasPathSum(root->right, sum - root->val);
}
```

### Path Sum II
题目描述：Given a binary tree and a sum, find all root-to-leaf paths where each path's sum equals the given sum.[LeetCode](https://leetcode.com/problems/path-sum-ii/description/)
```cpp
For example: Given the below binary tree and sum = 22,
              5
             / \
            4   8
           /   / \
          11  13  4
         /  \    / \
        7    2  5   1
return
[
   [5,4,11,2],
   [5,8,4,5]
]

//深度优先遍历 + 回溯法
class Solution {
public:
    vector<vector<int>> pathSum(TreeNode* root, int sum) {
        vector<vector<int>> result;
        if(!root)   return result;
        vector<int> path;
        traverse(root, sum, result, path);
        return result;
    }
private:
    void traverse(TreeNode* node, int sum, vector<vector<int>>& result, vector<int>& path)
    {
        if(node == NULL)    return;
        path.push_back(node->val);
        if(!node->left && !node->right && sum == node->val)
            result.push_back(path);
        traverse(node->left, sum - node->val, result, path);
        traverse(node->right, sum - node->val, result, path);
        path.pop_back();
    }

    //与上面等价
    void traverse(TreeNode* node, int sum, vector<vector<int>>& result, vector<int>& path)
    {
        if(node == NULL)    return;
        path.push_back(node->val);
        if(!node->left && !node->right && sum == node->val)
        {
            result.push_back(path);
            path.pop_back();
            return;
        }
        traverse(node->left, sum - node->val, result, path);
        traverse(node->right, sum - node->val, result, path);
        path.pop_back();
    }
};
```

### Flatten Binary Tree to Linked List
题目描述：Given a binary tree, flatten it to a linked list in-place.[LeetCode](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/description/)
```cpp
/**
Given a binary tree, flatten it to a linked list in-place.
For example, Given

         1
        / \
       2   5
      / \   \
     3   4   6
The flattened tree should look like:
   1
    \
     2
      \
       3
        \
         4
          \
           5
            \
             6
*/

/**
迭代法：对任意一个结点curr，其左子树（如果存在）按先序遍历的最后一个结点的右子结点为curr的右子结点，
curr的右子结点为curr的左子结点
*/
void flatten(TreeNode* root) 
{
    TreeNode* curr = root;
    while(curr)
    {
        if(curr->left)
        {
            TreeNode* prev = curr->left;
            while(prev->right)
                prev = prev->right; //遍历到左子树的最后一个结点，用于连接curr的右子树
            prev->right = curr->right;
            curr->right = curr->left;   //用当前结点的左子树替换右子树
            curr->left = NULL;
        }
        curr = curr->right; //往右移动结点
    }
}

//递归法（先序遍历的逆转版本：右-左-中）
class Solution {
public:
    void flatten(TreeNode* root) {
        TreeNode* prev = NULL;
        reversePreorder(root, prev);
    }
private:
    //prev指针必须得传引用，保持更新，或者定义成一个成员变量
    void reversePreorder(TreeNode* node, TreeNode*& prev)   
    {
        if(node == NULL)    return;
        reversePreorder(node->right, prev);
        reversePreorder(node->left, prev);
        node->right = prev;
        node->left = NULL;
        prev = node;
    }
};
```

### Binary Tree Maximum Path Sum
题目描述：Given a binary tree, find the maximum path sum. For this problem, a path is defined as any sequence of nodes from some starting node to any node in the tree along the parent-child connections. The path must contain at least one node and does not need to go through the root.[LeetCode](https://leetcode.com/problems/binary-tree-maximum-path-sum/description/)
分析：要求二叉树中任意两个结点之间最大路径和，而不是从根结点出发到某一子结点的最大路径和。可采用深度优先遍历求解，只不过递归函数的返回值不是要求的最大和（最大和通过一个全局变量或者引用参数来同步更新），而是结点自身为根结点时到其子结点的最大路径，该值用于提供给其父结点计算最长路径（当其父节点为根结点时，下面的子结点只能是单向的）。简单来说，一个结点的最大路径和是其左子树路径和 + 右子树路径和 + 当前节点值，而返回值则是当前结点值加上左子树路径和与右子树路径和的较大值。考虑负数的影响，加入与0的比较。
```cpp
int maxPathSum(TreeNode* root) 
{
    int maxVal = INT_MIN;
    maxPathDown(maxVal, root);
    return maxVal;
}
int maxPathDown(int& maxVal, TreeNode* node)
{
    if(!node)   return 0;
    int left = max(0, maxPathDown(maxVal, node->left));     //与0比较考虑了负数的影响
    int right = max(0, maxPathDown(maxVal, node->right));
    maxVal = max(maxVal, left + right + node->val);
    return max(left, right) + node->val;
}
```