---
title: 数组(Array)和双指针
date: 2018-03-29
categories: Data Structure
tags: [code, array, two pointers]
---
数据结构与算法中数组（Array）问题总结归纳。
<!--more-->

### Remove Element
Given an array and a value, remove all instances of that value in-place and return the new length. Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory. The order of elements can be changed. It doesn't matter what you leave beyond the new length.
Example: Given nums = [3,2,2,3], val = 3, Your function should return length = 2, with the first two elements of nums being 2.
```cpp
int removeElement(vector<int>& nums, int val) 
{
    int i = 0, j = 0;
    while(i < nums.size())
    {
        if(nums[i] == val)
            i++;
        else
            nums[j++] = nums[i++];
    }
    return j;
}
```

### Container With Most Water
Given n non-negative integers a1, a2, ..., an, where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water. Note: You may not slant the container and n is at least 2.
```cpp
/*
从两边往中间遍历，如果左边高度height[left]小于右边height[right]，那么height[left]与
中间剩余线段的组合面积只会减小(right--)，因此选择left++，时间复杂度为O(n)
*/
class Solution {
public:
    int maxArea(vector<int>& height) {
        int size = height.size();
        int area = 0, maxArea = 0;
        int left = 0, right = size - 1;
        while(left < right)
        {
            if(height[left] > height[right])
            {   
                area = height[right] * (right - left);
                right--;               
            }
            else
            {
                area = height[left] * (right - left);
                left++;
            }   
            if(area > maxArea)
                maxArea = area;
        }
        return maxArea;
    }
};
```

### Trapping Rain Water
Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.
For example, Given [0,1,0,2,1,0,1,3,2,1,2,1], return 6. See https://github.com/SenitCo/Algorithm/blob/master/images/42_rainwatertrap.png
```cpp
/**
Instead of calculating area by height*width, we can think it in a cumulative way. In other words, 
sum water amount of each bin(width=1). Search from left to right and maintain a max height of left 
and right separately, which is like a one-side wall of partial container. Fix the higher one and 
flow water from the lower part. For example, if current height of left is lower, we fill water in 
the left bin. Until left meets right, we filled the whole container.
从两边向中间遍历，累积计算储水量总和
*/
class Solution {
public:
    int trap(vector<int>& height) {
        int left = 0, right = height.size() - 1;
        int maxLeft = 0, maxRight = 0, sum = 0;
        while(left <= right)
        {
            if(height[left] < height[right])
            {
                if(height[left] >= maxLeft)
                    maxLeft = height[left];
                else
                    sum += maxLeft - height[left];
                left++;
            }
            else
            {
                if(height[right] >= maxRight)
                    maxRight = height[right];
                else
                    sum += maxRight - height[right];
                right--;
            }
        }
        return sum;
    }
};

```

### Remove Duplicates from Sorted Array
[Description](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/): Given a sorted array, remove the duplicates in-place such that each element appear only once and return the new length. Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.
Example: Given nums = [1,1,2], Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively. It doesn't matter what you leave beyond the new length.
```cpp
/*直接使用STL的unique函数，不过向量的size不会改变，需重新计数*/
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        auto iters = unique(nums.begin(), nums.end());
        int cnt = 0;
        for(auto it = nums.begin(); it != iters; it++, cnt++);
        return cnt;           
    }
};

//借助快慢指针
int removeDuplicates(vector<int>& nums) 
{
    if(nums.size() <= 1)    
        return nums.size();
    int i = 0;
    for(int j = i + 1; j < nums.size(); j++)
    {
        if(nums[i] != nums[j])
            nums[++i] = nums[j];
    }
    return i + 1;
}

int removeDuplicates(vector<int>& nums) 
{
    if (nums.size() < 2)
        return nums.size();
    int cnt = 1;
    for(int i = 1; i < nums.size(); i++)
    {
        if(nums[i] != nums[i - 1])
            nums[cnt++] = nums[i];
    }
    return cnt;
}

```

