---
title: 数组(Array)问题集锦（一）
date: 2018-03-22
categories: Data Structure
tags: [code, array]
---
数据结构与算法中数组（Array）问题总结归纳。
<!--more-->

### Next Permutation
[Description](https://leetcode.com/problems/next-permutation/description/): Implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers. If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order). The replacement must be in-place, do not allocate extra memory.
> Here are some examples. Inputs are in the left-hand column and its corresponding outputs are in the right-hand column.
1,2,3 → 1,3,2
3,2,1 → 1,2,3
1,1,5 → 1,5,1

```cpp
/**
实现数组的下一个排列：先找到第一个降序的数字，nums[i+1] > nums[i]，然后从后往前找到第一个大于nums[i]的数，记为nums[j]，
交换nums[i]与nums[j]的值，并翻转 i 后面的元素。
*/
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int size = nums.size();
        if(size < 2)
            return;
        int i = 0, j = 0;
        for(i = size - 1; i > 0; i--)
            if(nums[i] > nums[i - 1])   break;
        if(i == 0)
        {
            reverse(nums.begin(), nums.end());
            return;
        }
        for(j = size - 1; j >= i; j--)
            if(nums[j] > nums[i - 1])   break;
        
        swap(nums[i - 1], nums[j]);
        reverse(nums.begin() + i, nums.end());       
    }
};

void nextPermutation(vector<int>& nums) 
{
    next_permutation(begin(nums), end(nums));
}
```

### Permutation Sequence
[Description](https://leetcode.com/problems/permutation-sequence/description/): The set [1,2,3,…,n] contains a total of n! unique permutations. By listing and labeling all of the permutations in order.
> We get the following sequence (ie, for n = 3):
"123"
"132"
"213"
"231"
"312"
"321"
Given n and k, return the kth permutation sequence.
Note: Given n will be between 1 and 9 inclusive.

```cpp
/**
定义两个数组，一个nums存储原始序列{1,2,...n}，一个fact用于存储对应数的阶乘{1!,2!,...n!}，对于n个数组组成的排列，
共有fact[n]=n!种情况，每个数i为起始的情况共有batch=fact[n]/n=fact[n-1]=(n-1)!，利用(k-1)/batch可求得第一个元素
的索引为idx=(k-1)/batch+1，即result[0]=nums[(k-1)/batch+1]，将其赋值后，在数组nums中移除对应元素，然后更新相关值，
继续下一轮迭代
*/
class Solution {
public:
    string getPermutation(int n, int k) {
        vector<int> nums(n + 1, 0);
        vector<int> fact(n + 1, 1);
        string result;
        for (int i = 1; i < n + 1; i++)
        {
            nums[i] = i;
            fact[i] = fact[i - 1] * i;
        }
        int idx = k - 1;
        for (int i = n; i > 0; i--)
        {
            int batch = fact[i] / n;
            int quo = idx / batch;
            idx = idx % batch;
            result.push_back(nums[quo + 1] + '0');
            nums.erase(nums.begin() + quo + 1);
            n--;
        }
        return result;
    }
};

class Solution {
public:
    string getPermutation(int n, int k) {
        vector<int> nums(n + 1, 0);
        vector<int> fact(n + 1, 1);
        string result;
        for (int i = 1; i < n + 1; i++)
        {
            nums[i] = i;
            fact[i] = fact[i - 1] * i;
        }
        int idx = k - 1, quo = 0;
        for (int i = n; i > 0; i--)
        {
            quo = idx / fact[i - 1];    //将batch直接替换为fact[i-1]
            idx = idx % fact[i - 1];
            result.push_back(nums[quo + 1] + '0');
            nums.erase(nums.begin() + quo + 1);
        }
        return result;
    }
};

```

### Plus One
[Description](https://leetcode.com/problems/plus-one/description/): Given a non-negative integer represented as a non-empty array of digits, plus one to the integer. You may assume the integer do not contain any leading zero, except the number 0 itself. The digits are stored such that the most significant digit is at the head of the list.
```cpp
vector<int> plusOne(vector<int>& digits) 
{
    int size = digits.size();
    int step = 1, temp = 0;
    for(int i = digits.size() - 1; i >= 0; i--)
    {
        temp = digits[i];
        digits[i] = (temp + step) % 10;
        step = (temp + step) / 10;
    }
    if(step > 0)
        digits.insert(digits.begin(), 1);
    return digits;
}
```





