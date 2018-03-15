---
title: 二叉树与深度优先遍历（二）
date: 2018-02-24
categories: Data Structure
tags: [code, dfs, binary tree]
---
数据结构与算法中利用深度优先遍历(DFS)借助二叉树问题。
<!--more-->

### Convert Sorted Array to Binary Search Tree
题目描述：Given an array where elements are sorted in ascending order, convert it to a height balanced BST. For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1.[LeetCode](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/description/)
```cpp
/**
Example: Given the sorted array: [-10,-3,0,5,9],
One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:
      0
     / \
   -3   9
   /   /
 -10  5
*/

//递归二分，将中点值置于合适的树结点中
class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        TreeNode* root = NULL;
        partition(root, nums, 0, nums.size() - 1);
        return root;
    }
private:
    void partition(TreeNode*& node, vector<int>& nums, int left, int right)
    {
        if(left > right)    return;
        int mid = (left + right) / 2;
        node = new TreeNode(nums[mid]);
        partition(node->left, nums, left, mid - 1);
        partition(node->right, nums, mid + 1, right);
    }
};
```

### Convert Sorted List to Binary Search Tree
题目描述：Given a singly linked list where elements are sorted in ascending order, convert it to a height balanced BST. For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1.[LeetCode](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/description/)
```cpp
/**
Example: Given the sorted linked list: [-10,-3,0,5,9],
One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:
      0
     / \
   -3   9
   /   /
 -10  5
*/

/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */

/**
同样采用递归二分的思想，由于链表不能随机存取，因此只能通过遍历的方式定位到区间中点，此处采用快慢指针
*/
class Solution {
public:
    TreeNode* sortedListToBST(ListNode* head) {
        TreeNode* root = NULL;
        partition(root, head, NULL);
        return root;
    }
private:
    void partition(TreeNode*& tree, ListNode* start, ListNode* end)
    {
        ListNode *slow = start, *fast = start;
        if(start == end)
            return;   
        while(fast != end && fast->next != end)
        {
            slow = slow->next;
            fast = fast->next->next;
        }
        tree = new TreeNode(slow->val);
        partition(tree->left, start, slow);
        partition(tree->right, slow->next, end);
    }
};

//采用累计步长定位到区间中点
class Solution {
public:
    TreeNode* sortedListToBST(ListNode* head) {
        TreeNode* root = NULL;
        int size = 0;
        for(ListNode* node = head; node != NULL; node = node->next) size++;
        partition(root, head, size);
        return root;
    }
private:
    void partition(TreeNode*& tree, ListNode* start, int size)
    {
        if(size < 1)
            return;   
        ListNode *mid = start;
        int half = size >> 1;
        for(int i = 0; i < half; i++)
            mid = mid->next;
        tree = new TreeNode(mid->val);
        partition(tree->left, start, half);
        partition(tree->right, mid->next, size - half - 1); //此处size-half不能直接替换为size/2
    }
};
```