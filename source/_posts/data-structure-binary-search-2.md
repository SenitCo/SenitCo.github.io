---
title: 二分查找（二）
date: 2018-03-13
categories: Data Structure
tags: [code, binary search, array]
---

数据结构中二分查找问题总结归纳。
<!--more-->

### Search in Rotated Sorted Array
Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.
(i.e., 0 1 2 4 5 6 7 might become 4 5 6 7 0 1 2).
You are given a target value to search. If found in the array return its index, otherwise return -1.
You may assume no duplicate exists in the array.[LeetCode](https://leetcode.com/problems/search-in-rotated-sorted-array/description/)
```cpp
/**
二分查找的变体：由于序列存在旋转的情况，需要做进一步的判断
（1）如果 target == nums[mid]，返回mid
（2）如果 nums[mid] < nums[right]，说明右边一定是有序的；然后判断是否有 nums[mid] < target <= nums[right]，
    若是则 left = mid + 1，否则 right = mid - 1
（3）如果 nums[mid] >= nums[right]，说明左边一定是有序的；然后判断是否有 nums[left] <= target < nums[mid]，
    若是则 right = mid - 1，否则 left = mid + 1
时间复杂度为O(n)，空间复杂度为O(1)，前提条件是没有重复元素
*/
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int size = nums.size();
        if(size == 0)
            return -1;
        int left = 0, mid = 0, right = size - 1;
        while(left <= right)
        {
            mid = (left + right) / 2;
            if(nums[mid] == target)
                return mid;
            else if(nums[mid] < nums[right])
            {
                if(nums[mid] < target && nums[right] >= target)
                    left = mid + 1;
                else
                    right = mid - 1;
            }
            else
            {
                if(nums[mid] > target && nums[left] <= target)
                    right = mid - 1;
                else
                    left = mid + 1;
            }
        }
        return -1;
    }
};
```

### Search in Rotated Sorted Array II
Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.
(i.e., 0 1 2 4 5 6 7 might become 4 5 6 7 0 1 2).
Write a function to determine if a given target is in the array.
The array may contain duplicates.[LeetCode](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/description/)
```cpp
/**
重复元素的存在可能导致中间和边缘相等的情况，这样就丢失了哪边有序的信息，因为哪边都有可能是有序的结果。
假设原数组是{1,2,3,3,3,3,3}，那么旋转之后有可能是{3,3,3,3,3,1,2}，或者{3,1,2,3,3,3,3}，这样判断左边缘
和中心的时候都是3，如果要寻找1或者2，并不知道应该跳向哪一半。解决的办法只能是对边缘移动一步，直到边缘
和中间不再相等或者相遇，这就导致了会有不能切去一半的可能。所以最坏情况（比如全部都是一个元素，或者只有
一个元素不同于其他元素，而且就在最后一个）就会出现每次移动一步，总共是n步，算法的时间复杂度变成O(n)
*/
class Solution {
public:
    bool search(vector<int>& nums, int target) {
        int size = nums.size();
        if(size == 0)
            return false;
        int left = 0, mid = 0, right = size - 1;
        while(left <= right)
        {
            mid = (left + right) / 2;
            if(nums[mid] == target)
                return true;
            else if(nums[mid] > nums[left])     //注意此处是和最左边元素(left)比较，对应的else就是left++
            {                                   //如果是和right比较，那就应该是right--
                if(nums[mid] > target && nums[left] <= target)
                    right = mid - 1;
                else
                    left = mid + 1;
            }
            else if(nums[mid] < nums[left])
            {
                if(nums[mid] < target && nums[right] >= target)
                    left = mid + 1;
                else
                    right = mid - 1;
            }
            else
                left++;
        }
        return false;
    }
};
```

### Find Minimum in Rotated Sorted Array
Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand. (i.e., 0 1 2 4 5 6 7 might become 4 5 6 7 0 1 2). Find the minimum element. You may assume no duplicate exists in the array.[LeetCode](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)
```cpp
/**
如果 nums[mid] < nums[right]，说明右边一定是有序的，令right = mid（不能为mid-1，因为mid可能为最小的数）
如果 nums[mid] >= nums[right]，说明左边一定是有序的，令left = mid + 1（最小值一定在mid的右边）
*/
class Solution {
public:
    int findMin(vector<int>& nums) {
        int size = nums.size();
        int left = 0, right = size - 1, mid = 0;
        while(left < right)
        {
            mid = (left + right) / 2;
            if(nums[mid] < nums[right])
                right = mid;
            else
                left = mid + 1;
        }
        return nums[left];
    }
};
```

### Find Minimum in Rotated Sorted Array II
Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand. (i.e., 0 1 2 4 5 6 7 might become 4 5 6 7 0 1 2). Find the minimum element. The array may contain duplicates.[LeetCode](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/description/)
```cpp
/**
（1）如果 nums[mid] < nums[right]，说明右边一定是有序的，令right = mid（不能为mid-1，因为mid可能为最小的数）
（2）如果 nums[mid] > nums[right]，说明左边一定是有序的，令left = mid + 1（最小值一定在mid的右边）
（3）如果 nums[mid] == nums[right]，可能出现以下情况，唯一能确定的是最小值一定在right左边，因此right--
    [1, 3, 3]
    [3, 3, 3, 1, 3]
    [3, 1, 3, 3, 3]
*/
class Solution {
public:
    int findMin(vector<int>& nums) {
        int size = nums.size();
        int left = 0, right = size - 1, mid = 0;
        while(left < right)
        {
            mid = (left + right) / 2;
            if(nums[mid] < nums[right])
                right = mid;
            else if(nums[mid] > nums[right])
                left = mid + 1;
            else
                right--;
        }
        return nums[left];
    }
};
```