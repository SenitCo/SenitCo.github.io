---
title: 链表排序
date: 2018-03-24
categories: Data Structure
tags: [code, linked list, sort, two pointers]
---
链表排序问题总结归纳。
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

### Sort List
[Description](https://leetcode.com/problems/sort-list/description/): Sort a linked list in O(n log n) time using constant space complexity.
```cpp
/**
归并排序的递归解法，不符合要求，空间复杂度为O(logn)，自顶向下执行
*/
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if(head == NULL || head->next == NULL)
            return head;
        ListNode *prev = NULL, *slow, *fast;
        slow = fast = head;
        while(fast && fast->next)
        {
            prev = slow;
            slow = slow->next;
            fast = fast->next->next;    //快指针每次走两步，最终慢指针指向链表中间
        }
        prev->next = NULL;
        ListNode* l1 = sortList(head);
        ListNode* l2 = sortList(slow);
        return mergeList(l1, l2);
    }
private:
    ListNode* mergeList(ListNode* l1, ListNode* l2)
    {
        ListNode* node = new ListNode(0);
        ListNode* p = node;
        while(l1 && l2)
        {
            if(l1->val < l2->val)
            {
                p->next = l1;
                l1 = l1->next;
            }
            else
            {
                p->next = l2;
                l2 = l2->next;
            }
            p = p->next;
        }
        if(l1)
            p->next = l1;
        if(l2)
            p->next = l2;
        return node->next;
    }
};

/**
归并排序的迭代解法，空间复杂度为O(1)，自底向上执行
*/
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if(head == NULL || head->next == NULL)
            return head;
        ListNode *cur = head;
        int length = 0;
        while(cur)
        {
            length++;
            cur = cur->next;
        }
        ListNode *start = new ListNode(0);
        start->next = head;
        ListNode *left, *right, *tail;
        for(int step = 1; step < length; step <<= 1)
        {
            cur = start->next;
            tail = start;
            while(cur)  //每次两两归并，得到一个有序序列
            {
                left = cur;
                right = split(left, step);
                cur = split(right, step);
                tail = merge(left, right, tail);
            }
        }
        return start->next;
    }
    
private:
    //将链表划分为两个链表，每个链表n个元素，并返回下一个链表的首部
    ListNode* split(ListNode* head, int n)
    {
        for(int i = 1; head && i < n; i++)
            head = head->next;
        if(!head)    return NULL;
        ListNode* second = head->next;
        head->next = NULL;  
        return second;
    }
    //合并两个链表，并返回合并后的链表的尾部，即下一个链表的首部
    ListNode* merge(ListNode* l1, ListNode* l2, ListNode* head)
    {
        ListNode* cur = head;
        while(l1 && l2)
        {
            if(l1->val < l2->val)
            {
                cur->next = l1;
                l1 = l1->next;
            }
            else
            {
                cur->next = l2;
                l2 = l2->next;
            }
            cur = cur->next;
        }
        cur->next = l1 ? l1 : l2;
        while(cur->next)  cur = cur->next;
        return cur;
    }
   
};
```

### Insertion Sort List
[Description](https://leetcode.com/problems/insertion-sort-list/description/): Sort a linked list using insertion sort.
```cpp
ListNode* insertionSortList(ListNode* head) 
{
    if(head == NULL || head->next == NULL)
        return head;
    ListNode *start = new ListNode(0);
    ListNode *cur = head, *pre = start, *next = NULL;
    while(cur)
    {
        pre = start;
        while(pre->next && pre->next->val < cur->val)
            pre = pre->next;
        next = cur->next;
        cur->next = pre->next;
        pre->next = cur;
        cur = next;
    }
    return start->next;
}
```

### Merge k Sorted Lists
[Description](https://leetcode.com/problems/merge-k-sorted-lists/description/): Merge k sorted linked lists and return it as one sorted list. Analyze and describe its complexity.
```cpp
//借助堆来实现，每个链表是有序的，堆中包含k个元素，分别是每个链表中最小的的元素，因此每次可以从堆中取出一个最小的元素
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

//直接借助优先队列实现（优先队列通过堆来实现）
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
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

    struct cmp
    {
        bool operator()(ListNode* l1, ListNode* l2)
        {
            return l1->val > l2->val;
        }
    };
};

/*递归解法，将有序列表两两合并*/
class Solution {
public:
    ListNode *mergeKLists(vector<ListNode *> &lists) {
        if(lists.empty()){
            return nullptr;
        }
        while(lists.size() > 1){
            lists.push_back(mergeTwoLists(lists[0], lists[1]));
            lists.erase(lists.begin());
            lists.erase(lists.begin());
        }
        return lists.front();
    }
    
    ListNode *mergeTwoLists(ListNode *l1, ListNode *l2) {
        if(l1 == nullptr){
            return l2;
        }
        if(l2 == nullptr){
            return l1;
        }
        if(l1->val <= l2->val)
        {
            l1->next = mergeTwoLists(l1->next, l2);
            return l1;
        }
        else
        {
            l2->next = mergeTwoLists(l1, l2->next);
            return l2;
        }
    }
};
```

### Partition List
[Description](https://leetcode.com/problems/partition-list/description/): Given a linked list and a value x, partition it such that all nodes less than x come before nodes 
greater than or equal to x. You should preserve the original relative order of the nodes in each 
of the two partitions.
For example, Given 1->4->3->2->5->2 and x = 3, return 1->2->2->4->3->5.
```cpp
/**
借助两个指针，less和greater，分别指向小于目标值的元素和大于等于目标值的元素，遍历一次，即可得到两个链表，
然后将其合并为一个
*/
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        ListNode* less = new ListNode(0);
        ListNode* greater = new ListNode(0);
        ListNode *ptr1 = less, *ptr2 = greater, *pNode = head;
        while(pNode)
        {
            if(pNode->val < x)
            {
                ptr1->next = pNode;
                ptr1 = ptr1->next;
            }
            else
            {
                ptr2->next = pNode;
                ptr2 = ptr2->next;
            }
            pNode = pNode->next;
        }
        ptr1->next = greater->next;
        ptr2->next = NULL;
        return less->next;
    }
};
```


