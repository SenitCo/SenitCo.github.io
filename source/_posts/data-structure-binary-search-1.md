---
title: 二分查找（一）
date: 2018-03-12
categories: Data Structure
tags: [code, binary search, array]
---

数据结构中二分查找问题总结归纳。
<!--more-->

### Search Insert Position
Given a sorted array and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order. You may assume no duplicates in the array.[LeetCode](https://leetcode.com/problems/search-insert-position/description/)
```cpp
/**
Example 1:
Input: [1,3,5,6], 5
Output: 2

Example 2:
Input: [1,3,5,6], 2
Output: 1

Example 3:
Input: [1,3,5,6], 7
Output: 4

Example 4:
Input: [1,3,5,6], 0
Output: 0
*/

int searchInsert(vector<int>& nums, int target) 
{
    int size = nums.size();
    int left = 0, mid = 0, right = size - 1;
    while(left <= right)
    {
        mid = (left + right) / 2;
        if(nums[mid] == target)
            return mid;
        else if(nums[mid] > target)
            right = mid - 1;
        else
            left = mid + 1;
    }
    return left;        //没找到的情况下插入位置应该为left
}
```

### Search for a Range
Given an array of integers sorted in ascending order, find the starting and ending position of a given target value.
Your algorithm's runtime complexity must be in the order of O(log n).
If the target is not found in the array, return [-1, -1].
For example, Given [5, 7, 7, 8, 8, 10] and target value 8, return [3, 4]. [LeetCode](https://leetcode.com/problems/search-for-a-range/description/)
```cpp
/*********二分法查找，找到目标值后前后搜索边界**********/
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        int size = nums.size();
        if(size < 1 || target < nums[0] || target > nums[size - 1])
            return vector<int>({ -1, -1 });
        int left = 0, mid = 0, right = size - 1;
        vector<int> result;
        while(left <= right)
        {
            mid = (left + right) / 2;
            if(nums[mid] == target)
            {
                int j = mid - 1;
                while(j >= 0 && nums[j] == target)    j--;
                result.push_back(j + 1);
                j = mid + 1;
                while(j < size && nums[j] == target)    j++;
                result.push_back(j - 1);
                return result;
            }
            else if(nums[mid] > target)
            {
                right = mid - 1;
            }
            else 
            {
                left = mid + 1;
            }
        }
        return vector<int>({-1, -1});
    }
};
```

### Search a 2D Matrix
Write an efficient algorithm that searches for a value in an m x n matrix. This matrix has the following properties:
Integers in each row are sorted from left to right.
The first integer of each row is greater than the last integer of the previous row.
For example, Consider the following matrix:
[
  [1,   3,  5,  7],
  [10, 11, 16, 20],
  [23, 30, 34, 50]
]
Given target = 3, return true.[LeetCode](https://leetcode.com/problems/search-a-2d-matrix/description/)
```cpp
/**
二分法查找，首先利用二分法定位目标值位于哪一行，然后在该行再次利用二分法查找
*/
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        if(matrix.empty() || matrix[0].empty())
            return false;
        int m = matrix.size(), n = matrix[0].size();
        int left = 0, right = m - 1, mid = 0, row = 0;
        while(left <= right)
        {
            mid = (left + right) / 2;
            //如果目标值在上一行的尾值和当前行的首值之间，则无法确定位于哪一行
            if(matrix[mid][0] <= target && matrix[mid][n - 1] >= target)
            {
                row = mid;
                break;
            }
            else if(matrix[mid][0] > target)
            {
                right = mid - 1;
            }
            else
            {
                left = mid + 1;
            }    
        }
        //目标值不在任一行之中，直接返回false；不要判断条件也能返回正确结果，但该条件更符合逻辑
        if(left > right)    return false;   
        left = 0, right = n - 1;
        while(left <= right)
        {
            mid = (left + right) / 2;
            if(matrix[row][mid] == target)
                return true;
            else if(matrix[row][mid] < target)
                left = mid + 1;
            else
                right = mid - 1;
        }
        return false;
    }
};

/**
矩阵和数组通过坐标对应，直接进行二分法查找
*/
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        if(matrix.empty() || matrix[0].empty())
            return false;
        int m = matrix.size(), n = matrix[0].size();
        int left = 0, right = m * n - 1, mid = 0;
        while(left <= right)
        {
            mid = (left + right) / 2;
            if(matrix[mid / n][mid % n] == target)
                return true;
            else if(matrix[mid / n][mid % n] < target)
                left = mid + 1;
            else
                right = mid - 1;
        }
        return false;
    }
};
```

### Median of Two Sorted Arrays
There are two sorted arrays nums1 and nums2 of size m and n respectively. Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)). [LeetCode](https://leetcode.com/problems/median-of-two-sorted-arrays/description/)
```cpp
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int length = nums1.size() + nums2.size();
        vector<int> nums;
        int i = 0, j = 0, k = 0;
        double median = 0.0;
        if(nums1.empty() && nums2.empty())
            return 0.0;
        else if(nums1.empty())
            nums = nums2;
        else if(nums2.empty())
            nums = nums1;
        else
        {
            while(i < nums1.size() && j < nums2.size())
            {
                if(nums1[i] > nums2[j])
                {
                    nums.push_back(nums2[j]);
                    j++;
                }
                else
                {
                    nums.push_back(nums1[i]);
                    i++;
                }
            }
            while(i < nums1.size())
                nums.push_back(nums1[i++]);
            while(j < nums2.size())
                nums.push_back(nums2[j++]);
        }
        if(length % 2 == 1)
            median = nums[length / 2];
        else
            median = (nums[length / 2 - 1] + nums[length / 2]) / 2.0;
        return median;
    }
};

/*利用STL函数merge()合并两个已排序的向量，也可将两个向量首尾拼接，然后调用sort()函数*/
double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) 
{
    int length = nums1.size() + nums2.size();
    vector<int> nums(length);
    double median = 0.0;       
    merge(nums1.begin(), nums1.end(), nums2.begin(), nums2.end(), nums.begin());
    if(length % 2 == 1)
        median = nums[length / 2];
    else
        median = (nums[length / 2 - 1] + nums[length / 2]) / 2.0;
    return median;
}

/**
log(min(m, n))解法 
https://leetcode.com/problems/median-of-two-sorted-arrays/discuss/2481/Share-my-O(log(min(mn))-solution-with-explanation
http://windliang.cc/2018/07/18/leetCode-4-Median-of-Two-Sorted-Arrays/
*/
double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) 
{
    int m = nums1.size(), n = nums2.size();
    if(m > n)
        return findMedianSortedArrays(nums2, nums1);
    int iMin = 0, iMax = m;
    while(iMin <= iMax)
    {
        int i = (iMin + iMax) / 2;
        int j = (m + n + 1) / 2 - i;
        if(j != 0 && i != m && nums1[i] < nums2[j - 1])
            iMin = i + 1;
        else if(j != n && i != 0 && nums1[i - 1] > nums2[j])
            iMax = i - 1;
        else
        {
            int maxLeft = 0;
            if(i == 0)  maxLeft = nums2[j - 1];
            else if(j == 0) maxLeft = nums1[i - 1];
            else    maxLeft = max(nums1[i - 1], nums2[j - 1]);
            if((m + n) % 2 == 1)    return maxLeft;
            
            int minRight = 0;
            if(i == m)  minRight = nums2[j];
            else if(j == n) minRight = nums1[i];
            else    minRight = min(nums1[i], nums2[j]);
            return (maxLeft + minRight) / 2.0;
        }
    }
    return 0.0;
}

//O(log(m + n))解法 
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int m = nums1.size(), n = nums2.size();
        int left = (n + m + 1) / 2, right = (n + m + 2) / 2;
        return (getKth(nums1, 0, m - 1, nums2, 0, n - 1, left) + getKth(nums1, 0, m - 1, nums2, 0, n - 1, right)) * 0.5;
    }

    int getKth(vector<int>& nums1, int start1, int end1, vector<int>& nums2, int start2, int end2, int k)
    {
        int len1 = end1 - start1 + 1;
        int len2 = end2 - start2 + 1;
        if(len1 > len2)
            return getKth(nums2, start2, end2, nums1, start1, end1, k);
        if(len1 == 0)
            return nums2[start2 + k - 1];
        if(k == 1)
            return min(nums1[start1], nums2[start2]);

        int i = start1 + min(len1, k / 2) - 1;
        int j = start2 + min(len2, k / 2) - 1;
        if(nums1[i] > nums2[j])
            return getKth(nums1, start1, end1, nums2, j + 1, end2, k - (j - start2 + 1));
        else
            return getKth(nums1, i + 1, end1, nums2, start2, end2, k - (i - start1 + 1));
    }
};

```

