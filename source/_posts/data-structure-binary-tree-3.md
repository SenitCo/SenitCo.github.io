---
title: 二叉树问题集锦（三）
date: 2018-03-10
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

### Validate Binary Search Tree
Given a binary tree, determine if it is a valid binary search tree (BST).
Assume a BST is defined as follows:
The left subtree of a node contains only nodes with keys less than the node's key.
The right subtree of a node contains only nodes with keys greater than the node's key.
Both the left and right subtrees must also be binary search trees.[LeetCode](https://leetcode.com/problems/validate-binary-search-tree/description/)
```cpp
/**
Example 1:
    2
   / \
  1   3
Binary tree [2,1,3], return true.
Example 2:
    1
   / \
  2   3
Binary tree [1,2,3], return false.
*/

/**
迭代法中序遍历，对于合法二叉搜索树，中序遍历的结果为递增关系
*/
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        if(root == NULL)    return true;
        TreeNode* prev = NULL;
        stack<TreeNode*> toVisit;
        while(root != NULL || !toVisit.empty())
        {
            while(root != NULL)
            {
                toVisit.push(root);
                root = root->left;
            }
            root = toVisit.top();
            toVisit.pop();
            if(prev != NULL && root->val <= prev->val)
                return false;
            prev = root;
            root = root->right;
        }
        return true;
    }
};

/**
递归法中序遍历
*/
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        TreeNode* prev = NULL;
        return isValid(root, prev);
    }
private:
    bool isValid(TreeNode* node, TreeNode* &prev)   //注意此处传的为指针的引用
    {
        if(node == NULL)    return true;
        if(!isValid(node->left, prev))   return false;
        if(prev != NULL && prev->val >= node->val)
            return false;
        prev = node;
        return isValid(node->right, prev);
    }
};
```

### Recover Binary Search Tree
Two elements of a binary search tree (BST) are swapped by mistake. Recover the tree without changing its structure.
Note: A solution using O(n) space is pretty straight forward. Could you devise a constant space solution?[LeetCode](https://leetcode.com/problems/recover-binary-search-tree/description/)
```cpp
/**
递归法中序遍历，对于异常点，存在前一个元素大于后一个元素，但两个异常点略有差异（第一个异常点，
其值大于后一个元素；第二个异常点，其值小于前一个元素）
*/
class Solution {
public:
    void recoverTree(TreeNode* root) {
        inorderTraverse(root);
        int temp = first->val;
        first->val = second->val;
        second->val = temp;
    }
private:
    TreeNode *first = NULL, *second = NULL; //如果不定义为成员变量，而是作为形参，则应该传指针变量的引用
    TreeNode *prev = new TreeNode(INT_MIN);
    void inorderTraverse(TreeNode* root)
    {
        if(!root)   return;
        inorderTraverse(root->left);
        if(first == NULL && prev->val >= root->val)
            first = prev;
        if(first != NULL && prev->val >= root->val)
            second = root;
        prev = root;
        inorderTraverse(root->right);
    }
};

/**
迭代法中序遍历
*/
class Solution {
public:
    void recoverTree(TreeNode* root) {
        TreeNode *first = NULL, *second = NULL;
        TreeNode *prev = new TreeNode(INT_MIN);
        stack<TreeNode*> toVisit;
        while(root != NULL || !toVisit.empty())
        {
            while(root)
            {
                toVisit.push(root);
                root = root->left;
            }
            root = toVisit.top();
            toVisit.pop();
            if(first == NULL && prev->val >= root->val)
                first = prev;
            if(first != NULL && prev->val >= root->val)
                second = root;
            prev = root;
            root = root->right;
        }
        
        int temp = first->val;
        first->val = second->val;
        second->val = temp;
    }
};
```

### Same Tree
Given two binary trees, write a function to check if they are the same or not. Two binary trees are considered the same if they are structurally identical and the nodes have the same value.
```cpp
/**
Example 1:
Input:     1         1
          / \       / \
         2   3     2   3
        [1,2,3],   [1,2,3]
Output: true

Example 2:
Input:     1         1
          /           \
         2             2
        [1,2],     [1,null,2]
Output: false

Example 3:
Input:     1         1
          / \       / \
         2   1     1   2
        [1,2,1],   [1,1,2]
Output: false
*/

/**
递归法中序遍历，对于这种判断真假的递归，与一般的递归遍历相比有些特殊，因为在遍历过程中，
如果不符合条件，则终止递归直接返回false，而最终的结果若要返回true，则需完整遍历一次，且
遍历过程中所有判断条件都要为真
*/
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        return traverse(p, q);
    }
private:
    bool traverse(TreeNode* p, TreeNode* q)
    {
        if(p == NULL && q == NULL)  return true;
        else if((p && !q) || (!p && q)) return false;
        if(!traverse(p->left, q->left)) return false;
        if (p->val != q->val)
            return false;
        return traverse(p->right, q->right);
    }
};

/**
递归法先序遍历
*/
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        if(p == NULL && q == NULL) return true;
        if(p == NULL || q == NULL) return false;
        if(p->val == q->val)
            return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
        return false;
    }
};

```

### Symmetric Tree
Given a binary tree, check whether it is a mirror of itself (ie, symmetric around its center).[LeetCode](https://leetcode.com/problems/symmetric-tree/description/)
```cpp
/**
For example, this binary tree [1,2,2,3,4,4,3] is symmetric:
    1
   / \
  2   2
 / \ / \
3  4 4  3
But the following [1,2,2,null,3,null,3] is not:
    1
   / \
  2   2
   \   \
   3    3
*/

/**
借助队列实现层序遍历（广度优先遍历），使用一个layer数组存储每一层的数据，对于左子结点或右子结点为空的情况，往数组中对应位置存0
（其实略有不妥，应该存一个不会出现的数据）。访问完上一层的结点后，下一层的结点全部进入队列，同时下一层的数据（包括部分空结点）也
全部存入数组，这时数组应该是前后对称的。至于要处理子结点为空的情况，是为了避免将上述第二个二叉树那样的结构认为是对称的。
*/
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if(root == NULL)    return true;
        queue<TreeNode*> toVisit;
        toVisit.push(root);
        while(!toVisit.empty())
        {
            vector<int> layer;
            int size = toVisit.size();
            for(int i = 0; i < size; i++)
            {
                TreeNode* curr = toVisit.front();
                toVisit.pop();
                if(curr->left)  
                {
                    toVisit.push(curr->left);
                    layer.push_back(curr->left->val);
                }
                else    
                    layer.push_back(0);
                if(curr->right)
                {
                    toVisit.push(curr->right);
                    layer.push_back(curr->right->val);
                }
                else
                    layer.push_back(0);
            }
            for(int i = 0, j = layer.size() - 1; i < j; i++, j--)
                if(layer[i] != layer[j])
                    return false;
        }
        return true;
    }
};

//递归解法
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if(root == NULL)    return true;
        return recursive(root->left, root->right);
    }
private:
    bool recursive(TreeNode* left, TreeNode* right)
    {
        if(left == NULL || right == NULL)
            return left == right;
        if(left->val != right->val)
            return false;
        return recursive(left->left, right->right) && recursive(left->right, right->left);
    }
};

/**
迭代解法，借助两个队列实现（栈也可以）
*/
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if(root == NULL)    return true;
        queue<TreeNode*> q1, q2;
        q1.push(root->left);
        q2.push(root->right);
        while(!q1.empty() && !q2.empty())
        {
            TreeNode* left = q1.front();
            TreeNode* right = q2.front();
            q1.pop();
            q2.pop();
            if(left == NULL && right == NULL)
                continue;
            if(left == NULL || right == NULL)
                return false;
            if(left->val != right->val)
                return false;
            q1.push(left->left);    //入队列的顺序为左右
            q1.push(left->right);
            q2.push(right->right);  //入队列的顺序为右左
            q2.push(right->left);
        }
        return true;
    }
};
```
