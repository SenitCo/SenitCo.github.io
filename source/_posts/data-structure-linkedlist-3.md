---
title: 链表问题集锦（三）
date: 2018-03-28
categories: Data Structure
tags: [code, linked list]
---

### Remove Duplicates from Sorted List
[Description](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/description/): Given a sorted linked list, delete all duplicates such that each element appear only once.
Example 1: Input: 1->1->2, Output: 1->2
Example 2: Input: 1->1->2->3->3, Output: 1->2->3
```cpp
ListNode* deleteDuplicates(ListNode* head) 
{
    ListNode *pNode = head;
    ListNode *pNext = pNode;
    while(pNode)
    {
        pNext = pNode->next;
        if(pNext && pNode->val == pNext->val)
        {
            pNode->next = pNext->next;
            delete pNext;
        }
        else
            pNode = pNext;
    }
    return head;
}
```

### Remove Duplicates from Sorted List II
[Description](https://leetcode.com/problems/remove-duplicates-from-sorted-list/description/): Given a sorted linked list, delete all nodes that have duplicate numbers, leaving only distinct numbers from the original list.
For example, Given 1->2->3->3->4->4->5, return 1->2->5. Given 1->1->1->2->3, return 2->3.
```cpp
/**
利用两个指针pre、cur，pre指向重复元素区间的前一个元素，cur指向区间内的最后一个元素，比较cur指针相邻元素的值，如果不存在重复元素，则有pre->next = cur，移动pre指针继续遍历，即pre = pre->next；否则去掉重复元素，使pre->next = cur->next
*/
ListNode* deleteDuplicates(ListNode* head) 
{
    ListNode *pNode = new ListNode(0);
    pNode->next = head;
    ListNode *pre = pNode, *cur = head;
    while(cur)
    {
        while(cur->next && cur->val == cur->next->val)
            cur = cur->next;
        if(pre->next == cur)
            pre = pre->next;
        else
            pre->next = cur->next;
        cur = cur->next;
    }
    return pNode->next;
}
```