### Remove Duplicates from Sorted Array II
[Description](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/description/): Follow up for "Remove Duplicates": What if duplicates are allowed at most twice?

For example, Given sorted array nums = [1,1,1,2,2,3], Your function should return length = 5, with the first five elements of nums being 1, 1, 2, 2 and 3. It doesn't matter what you leave beyond the new length.
```cpp

/**
sum变量统计所有有效元素的个数，count变量统计连续相同元素的个数，如果超过两个，则只在sum后面复制两个相同元素
*/
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int sum = 0, count = 0;
        for(int i = 0; i < nums.size();)
        {
            count = 1;
            while(i + count < nums.size() && nums[i + count] == nums[i])
                count++;
            i += count;
            count = count > 2 ? 2 : count;
            for(int j = sum; j < sum + count; j++)
                nums[j] = nums[i - count];  
            sum += count;        
        }
        return sum;
    }
};

/**
i为慢指针，j为快指针，cnt统计重复元素出现的次数
*/
int removeDuplicates(vector<int>& nums) 
{
    if(nums.size() <= 2)
        return nums.size();
    int i = 0;
    for(int j = 1; j < nums.size();)
    {
        int cnt = 1;
        while(j < nums.size() && nums[i] == nums[j])
        {
            cnt++;
            j++;
        }
        if(cnt >= 2)
        {
            nums[i + 1] = nums[i];
            if(j < nums.size())     //需要考虑边界情况，否则会越界
            {
                nums[i + 2] = nums[j];
                i = i + 2;
                j++;
            }
            else
                i++;
            
        }
        else
        {
            nums[i + 1] = nums[j];
            i++;
            j++;
        }
    }
    return i + 1;
}

/**
只保留k个相同元素的通用解法，注意数组是已经经过排序的
n表示fast指针，用于遍历每一个元素；i则是slow指针，表示有效元素的个数，
并在遍历过程中，逐个添加有效元素
*/
class Solution {
public:
    int removeDuplicates(vector<int>& nums, int k) {
        int i = 0;
        for (int n : nums)
            if (i < k || n > nums[i-k])
                nums[i++] = n;
        return i;
    }
};

/**
i表示fast指针，j表示slow指针
*/
class Solution {
public:
    int removeDuplicates(vector<int>& nums, int k) {
        if(nums.size() <= k)
            return nums.size();
        int cnt = 1, i = 1, j = 1;
        while (i < nums.size())
        {
            if(nums[i] != nums[i - 1])
            {
                cnt = 1;
                nums[j++] = nums[i];
            }
            else if(cnt < k)
            {
                nums[j++] = nums[i];
                cnt++;
            }
            i++;
        }
        return j;
    }
};
```

### Sort Colors
[Description](https://leetcode.com/problems/sort-colors/description/): Given an array with n objects colored red, white or blue, sort them so that objects of the same color are adjacent, with the colors in the order red, white and blue. Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.
Note: You are not suppose to use the library's sort function for this problem.
Follow up: A rather straight forward solution is a two-pass algorithm using counting sort.
First, iterate the array counting number of 0's, 1's, and 2's, then overwrite array with total number of 0's, then 1's 
and followed by 2's. Could you come up with an one-pass algorithm using only constant space?
```cpp
/**
一次遍历，借助两个“指针”，分别指向调整位置后，0的下一个元素和2的前一个元素，对2的判断一定要在0之前，如果当前元素是2，
那么将后面的元素调整至当前位置时，还需进一步判断该元素是否为0；如果当前元素是0，那么将前面的元素调整至当前位置时，
不需要再判断该元素的值，因为前面的元素是确定的已经经过判断的，只可能为1或者0
*/
class Solution {
public:
    void sortColors(vector<int>& nums) {
        int zero = 0, two = nums.size() - 1;
        for(int i = 0; i <= two; i++)
        {
            while(nums[i] == 2 && i < two) swap(nums[i], nums[two--]);
            while(nums[i] == 0 && i > zero) swap(nums[i], nums[zero++]);
        }
    }
};
```