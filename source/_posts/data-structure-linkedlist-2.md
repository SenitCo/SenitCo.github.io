---
title: 链表问题集锦（二）
date: 2018-02-02
categories: Data Structure
tags: [code, linked list]
---

数据结构中经典链表问题总结归纳，接上一篇[链表问题集锦（一）](https://senitco.github.io/2018/02/01/data-structure-linkedlist/)
<!--more-->

### 单链表逆转
题目描述：给定一个单链表，将其逆转。[LeetCode](https://leetcode.com/problems/reverse-linked-list/description/)
```cpp
/**
迭代法求解，借助三个指针start、head、next，start指向逆转后（已重排）的第一个结点，head指向未重排的第一个结点，
next指向head的下一个结点，每次将未重排的第一个结点插入到已重排序列的首部。也可利用递归法实现。
*/
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

/**
增加一个头结点的迭代版本，start为头指针，start->next指向已重排的第一个结点，last指向已重排的最后一个结点
（原始链表的第一个结点），curr指向未重排的第一个结点（即将插入到重排序列中），last->next指向curr的下一个
结点即curr->next。
*/
ListNode* reverseList(ListNode* head) 
{
    if(!head)
        return NULL;
    ListNode* start = new ListNode(0);
    start->next = head;
    ListNode *last = head;
    ListNode *curr = head->next;
    while(curr)
    {
        last->next = curr->next;
        curr->next = start->next;
        start->next = curr;
        curr = last->next;
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

### 将单链表中的某一段逆转
问题描述：将单链表中第m到第n个结点之间的序列逆转。[LeetCode](https://leetcode.com/problems/reverse-linked-list-ii/description/)
```cpp
/**
思路和逆转整个链表一样，借助三个指针，prev指向链表中的第 m-1 个元素，prev->next指向重排后的第一个结点，
last指向已重排区间的最后一个元素（原始未重排区间的第一个元素），curr指向未重排序列的的第一个元素（即将插入
到重排序列中），last->next指向curr的下一个结点即curr->next，保证所有元素能连接成串。
*/
ListNode* reverseBetween(ListNode* head, int m, int n) 
{
    ListNode* start = new ListNode(0);  
    start->next = head;
    ListNode *prev = start;
    for(int i = 0; i < m - 1; i++)
        prev = prev->next;
    ListNode* last = prev->next;
    ListNode* curr = last->next;
   
    for(int i = 0; i < n - m && curr; i++)
    {
        last->next = curr->next;
        curr->next = prev->next;
        prev->next = curr;
        curr = last->next;
    }
    return start->next;
}

ListNode* reverseBetween(ListNode* head, int m, int n) 
{
    ListNode* start = new ListNode(0);
    ListNode* cur, *nex, *tmp;
    start->next = head;
    cur = start;
    int i = 0;
    for(; i < m - 1; i++)
    {
        cur = cur->next;
    }
    nex = cur->next;
    for(; i < n - 1 && nex->next; i++)  //i初值为 m-1，后续插入操作次数为 n-m
    {
        tmp = cur->next;                //暂时存储cur后面的元素
        cur->next = nex->next;          //将nex后面的元素插入到cur后
        nex->next = nex->next->next;    //取出nex的下一个元素后，nex的下一个元素变成更后面的元素
        cur->next->next = tmp;          //将插入到cur的元素与之前暂存的元素链接起来
    }
    return start->next;
}

```

### 将单链表进行分组逆转
问题描述：将单链表按顺序分成N组，每组K个元素，最后一组可能少于K个，分别将每组元素逆转，并链接成一个完整的序列。例如一个链表：1->2->3->4->5，如果k=2，则逆转后的序列为2->1->4->3->5；如果k=3，则逆转序列为3->2->1->4->5。[LeetCode](https://leetcode.com/problems/reverse-nodes-in-k-group/description/)
分析：分别对每组结点进行逆转，可用迭代法和递归法
```cpp
/**迭代法：
-1 -> 1 -> 2 -> 3 -> 4 -> 5
 |    |    |     
pre  cur  nex  

-1 -> 2 -> 1 -> 3 -> 4 -> 5
 |         |    |     
pre       cur  nex  

-1 -> 3 -> 2 -> 1 -> 4 -> 5
 |              |    |    
pre            cur  nex 

三个重要指针，头指针pre，始终指向已重新排列的第一个元素；当前指针cur，指向已排序的最后一个元素；
cur->next则指向未排序的第一个元素也就是nex指针，nex的作用是即将作为第一个元素赋给pre->next.
*/
ListNode* reverseKGroup(ListNode* head, int k) 
{
    if(head == nullptr || k == 1)
        return head;
    int num = 0;
    ListNode* preheader = new ListNode(0);
    ListNode *pre, *nex, *cur;
    preheader->next = head;
    cur = pre = preheader;
    while(cur = cur->next)
        num++;
    while(num >= k)
    {
        cur = pre->next;
        nex = cur->next;
        for(int i = 1; i < k; i++)
        {
            cur->next = nex->next;
            nex->next = pre->next;
            pre->next = nex;
            nex = cur->next;
        }
        pre = cur;
        num -= k;
    }
    return preheader->next;
}

/**递归法：
head：指向正序起始节点
curr：指向反转后的起始节点
tmp：head->next，用于保存下一个插入反转序列的节点
*/
ListNode* reverseKGroup(ListNode* head, int k) 
{
    ListNode* curr = head;
    int count = 0;
    while (curr != nullptr && count != k) { // find the k+1 node
        curr = curr->next;
        count++;
    }
    if (count == k) { // if k+1 node is found
        curr = reverseKGroup(curr, k); // reverse list with k+1 node as head
        // head - head-pointer to direct part, 
        // curr - head-pointer to reversed part;
        while (count-- > 0) { // reverse current k-group: 
            ListNode* tmp = head->next; // tmp - next head in direct part
            head->next = curr; // preappending "direct" head to the reversed list 
            curr = head; // move head of reversed part to a new node
            head = tmp; // move "direct" head to the next node in direct part
        }
        head = curr;
    }
    return head;
}
```

### 将单链表的结点两两交换
题目描述：给定一个单链表，将相邻两个结点两两交换。例如链表：1->2->3->4, 交换后的序列为2->1->4->3。[LeetCode](https://leetcode.com/problems/swap-nodes-in-pairs/description/)
```cpp
ListNode* swapPairs(ListNode* head) 
{
    ListNode* start = new ListNode(0);
    start->next = head;
    ListNode *pre = start, *cur, *nex;
    while(pre->next && pre->next->next)
    {
        cur = pre->next;
        nex = cur->next;
        cur->next = nex->next;
        nex->next = pre->next;
        pre->next = nex;
        pre = cur;
    }
    return start->next;
}

//遍历元素，两两交换
ListNode* swapPairs(ListNode* head) 
{
    ListNode* start = new ListNode(0);
    start->next = head;
    ListNode* current = start;
    ListNode *first, *second;
    while(current->next && current->next->next)
    {
        first = current->next;
        second = first->next;
        first->next = second->next;
        current->next = second;
        second->next = first;
        current = current->next->next;
    }
    return start->next;
}

//递归法
ListNode* swapPairs(ListNode* head) 
{
    if(head == nullptr || head->next == nullptr)
        return head;
    ListNode* pNode = head->next;
    head->next = swapPairs(head->next->next);
    pNode->next = head;
    return pNode;
}

//二级指针
ListNode* swapPairs(ListNode* head) 
{
    ListNode **pp = &head, *a, *b;
    while ((a = *pp) && (b = a->next)) 
    {
        a->next = b->next;
        b->next = a;
        *pp = b;    //改变指针指指向的地址
        pp = &(a->next);
    }
    return head;
}
```

### 按特定序列将单链表重排
问题描述：Given a singly linked list L: L0→L1→…→Ln-1→Ln, reorder it to: L0→Ln→L1→Ln-1→L2→Ln-2→…
You must do this in-place without altering the nodes' values.
For example, Given {1,2,3,4}, reorder it to {1,4,2,3}.[LeetCode](https://leetcode.com/problems/reorder-list/description/)
分析：分三步走：
(1)找到链表的中点位置middle=(len-1)/2
(2)将后半部分（middle之后的元素）链表逆转
(3)同时遍历两部分链表元素，依次将后半部分元素插入到前半部分的相应位置
```cpp
void reorderList(ListNode* head) 
{
    if(!head || !head->next)
        return;
    ListNode *p1, *p2;
    p1 = p2 = head;
    //Find the middle of the list, 1->2->3->4->5->6, middle point to 3
    while(p2->next && p2->next->next)
    {
        p1 = p1->next;
        p2 = p2->next->next;
    }
    //Reverse the half after middle  1->2->3->4->5->6 to 1->2->3->6->5->4
    ListNode *middle = p1, *prev = p1->next, *cur;
    while(prev->next)
    {
        cur = prev->next;
        prev->next = cur->next;
        cur->next = middle->next;
        middle->next = cur;
    }
    //Start reorder one by one  1->2->3->6->5->4 to 1->6->2->5->3->4
    p1 = head;
    p2 = middle->next;
    while(p1 != middle)
    {
        middle->next = p2->next;
        p2->next = p1->next;
        p1->next = p2;
        p1 = p2->next;
        p2 = middle->next;
    }
}
```

### 合并两个有序链表
问题描述：将两个有序链表合并成一个新的有序链表。[LeetCode](https://leetcode.com/problems/merge-two-sorted-lists/description/)
```cpp
//迭代法
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) 
{
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

//递归法
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

### 合并K个有序链表
问题描述：[LeetCode](https://leetcode.com/problems/merge-k-sorted-lists/description/)
```cpp
/**
法一：与合并两个有序列表的思想一致，每次比较 K 个节点，取其最小值；在遍历过程中，
若有列表已到达末尾节点，则将该列表从vector中移除
*/
ListNode* mergeKLists(vector<ListNode*>& lists) 
{
    if (lists.empty())
        return NULL;
    vector<ListNode*> pl;
    for (auto node : lists)
        pl.push_back(node);
    ListNode* start = new ListNode(0);
    ListNode* pNode = start;
    int value = INT_MAX, index = 0;
    while (true)
    {
        for (auto iter = pl.begin(); iter != pl.end();)
        {
            if (!(*iter))
            {
                iter = pl.erase(iter);  //注意erase()可能会造成后面的迭代器失效
            }
            else
                iter++;
        }
        if (pl.empty())
            break;
        value = INT_MAX;
        index = 0;
        for (int i = 0; i < pl.size(); i++)
        {
            if (pl[i]->val < value)
            {
                value = pl[i]->val;
                index = i;
            }
        }
        pNode->next = pl[index];
        pl[index] = pl[index]->next;
        pNode = pNode->next;
    }
    return start->next;
}

//法二：借助优先队列实现
struct cmp
{
    bool operator()(ListNode* l1, ListNode* l2)
    {
        return l1->val > l2->val;
    }
};

ListNode* mergeKLists(vector<ListNode*>& lists) 
{
    if (lists.empty())
        return NULL;     
    ListNode* start = new ListNode(0);
    ListNode* pNode = start;
    priority_queue<ListNode*, vector<ListNode*>, cmp> pql;
    for (auto node : lists)
        if(node)            //注意此处要判断节点(链表)是否为空
            pql.push(node);
    while(!pql.empty())
    {
        pNode->next = pql.top();
        pql.pop();
        pNode = pNode->next;
        if(pNode->next != NULL)
            pql.push(pNode->next);
    }     
    return start->next;
}

//借助最小堆实现
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        ListNode* start = new ListNode(0);
        ListNode* pNode = start;
        vector<ListNode*> minHeap;
        for(auto iter : lists)
        {
            if(iter)
                minHeap.push_back(iter);
        }
        make_heap(minHeap.begin(), minHeap.end(), cmp());
        while(!minHeap.empty())
        {
            pNode->next = minHeap.front();
            pNode = pNode->next;
            pop_heap(minHeap.begin(), minHeap.end(), cmp());
            minHeap.pop_back();
            
            if(pNode->next)
                minHeap.push_back(pNode->next);
            push_heap(minHeap.begin(), minHeap.end(), cmp());
        }
        return start->next;
    }
    
    struct cmp
    {
        bool operator()(ListNode* l1, ListNode* l2)
        {
            return l1->val > l2->val;
        }
    };
};

//将有序列表两两合并
ListNode *mergeKLists(vector<ListNode *> &lists) 
{
    if(lists.empty()){
        return nullptr;
    }
    while(lists.size() > 1){    
        lists.push_back(mergeTwoLists(lists[0], lists[1])); //mergeTwoLists()见上
        lists.erase(lists.begin());
        lists.erase(lists.begin());
    }
    return lists.front();
}
```

