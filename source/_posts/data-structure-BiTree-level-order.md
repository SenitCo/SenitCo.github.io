---
title: 二叉树与层序遍历
date: 2018-02-25
categories: Data Structure
tags: [code, binary tree, bfs, level order]
---
数据结构与算法中利用层序遍历(DFS)借助二叉树问题。
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

### Binary Tree Level Order Traversal
题目描述：Given a binary tree, return the level order traversal of its nodes' values. (ie, from left to right, level by level).[LeetCode](https://leetcode.com/problems/binary-tree-level-order-traversal/description/)
```cpp
/**
For example: Given binary tree [3,9,20,null,null,15,7],
    3
   / \
  9  20
    /  \
   15   7
return its level order traversal as:
[
  [3],
  [9,20],
  [15,7]
]
*/
/**
借助队列实现广度优先遍历
*/
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> result;
        if(root == NULL)    return result;
        queue<TreeNode*> toVisit;
        toVisit.push(root);
        while(!toVisit.empty())
        {
            int size = toVisit.size();
            vector<int> layer;
            for(int i = 0; i < size; i++)   //区分每一层的数据
            {
                TreeNode* curr = toVisit.front();
                toVisit.pop();
                if(curr->left)  toVisit.push(curr->left);
                if(curr->right) toVisit.push(curr->right);
                layer.push_back(curr->val);
            }
            result.push_back(layer);
        }
        return result;
    }
};

/**
深度优先遍历（先序遍历），借助一个变量表示当前的层数，如此遍历顺序不再关键
*/
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> result;
        preorderDFS(result, root, 0);
        return result;
    }
private:
    void preorderDFS(vector<vector<int>>& result, TreeNode* node, int depth)
    {
        if(node == NULL)    return;
        if(result.size() == depth)
            result.push_back(vector<int>());
        result[depth].push_back(node->val);
        preorderDFS(result, node->left, depth + 1);
        preorderDFS(result, node->right, depth + 1);
    }
};
```

### Populating Next Right Pointers in Each Node
题目描述：Given a binary tree. Populate each next pointer to point to its next right node. If there is no next right node, the next pointer should be set to NULL. Initially, all next pointers are set to NULL.
Note: You may only use constant extra space. You may assume that it is a perfect binary tree (ie, all leaves are at the same level, and every parent has two children).[LeetCode](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/description/)
```cpp
/**
For example, Given the following perfect binary tree,
         1
       /  \
      2    3
     / \  / \
    4  5  6  7
After calling your function, the tree should look like:
         1 -> NULL
       /  \
      2 -> 3 -> NULL
     / \  / \
    4->5->6->7 -> NULL
*/

//Definition for binary tree with next pointer.
struct TreeLinkNode {
    int val;
    TreeLinkNode *left, *right, *next;
    TreeLinkNode(int x) : val(x), left(NULL), right(NULL), next(NULL) {}
};

/**
借助队列实现广度优先遍历，但空间复杂度不是常量存储
*/
class Solution {
public:
    void connect(TreeLinkNode *root) {
        queue<TreeLinkNode*> toVisit;
        if(!root)   return;
        toVisit.push(root);
        while(!toVisit.empty())
        {
            int size = toVisit.size();
            TreeLinkNode* cur = toVisit.front();
            if(!cur)  return;
            toVisit.pop();
            toVisit.push(cur->left);
            toVisit.push(cur->right);
            TreeLinkNode* next;
            for(int i = 0; i < size - 1; i++)
            {
                
                next = toVisit.front();
                toVisit.pop();
                cur->next = next;
                toVisit.push(next->left);
                toVisit.push(next->right);
                cur = next;
            }
        }
        return;
    }
};

class Solution {
public:
    void connect(TreeLinkNode *root) {
        if(!root)   return;
        TreeLinkNode *pre = root, *cur = NULL;
        while(pre->left)
        {
            cur = pre;
            while(cur)
            {
                cur->left->next = cur->right;
                if(cur->next)
                    cur->right->next = cur->next->left;
                cur = cur->next;
            }
            pre = pre->left;
        }
    }
};
```

### Populating Next Right Pointers in Each Node II
题目描述：Follow up for problem "Populating Next Right Pointers in Each Node". What if the given tree could be any binary tree? Would your previous solution still work? Note: You may only use constant extra space.[LeetCode](https://leetcode.com/problems/populating-next-right-pointers-in-each-node-ii/description/)
```cpp
/**
For example, Given the following binary tree,
         1
       /  \
      2    3
     / \    \
    4   5    7
After calling your function, the tree should look like:
         1 -> NULL
       /  \
      2 -> 3 -> NULL
     / \    \
    4-> 5 -> 7 -> NULL
*/

//Definition for binary tree with next pointer.
struct TreeLinkNode {
    int val;
    TreeLinkNode *left, *right, *next;
    TreeLinkNode(int x) : val(x), left(NULL), right(NULL), next(NULL) {}
};

//层序遍历
class Solution {
public:
    void connect(TreeLinkNode *root) {
        TreeLinkNode *head = root;  //下一层最左边的结点（起始有效结点）
        TreeLinkNode *curr = NULL;  //上一层中的当前结点
        TreeLinkNode *prev = NULL;  //下一层中前一个结点（用于连接当前结点的左子结点或右子结点作为其下一个结点）
        while(head)
        {
            curr = head;
            head = prev = NULL;
            while(curr)
            {
                if(curr->left)      
                {
                    if(prev)
                        prev->next = curr->left;    //给前一个结点的next指针赋值
                    else
                        head = curr->left;  //如果前一个结点不存在，则给起始结点赋值
                    prev = curr->left;      //更新前一个结点
                }
                if(curr->right)
                {
                    if(prev)
                        prev->next = curr->right;
                    else
                        head = curr->right;
                    prev = curr->right;
                }
                curr = curr->next;
            }
        }
    }
};
```