---
title: 链表问题集锦（三）
date: 2018-03-28
categories: Data Structure
tags: [code, linked list]
---
数据结构中经典链表问题总结归纳。
<!--more-->

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

### Rotate List
[Description](https://leetcode.com/problems/rotate-list/description/): Given a list, rotate the list to the right by k places, where k is non-negative.
Example: Given 1->2->3->4->5->NULL and k = 2, return 4->5->1->2->3->NULL.
```cpp
/**
此处没必要利用快慢指针来定位到倒数第k个位置，因为k值可能大于链表结点数，所以必须先遍历一次获取链表的长度，
然后取模相减，得到需要移动的步长(len - k % len)，在第一次遍历至链表尾时，可直接将链表首尾相连，然后继续移动
(len - k % len)个步长，记录首结点指针start=tail->next，并使tail=NULL，解除环形
*/
class Solution {
public:
    ListNode* rotateRight(ListNode* head, int k) {
        if(!head)
            return head;
        int len = 1;
        ListNode *start = head, *tail = head;
        while(tail->next)
        {
            tail = tail->next;
            len++;
        }
        tail->next = head;  //首尾相连
        k = k % len;    //k值可大于结点数
        if(k > 0)
        {
            for(int i = 0; i < len - k; i++)
                tail = tail->next;
        }
        start = tail->next;
        tail->next = NULL;  //解除环形
        return start;
    }
};
```