---
title: 链表问题集锦
date: 2018-02-01
categories: Data Structure
tags: code
---

将数据结构中一些经典的链表问题做一个总结归纳。
<!--more-->

单链表结点定义：
```cpp
struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};

```

### 在O(1)时间内删除指定链表结点
题目描述：给定链表的头指针和一个结点指针，在O(1)时间内删除该结点。
分析：一般做法是借助两个指针p1、p2（一前一后），从头开始遍历，直到后一个指针p2和待删除结点相同，然后使p1->next=p2->next，并释放掉p2结点。但此处强调是O(1)时间，因此可直接将待删除结点p的值替换为下一个结点的值，并使p->next=p->next->next，并删除下一个结点。当然前提是待删除结点不能是尾结点，即其下一个结点不能为空。
```cpp
void deleteNode(ListNode *node)
{
	if(!node || !node->next)
		return;
	ListNode* pNext = cur->next;
	node->val = pNext->val;
	node->next = pNext->next;
	delete pNext;
}

```

### 单链表逆转
题目描述：给定一个单链表，将其逆转。[LeetCode](https://leetcode.com/problems/reverse-linked-list/description/)
分析：迭代法求解，借助三个指针start、head、next，start指向逆转后（已重排）的第一个结点，head指向未重排的第一个结点，next指向head的下一个结点，每次将未重排的第一个结点插入到已重排序列的首部。也可利用递归法实现。
```cpp
ListNode* reverseList(ListNode* head) 
{
    ListNode *start = NULL, next;
    while(head)
    {
        next = head->next;
        head->next = start;
        start = head;
        head = next;
    }
    return start;
}

//增加一个头结点的迭代版本
ListNode* reverseList(ListNode* head) 
{
    if(!head)
        return NULL;
    ListNode* start = new ListNode(0);
    start->next = head;
    ListNode *first = head;
    ListNode *second = first->next;
    while(second)
    {
        first->next = second->next;
        second->next = start->next;
        start->next = second;
        second = first->next;
    }
    return start->next;
}

//递归法
ListNode* reverseList(ListNode* head) 
{
    return recursive(head, NULL);
}

ListNode* recursive(ListNode* head, ListNode* start)
{
    if(!head)   return start;
    ListNode* next = head->next;
    head->next = start;
    return recursive(next, head);
}

//直接递归
Node* reverseByRecursion(Node *head)
{
    //第一个条件是判断异常，第二个条件是结束判断
    if(head == NULL || head->next == NULL) 
        return head;
    Node *newHead = reverseByRecursion(head->next);
    head->next->next = head;
    head->next = NULL;
    return newHead;    //返回新链表的头指针
}

```

###求链表倒数第k个结点
题目描述：给定一个单链表，找到倒数第k个结点（或移除该结点），例如链表 1->2->3->4->5，k = 2，则倒数第k个结点值为4，移除该结点后，链表为1->2->3->5。[LeetCode](https://leetcode.com/problems/remove-nth-node-from-end-of-list/description/)
分析：借助快慢两个指针fast、slow，首先fast先正向移动到第n个结点，然后两个指针同步移动，直至fast指针为空（走到链表末尾），此时slow指针所指向的即是倒数第k个结点。
```cpp
//找到倒数第n个结点
ListNode* findNthFromEnd(ListNode* head, int n)
{
	if(n < 0)	return NULL;
	ListNode *slow = head, *fast = head;
	for(int i = 0; i < n && fast; i++)
	{
		fast = fast->next;
	}
	if(i < n)	return NULL;
	while(fast)
	{
		slow = slow->next;
		fast = fast->next;
	}
	return slow;
}

//删除倒数第n个结点
ListNode* removeNthFromEnd(ListNode* head, int n) 
{
	if(n < 0)	return NULL;
    ListNode* start = new ListNode(0);	//加一个起始（首）结点可有效解决只有一个元素的情况
    ListNode *front = start, *back = start;
    back->next = head;
    for(int i = 0; i < n + 1 && front; i++)
    {
        front = front->next;
    }
    if(i < n + 1)	return NULL;
    while(front)
    {
        front = front->next;
        back = back->next;
    }
    front = back->next;
    back->next = back->next->next;
    free(front);
    return start->next;
}
```

### 求链表的中间节点
题目描述：求链表的中间结点，如果链表长度为奇数，则返回中间结点；若为偶数，则返回中间两个结点中的任意一个。
分析：一般思路是先完整遍历一次链表，求得整个链表的长度，然后长度取半，再从头开始移动指针，即可定位到中间结点。如果只允许遍历一次链表的话，可通过两个指针完成，两个指针均从头部结点开始，一个每次移动一步，另一个每次移动两步，这样快指针走到链表末尾时，慢指针刚好指向中间结点。
```cpp
ListNode* findMiddleNode(ListNode* head)
{
	if(head == NULL)	return NULL;
	ListNode *slow = head, *fast = head;
	//while(fast->next && fast->next->next)	//对于链表长度为偶数的情况，返回中间两个结点中的第一个
	while(fast && fast->next)	//对于链表长度为偶数的情况，返回中间两个结点中的第二个
	{
		fast = fast->next->next;
		slow = slow->next;
	}
	return slow;
}
```

### 判断单链表是否存在环
题目描述：输入一个单链表，判断链表是否存在环。[LeetCode](https://leetcode.com/problems/linked-list-cycle/description/)
分析：借助两个指针，分别从头结点出发，一个指针每次移动一个步长，另一个指针每次移动两个步长，如果存在环，则两个指针必在环内相遇。
```cpp
bool hasCycle(ListNode *head) 
{
    if(!head || !head->next)
        return NULL;
    ListNode *p1, *p2;
    p1 = p2 = head;
    while(p2 && p2->next)
    {
        p1 = p1->next;
        p2 = p2->next->next;
        if(p1 == p2)  
        	return true;
    }
    return false;
}
```

### 找到单链表成环的入口点
题目描述，如果单链表存在环，则找到环的入口点，若不存在，则返回NULL。[LeetCode](https://leetcode.com/problems/linked-list-cycle-ii/description/)
分析：借助两个指针，分别从头结点出发，一个指针每次移动一个步长，另一个指针每次移动两个步长，如果存在环，则两个指针必在环内相遇。然后一个指针指向链表头结点，另一个指针仍位于相遇点，两个指针均以单步长移动，再次相遇点即为环的起始点。设第一次相遇点距离链表起点为k，环起始点距离链表起点为s，环起始点到第一次相遇点距离为m，环长度为r，则有k = s + m, 且2k - k = nr = k, 即 s = nr - m。
```cpp
ListNode *detectCycle(ListNode *head) 
{
    if(!head || !head->next)
        return NULL;
    ListNode *p1, *p2;
    p1 = p2 = head;
    bool flag = false;
    while(p2 && p2->next)
    {
        p1 = p1->next;
        p2 = p2->next->next;
        if(p1 == p2)  
        {
            flag = true;
            break;
        }
    }
    if(!flag)   return NULL;
    
    p1 = head;
    while(p1 != p2)
    {
        p1 = p1->next;
        p2 = p2->next;
    }
    return p1;
}
```

### 判断两个链表是否相交
