---
title: 数组(Array)问题集锦（二）
date: 2018-03-24
categories: Data Structure
tags: [code, array, linkedlist, two pointers]
---
数据结构与算法中数组（Array）问题总结归纳。
<!--more-->

### Merge Sorted Array
[Description](https://leetcode.com/problems/merge-sorted-array/description/): Given two sorted integer arrays nums1 and nums2, merge nums2 into nums1 as one sorted array. 
You may assume that nums1 has enough space (size that is greater or equal to m + n) to hold additional elements from nums2. The number of elements initialized in nums1 and nums2 are m and n respectively.
```cpp
/**
nums1空间足够，可从后往前比较两个数组，并从nums1(m+n-1)尾部开始填充较大值，如果nums1数组遍历结束，只剩下nums2数组，则逐个复制
nums2剩下元素，如果nums2数组遍历结束，只剩下nums1数组，则不需要额外处理，因为合并后的元素都是存在nums1中
*/
void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) 

    int i = m - 1, j = n - 1, k = m + n - 1;
    while(i >= 0 && j >= 0)
    {
        if(nums1[i] > nums2[j])
        {
            nums1[k] = nums1[i];
            i--;
        }
        else
        {
            nums1[k] = nums2[j];
            j--;
        }          
        k--;
    }
    while(j >= 0)
        nums1[k--] = nums2[j--];          
}

/**
调用STL函数merge，注意合并后的元素不能直接填充到nums1中，需要另外定义一个数组存储空间
*/
void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) 
{
    vector<int> result(m + n);
    std::merge(nums1.begin(), nums1.begin() + m, nums2.begin(), nums2.begin() + n, result.begin());
    nums1 = result;
}
```

### Merge Two Sorted Lists
[Description](https://leetcode.com/problems/merge-two-sorted-lists/description/): Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.
Example:
Input: 1->2->4, 1->3->4
Output: 1->1->2->3->4->4
```cpp

/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode* start = new ListNode(0);
        ListNode *pNode = start;
        while(l1 && l2)
        {
            if(l1->val > l2->val)
            {
                pNode->next = l2;
                l2 = l2->next;
            }
            else
            {
                pNode->next = l1;
                l1 = l1->next;
            }
            pNode = pNode->next;
        }
        if(l1)  
            pNode->next = l1;
        else
            pNode->next = l2;
        
        return start->next;
    }
};


/*递归解法*/
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) 
{
    if(l1 == NULL) return l2;
    if(l2 == NULL) return l1;
    
    if(l1->val < l2->val) 
    {
        l1->next = mergeTwoLists(l1->next, l2);
        return l1;
    } 
    else 
    {
        l2->next = mergeTwoLists(l2->next, l1);
        return l2;
    }
}
```

