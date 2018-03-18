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

### Construct Binary Tree from Preorder and Inorder Traversal
题目描述：Given preorder and inorder traversal of a tree, construct the binary tree.
Note: You may assume that duplicates do not exist in the tree.[LeetCode](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description/)
```cpp
For example, given preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
Return the following binary tree:
    3
   / \
  9  20
    /  \
   15   7

/**递归法
对于先序中的第一个元素，找到中序中的对应元素的位置（可借助一个hash表直接存取，无需每次遍历），然后以该位置为中点，
将中序序列分成两部分递归进行，同时确定两个子序列的先序起点和终点
*/
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        if(preorder.empty())    return NULL;
        unordered_map<int, int> mp;
        for(int i = 0; i < inorder.size(); i++)
            mp[inorder[i]] = i;
        TreeNode* root = recursive(preorder, inorder, 0, preorder.size() - 1, 0, inorder.size() - 1, mp);
        return root;
    }
private:
    TreeNode* recursive(vector<int>& preorder, vector<int>& inorder, int preStart, int preEnd, int inStart, int inEnd, unordered_map<int, int>& mp)
    {
        if(preStart > preEnd || inStart > inEnd)    return NULL;
        TreeNode* root = new TreeNode(preorder[preStart]);
        int inRoot = mp[root->val];
        int numLeft = inRoot - inStart; //左子序列的长度
        root->left = recursive(preorder, inorder, preStart + 1, preStart + numLeft, inStart, inRoot - 1, mp);
        root->right = recursive(preorder, inorder, preStart + numLeft + 1, preEnd, inRoot + 1, inEnd, mp);
        return root;
    }
};

/**迭代法
使用两个变量i,j分别表示先序和中序的索引，并将先序中的元素依次压入栈中，进栈的同时依次生成每颗子树的左子结点，
直至栈顶元素和中序元素一致，说明该结点为某颗子树的根结点，这时应该为该结点插入右子结点，并将该该结点出栈。
*/
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        if(preorder.empty())    return NULL;
        stack<TreeNode*> st;
        TreeNode* root = new TreeNode(preorder[0]);
        st.push(root);
        int i = 1, j = 0, flag = 0; //使用flag标记是插入左子结点还是右子结点
        TreeNode* t = root;
        while(i < preorder.size())
        {
            if(!st.empty() && st.top()->val == inorder[j])
            {
                t = st.top();
                st.pop();
                flag = 1;   //只有在回溯时（栈实现）遇到到子树的根结点才会右转生成右子结点
                j++;
            }
            else
            {
                if(flag == 0)
                {
                    t->left = new TreeNode(preorder[i]);
                    t = t->left;
                    st.push(t);
                    i++;
                }
                else
                {
                    flag = 0;   //每次插入右子结点重置flag标志
                    t->right = new TreeNode(preorder[i]);
                    t = t->right;
                    st.push(t);
                    i++;
                }
            }
        }
        return root;
    }
};
```

### Construct Binary Tree from Inorder and Postorder Traversal
题目描述：Given inorder and postorder traversal of a tree, construct the binary tree.
Note: You may assume that duplicates do not exist in the tree.[LeetCode](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/description/)
```cpp
For example, given inorder = [9,3,15,20,7], postorder = [9,15,7,20,3]
Return the following binary tree:
    3
   / \
  9  20
    /  \
   15   7

/**递归法
对于后序（左-右-中）中的最后一个元素，找到中序中的对应元素的位置（可借助一个hash表直接存取，无需每次遍历），然后以该位置为中点，
将中序序列分成两部分递归进行，同时确定两个子序列的后序起点和终点
*/
class Solution {
public:
    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        if(inorder.empty()) return NULL;
        unordered_map<int, int> mp;
        for(int i = 0;i < inorder.size(); i++)
            mp[inorder[i]] = i;
        TreeNode* root = recursive(inorder, postorder, 0, inorder.size() - 1, 0, postorder.size() - 1, mp);
        return root;
    }
private:
    TreeNode* recursive(vector<int>& inorder, vector<int>& postorder, int inStart, int inEnd, int postStart, int postEnd, unordered_map<int, int>& mp)
    {
        if(inStart > inEnd || postStart > postEnd)  return NULL;
        TreeNode* node = new TreeNode(postorder[postEnd]);  //后序序列的最后一个结点
        int inRoot = mp[node->val];
        int numLeft = inRoot - inStart;
        node->left = recursive(inorder, postorder, inStart, inRoot - 1, postStart, postStart + numLeft - 1, mp);
        node->right = recursive(inorder, postorder, inRoot + 1, inEnd, postStart + numLeft, postEnd - 1, mp);
        return node;
    }
};

/**迭代法
使用两个变量i,j分别表示后序和中序的索引，并依次从后往前遍历后序和中序序列。将后序中的元素从后往前压入栈中，进栈的同时依次生成
每颗子树的右子结点，直至栈顶元素和中序元素一致，说明该结点为某颗子树的根结点，这时应该为该结点插入左子结点，并将该该结点出栈。
*/
class Solution {
public:
    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        if(inorder.empty()) return NULL;
        stack<TreeNode*> st;
        int size = inorder.size();
        TreeNode* root = new TreeNode(postorder[size - 1]);
        st.push(root);
        TreeNode* t = root;
        int i = size - 2, j = size - 1, flag = 0;   //使用flag标记是插入左子结点还是右子结点
        while(i >= 0)
        {
            if(!st.empty() && st.top()->val == inorder[j])
            {
                t = st.top();
                st.pop();
                flag = 1;       //只有在回溯时（栈实现）遇到到子树的根结点才会左转生成左子结点
                j--;
            }
            else
            {
                if(flag == 0)
                {
                    t->right = new TreeNode(postorder[i]);
                    t = t->right;
                    st.push(t);
                    i--;
                }
                else
                {
                    flag = 0;       //每次插入左子结点重置flag标志
                    t->left = new TreeNode(postorder[i]);
                    t = t->left;
                    st.push(t);
                    i--;
                }
            }
        }
        return root;
    }
};
```