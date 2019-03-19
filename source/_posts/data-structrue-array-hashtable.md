---
title: 数组(Array)与哈希表
date: 2018-04-12
categories: Data Structure
tags: [code, array, hash table, two pointers]
---
数据结构与算法中数组（Array）问题总结归纳。
<!--more-->

### Valid Sudoku
[Description](https://leetcode.com/problems/valid-sudoku/description/): The Sudoku board could be partially filled, where empty cells are filled with the character '.'. A partially filled sudoku which is valid.
Note: A valid Sudoku board (partially filled) is not necessarily solvable. Only the filled cells need to be validated.
```cpp
class Solution {
public:
    bool isValidSudoku(vector<vector<char>>& board) {
        int m = board.size(), n = board[0].size();
        vector<vector<bool>> row(m, vector<bool>(n, false));
        vector<vector<bool>> col(m, vector<bool>(n, false));
        vector<vector<bool>> cell(m, vector<bool>(n, false));
        
        for(int i = 0; i < m; i++)
        {
            for(int j = 0; j < n; j++)
            {
                if(board[i][j] != '.')
                {
                    int num = board[i][j] - '1';
                    int k = i / 3 * 3 + j / 3;
                    if(row[i][num] || col[num][j] || cell[k][num])
                        return false;
                    row[i][num] = col[num][j] = cell[k][num] = true;
                }
            }
        }
        return true;
    }
};
```

### Two Sum
[Description](https://leetcode.com/problems/two-sum/description/): Given an array of integers, return indices of the two numbers such that they add up to a specific target. You may assume that each input would have exactly one solution, and you may not use the same element twice.
```cpp
vector<int> twoSum(vector<int> &numbers, int target)
{
    //Key is the number and value is its index in the vector.
    unordered_map<int, int> hash;
    vector<int> result;
    for (int i = 0; i < numbers.size(); i++) {
        int numberToFind = target - numbers[i];
        //if numberToFind is found in map, return them
        if (hash.find(numberToFind) != hash.end()) {
            //+1 because indices are NOT zero based
            result.push_back(hash[numberToFind] + 1);
            result.push_back(i + 1);            
            return result;
        }
        //number was not found. Put it in the map.
        hash[numbers[i]] = i;
    }
    return result;
}

vector<int> twoSum(vector<int>& nums, int target) 
{
    vector<int> indices;
    for(int i = 0; i < nums.size(); i++)
    {
        for(int j = i + 1; j < nums.size(); j++)
        {
            if(nums[i] + nums[j] == target)
            {
                indices.push_back(i);
                indices.push_back(j);
                return indices;
            }
        }
    }
}
```

### Three Sum
[Description](https://leetcode.com/problems/3sum/description/): Given an array S of n integers, are there elements a, b, c in S such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero. Note: The solution set must not contain duplicate triplets.
For example, given array S = [-1, 0, 1, 2, -1, -4],
A solution set is:
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```cpp
/*
先取出一个数x，在剩下的数字里面找到两个数字，使得两者和等于(target - x)，那么问题就变成了2Sum；
将数组排序，然后分别从两边向中间遍历，判断是否满足条件。时间复杂度为O(N^2)
*/
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        int size = nums.size();
        vector<vector<int>> result;
        if(nums.size() < 3)
            return result;
        if(nums[size-1] < 0 || nums[0] > 0)
            return result;
        if(nums[0] == 0 && nums[size-1] == 0)
            return {{ 0, 0, 0 }};
        for(int i = 0; i < size; i++)
        {
            for(int j = i + 1, k = size - 1; j < k;)
            {
                if(nums[j] + nums[k] == -nums[i]) 
                {
                    vector<int> tmp{nums[i], nums[j], nums[k]}; //此处是有序数组
                    result.push_back(tmp);  
                    k--;
                    j++;
                }
                else if(nums[j] + nums[k] > -nums[i])
                {
                    k--;
                }
                else
                {
                    j++;
                }
            }
        }
        sort(result.begin(), result.end());
        vector<vector<int>>::iterator iter = unique(result.begin(), result.end());
        result.erase(iter, result.end());
        return result;
    }
};


 vector<vector<int>> threeSum(vector<int>& nums) 
 {
    sort(nums.begin(), nums.end());
    int size = nums.size();
    vector<vector<int>> result;
    if(nums.size() < 3)
        return result;
    if(nums[size-1] < 0 || nums[0] > 0)
        return result;
    if(nums[0] == 0 && nums[size-1] == 0)
        return {{ 0, 0, 0 }};
    int currentValue = nums[0];
    for(int i = 0; i < size; i++)
    {
        if(i > 0 && nums[i] == currentValue)    //连续多个相同的数字，只判断第一个
            continue;
        
        for(int j = i + 1, k = size - 1; j < k;)
        {
            if(nums[j] + nums[k] == -nums[i]) 
            {
                vector<int> tmp{nums[i], nums[j], nums[k]};
                result.push_back(tmp);
                k--;
                j++;
            }
            else if(nums[j] + nums[k] > -nums[i])
            {
                k--;
            }
            else
            {
                j++;
            }
        }
        currentValue = nums[i];
    }
    //sort(result.begin(), result.end());
    vector<vector<int>>::iterator iter = unique(result.begin(), result.end());  //最后得到的元组都是有序的，无需另外排序
    result.erase(iter, result.end());
    return result;
}

```

### 3Sum Closest
Given an array S of n integers, find three integers in S such that the sum is closest to a given number, target. Return the sum of the three integers. You may assume that each input would have exactly one solution. For example, given array S = {-1 2 1 -4}, and target = 1. The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).
```cpp
int threeSumClosest(vector<int>& nums, int target) 
{
    int result, sum, dist = INT_MAX;
    sort(nums.begin(), nums.end());
    for(int i = 0; i < nums.size(); i++)
    {
        for(int j = i + 1, k = nums.size() - 1; j < k;)
        {
            sum = nums[i] + nums[j] + nums[k];
            if(abs(sum - target) < dist)
            {
                dist = abs(sum - target);
                result = sum;
            }
            
            if(sum == target)
                return sum;
            else if(sum > target)
                k--;
            else
                j++;
        }
    }
    return result;
}
```

### 4Sum
[Description](https://leetcode.com/problems/4sum/description/): Given an array S of n integers, are there elements a, b, c, and d in S such that a + b + c + d = target? Find all unique quadruplets in the array which gives the sum of target. Note: The solution set must not contain duplicate quadruplets.
For example, given array S = [1, 0, -1, 0, -2, 2], and target = 0.
A solution set is:
[
  [-1,  0, 0, 1],
  [-2, -1, 1, 2],
  [-2,  0, 0, 2]
]
```cpp
/*hash表解法：将元素两两之和作为主键值key，元素索引(i, j)组成的vetor<pair>作为map value存入hash表中，时间复杂度为(O(N^2))*/
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        vector<vector<int>> result1;
        set<vector<int>> result2;
        typedef pair<int, int> p2Int;
        unordered_map<int, vector<p2Int>> nmap;
        int size = nums.size();
        if(size < 4)
            return result1;
        sort(nums.begin(), nums.end());
        
        for(int i = 0; i < size - 1; i++)
        {
            for(int j = i + 1; j < size; j++)
            {
                nmap[nums[i] + nums[j]].push_back(make_pair(i, j));
            }
        }
        int value = 0;
        for(int i = 0; i < size - 3; i++)
        {
            if(nums[i] + nums[i + 1] + nums[i + 2] + nums[i + 3] > target)
                break;
            for(int j = i + 1; j < size - 2; j++)
            {
                value = target - (nums[i] + nums[j]);
                if(nmap.find(value) != nmap.end())
                {
                    for(auto&& p : nmap[value])
                    {
                        if(p.first > j)
                        {
                            vector<int> tmp = { nums[i], nums[j], nums[p.first], nums[p.second] };
                            result2.insert(tmp);
                        }
                    }
                }                   
            }
        }
        for(auto&& x : result2)
            result1.push_back(x);
        return result1;
    }
};

/*转化为3Sum问题，(1)处的处理保证vector中的元素是有序的，时间复杂度为O(N^3)*/
vector<vector<int>> fourSum(vector<int>& nums, int target) 
{
    vector<vector<int>> result;
    int size = nums.size();
    if(size < 4)
        return result;
    sort(nums.begin(), nums.end());
    for(int i = 0; i < size -3; i++)
    {
        if(i > 0 && nums[i] == nums[i - 1]) continue;       //(1)
        if(nums[i] + nums[i + 1] + nums[i + 2] + nums[i + 3] > target)  break;
        if(nums[i] + nums[size - 1] + nums[size - 2] + nums[size - 3] < target) continue;
        
        for(int j = i + 1; j < size - 2; j++)
        {
            if(j > i + 1 && nums[j] == nums[j - 1]) continue;      //(1)
            if(nums[i] + nums[j] + nums[j + 1] + nums[j + 2] > target)  break;
            if(nums[i] + nums[j] + nums[size - 1] + nums[size - 2] < target)    continue;
            
            for(int left = j + 1, right = size - 1; left < right;)
            {              
                int value = nums[i] + nums[j] + nums[left] + nums[right];
                if(value == target)
                {
                    result.push_back({nums[i], nums[j], nums[left], nums[right]});
                    left++;
                    right--;
                }
                else if(value < target)
                    left++;
                else
                    right--;                   
            }
        }
    }
    vector<vector<int>>::iterator iter = unique(result.begin(), result.end());
    result.erase(iter, result.end());
    return result;
}

/*转化为3Sum问题，(1)处保证vector中的元素是有序的，(2)处保证元素是非重复的*/
vector<vector<int>> fourSum(vector<int>& nums, int target) 
{
    vector<vector<int>> result;
    int size = nums.size();
    if(size < 4)
        return result;
    sort(nums.begin(), nums.end());
    for(int i = 0; i < size -3; i++)
    {
        if(i > 0 && nums[i] == nums[i - 1]) continue;
        if(nums[i] + nums[i + 1] + nums[i + 2] + nums[i + 3] > target)  break;
        if(nums[i] + nums[size - 1] + nums[size - 2] + nums[size - 3] < target) continue;
        
        for(int j = i + 1; j < size - 2; j++)
        {
            if(j > i + 1 && nums[j] == nums[j - 1]) continue;
            if(nums[i] + nums[j] + nums[j + 1] + nums[j + 2] > target)  break;
            if(nums[i] + nums[j] + nums[size - 1] + nums[size - 2] < target)    continue;
            
            for(int left = j + 1, right = size - 1; left < right;)
            {              
                int value = nums[i] + nums[j] + nums[left] + nums[right];
                if(value == target)
                {
                    result.push_back({nums[i], nums[j], nums[left], nums[right]});
                    do { left++; } while(nums[left] == nums[left - 1] && left < right); //(2)跳过重复元素
                    do { right--; } while(nums[right] == nums[right + 1] && left < right);
                }
                else if(value < target)
                    left++;
                else
                    right--;                   
            }
        }
    }
    return result;
}
```