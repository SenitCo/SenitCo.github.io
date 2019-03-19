---
title: 回溯(Backtracking)问题集锦（一）
date: 2018-03-08
categories: Data Structure
tags: [code, recursive, backtracking]
---
数据结构与算法中回溯(Backtracking)问题总结归纳。
<!--more-->

### Permutations
[Description](https://leetcode.com/problems/permutations/description/): Given a collection of distinct numbers, return all possible permutations.
For example, [1,2,3] have the following permutations:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]
```cpp
//回溯法(深度优先遍历DFS)，无重复元素，无需排序
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        //sort(nums.begin(), nums.end());
        vector<vector<int>> result;
        vector<int> permutation;
        backtrack(nums, result, permutation);
        return result;
    }
private:
    void backtrack(vector<int>& nums, vector<vector<int>>& result, vector<int>& permutation)
    {
        if(permutation.size() == nums.size())
        {
            result.push_back(permutation);
            return;
        }
        for(int i = 0; i < nums.size(); i++)
        {
            auto iter = find(permutation.begin(), permutation.end(), nums[i]);  
            if(iter == permutation.end())   //新元素不在已有序列中，则添加进来
            {
                permutation.push_back(nums[i]);
                backtrack(nums, result, permutation);
                permutation.pop_back();
            }
        }
    }
};

//ref: http://xiaohuiliucuriosity.blogspot.com/2014/12/permutations.html
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int> > result;        
        permuteRecursive(nums, 0, result);
        return result;
    }   
    // permute num[begin..end], invariant: num[0..begin-1] have been fixed/permuted
    void permuteRecursive(vector<int> &nums, int begin, vector<vector<int> > &result) {
        if (begin >= nums.size()) {
            result.push_back(nums); // one permutation instance
            return;
        }
        
        for (int i = begin; i < nums.size(); i++) {
            swap(nums[begin], nums[i]);
            permuteRecursive(nums, begin + 1, result);
            swap(nums[begin], nums[i]);     // reset
        }
    }
};

/**
对于nums = { 1, 2, 3 }，首先将第一个元素 1 插入序列中，然后插入第二个元素 2 ，2 可以在 1 的前面，
也可以在 1 的后面，因此得到结果{{1, 2}, {2, 1}}，同样地，第三个元素 3 插入到已有的排列中时，可以
在索引 0、1、2；依次类推，即可得到全排列。
*/
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> result;
        if(nums.size() == 0)
            return result;
        result.push_back({ nums[0] });
        for(int i = 1; i < nums.size(); i++)
        {
            vector<vector<int>> newRes;
            for(int j = 0; j <= i; j++)
            {
                for(auto perm : result)
                {
                    vector<int> newPerm = perm;
                    newPerm.insert(newPerm.begin() + j, nums[i]);
                    newRes.push_back(newPerm);
                }
            }
            result = newRes;
        }
        return result;
    }
};
```

### Permutations II
[Description](https://leetcode.com/problems/permutations-ii/description/): Given a collection of numbers that might contain duplicates, return all possible unique permutations.
For example, [1,1,2] have the following unique permutations:
[
  [1,1,2],
  [1,2,1],
  [2,1,1]
]
```cpp
/**
回溯法，增加一个flag数组标志相应元素是否在已有排列中，并排除重复元素的影响
在判断条件(*)中，flag[i]为true则考虑下一个元素；如果是相邻重复元素，则只进行一次纵深排列，而跳过横向迭代；
也就是说对于连续的重复元素{a1, a2, ... ak}，在同一层循环中，只需考虑第一个元素。
*/
class Solution {
public:
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> result;
        vector<int> permutation;
        vector<bool> flag(nums.size(), false);
        backtrack(nums, result, permutation, flag);
        return result;
    }
private:
    void backtrack(vector<int>& nums, vector<vector<int>>& result, vector<int>& permutation, vector<bool>& flag)
    {
        if(permutation.size() == nums.size())
        {
            result.push_back(permutation);
            return;
        }
        for(int i = 0; i < nums.size(); i++)
        {         
            if(flag[i] || (i > 0 && nums[i] == nums[i - 1] && !flag[i - 1]))    // (*)
                continue;
            flag[i] = true;
            permutation.push_back(nums[i]);
            backtrack(nums, result, permutation, flag);
            permutation.pop_back();
            flag[i] = false;
        }
    }
};


class Solution {
public:
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> result;
        recursion(nums, result, 0);
        return result;
    }
private:
    //注意第一个参数不是引用，后面的swap操作只对本层以及后续递归有影响
    void recursion(vector<int> nums, vector<vector<int>>& result, int i)    
    {
        if(i == nums.size() - 1)    //只需递归到倒数第二个元素
        {
            result.push_back(nums);
            return;
        }
        for(int j = i; j < nums.size(); j++)
        {
            if(j != i && nums[j] == nums[i])
                continue;
            swap(nums[i], nums[j]);
            recursion(nums, result, i + 1);
        }
   }
};
```

### Combinations
[Description](https://leetcode.com/problems/combinations/description/): Given two integers n and k, return all possible combinations of k numbers out of 1 ... n.
For example, If n = 4 and k = 2, a solution is:
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]
```cpp
class Solution {
public:
    vector<vector<int>> combine(int n, int k) {
        vector<vector<int>> result;
        vector<int> combination;
        backtrack(result, combination, n, k, 0, 0);
        return result;
    }
private:
    void backtrack(vector<vector<int>>& result, vector<int>& combination, int n, int k, int begin, int num)
    {
        if(num == k)
        {
            result.push_back(combination);
            return;
        }
        for(int i = begin; i < n; i++)
        {
            combination.push_back(i + 1);
            backtrack(result, combination, n, k, i + 1, num + 1);
            combination.pop_back();
        }
    }   
};

// 简化版本，无需另外定义一个计数变量 num ，并将循环条件换为 i < n - k + 1，因为一个组合中必须有 k 个值
class Solution {
public:
    vector<vector<int>> combine(int n, int k) {
        vector<vector<int>> result;
        vector<int> combination;
        backtrack(result, combination, n, k, 0);
        return result;
    }
private:
    void backtrack(vector<vector<int>>& result, vector<int>& combination, int n, int k, int begin)
    {
        if(k == 0)
        {
            result.push_back(combination);
            return;
        }
        for(int i = begin; i < n - k + 1; i++)
        {
            combination.push_back(i + 1);
            backtrack(result, combination, n, k - 1, i + 1);
            combination.pop_back();
        }
    }  
};

// 组合迭代公式：C(n, k) = C(n - 1, k - 1) + C(n - 1, k)，n被选中和n没被选中两种情况
class Solution {
public:
    vector<vector<int>> combine(int n, int k) {
        vector<vector<int> > res,res1;      
        if(k==n || k==0)
        {
            vector<vector<int> > res2;
            vector<int> curr;
            for(int i=1;i<=k;i++)
                curr.push_back(i);
            res2.push_back(curr);
            return res2;
        }
        
        res = combine(n-1,k-1);
        
        for(auto itr=res.begin();itr!=res.end();itr++)
        {
            itr->push_back(n);
        }
        
        res1 = combine(n-1,k);
        
        res.insert(res.end(),res1.begin(),res1.end());
        return res;
    }   
};

//See https://leetcode.com/problems/combinations/discuss/26992
class Solution {
public:
    vector<vector<int>> combine(int n, int k) {
        vector<vector<int>> result;
        int i = 0;
        vector<int> p(k, 0);
        while (i >= 0) {
            p[i]++;
            if (p[i] > n) --i;
            else if (i == k - 1) result.push_back(p);
            else {
                ++i;
                p[i] = p[i - 1];
            }
        }
        return result;
    }  
};
```

### Combination Sum
[escription](https://leetcode.com/problems/combination-sum/description/): Given a set of candidate numbers (C) (without duplicates) and a target number (T), find all unique combinations 
in C where the candidate numbers sums to T. The same repeated number may be chosen from C unlimited number of times.
Note:
All numbers (including target) will be positive integers.
The solution set must not contain duplicate combinations.
For example, given candidate set [2, 3, 6, 7] and target 7, 
A solution set is: 
[
  [7],
  [2, 2, 3]
]
```cpp
class Solution {
public:
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        vector<vector<int>> result;
        vector<int> combination;
        search(candidates, combination, result, target, 0);
        return result;
    }
private:
    void search(vector<int>& candidates, vector<int>& combination, vector<vector<int>>& result, int target, int begin)
    {
        if(target == 0)
        {
            result.push_back(combination);
            return;
        }
        for(int i = begin; i < candidates.size() && target >= candidates[i]; i++)
        {
            combination.push_back(candidates[i]);
            search(candidates, combination, result, target - candidates[i], i);     //不是 i + 1 ,可重复利用同一元素
            combination.pop_back();
        }
    }
};
```

### Combination Sum II
[Description](https://leetcode.com/problems/combination-sum-ii/description/): Given a collection of candidate numbers (C) and a target number (T), find all unique combinations in C where 
the candidate numbers sums to T. Each number in C may only be used once in the combination.

Note:
All numbers (including target) will be positive integers.
The solution set must not contain duplicate combinations.
For example, given candidate set [10, 1, 2, 7, 6, 1, 5] and target 8, 
A solution set is: 
[
  [1, 7],
  [1, 2, 5],
  [2, 6],
  [1, 1, 6]
]
```cpp
class Solution {
public:
    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        vector<vector<int>> result;
        vector<int> combination;
        search(candidates, combination, result, target, 0);
        return result;
    }
private:
    void search(vector<int>& candidates, vector<int>& combination, vector<vector<int>>& result, int target, int begin)
    {
        if(target == 0)
        {
            result.push_back(combination);
            return;
        }
        for(int i = begin; i < candidates.size() && target >= candidates[i]; i++)
        {
            if(i == begin || candidates[i] != candidates[i - 1])            //跳过重复元素
            {
                combination.push_back(candidates[i]);
                search(candidates, combination, result, target - candidates[i], i + 1);     //每个元素只可用一次
                combination.pop_back();
            }
        }
    }
};
```

### Combination Sum III
[Description](https://leetcode.com/problems/combination-sum-iii/description/): Find all possible combinations of k numbers that add up to a number n, given that only numbers from 1 to 9 can be used and each combination should be a unique set of numbers.
Note: All numbers will be positive integers. The solution set must not contain duplicate combinations.
Example 1: Input: k = 3, n = 7, Output: [[1,2,4]]
Example 2: Input: k = 3, n = 9, Output: [[1,2,6], [1,3,5], [2,3,4]]
```cpp
/***
如果没限定数字的取值范围，可以将maxVal去掉，并将循环判断条件替换为i<=n
*/
class Solution {
public:
    vector<vector<int>> combinationSum3(int k, int n) {
        vector<vector<int>> results;
        vector<int> combination;
        int maxVal = 9;
        backtrack(k, maxVal, n, results, combination, 1);
        return results;
    }
    
    void backtrack(const int& k, const int& maxVal, int n, vector<vector<int>>& results, vector<int>& combination, int begin)
    {
        if(combination.size() == k && n == 0)
        {
            results.push_back(combination);
            return;
        }
        for(int i = begin; i <= maxVal && combination.size() < k; i++)
        {
            combination.push_back(i);
            backtrack(k, maxVal, n - i, results, combination, i + 1);
            combination.pop_back();
        }
    }
};
```

### Subsets
[Description](https://leetcode.com/problems/subsets/description/): Given a set of distinct integers, nums, return all possible subsets (the power set).
Note: The solution set must not contain duplicate subsets.
For example, If nums = [1,2,3], a solution is:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```cpp
/********回溯法*******/
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> result;
        vector<int> subset;
        backtrack(result, subset, nums, 0);
        return result;
    }
private:
    void backtrack(vector<vector<int>>& result, vector<int>& subset, vector<int>& nums, int begin)
    {
        result.push_back(subset);   //无需判断条件，每次递归都是一个子集
        for(int i = begin; i < nums.size(); i++)
        {
            subset.push_back(nums[i]);
            backtrack(result, subset, nums, i + 1);
            subset.pop_back();
        }
    }
};

// 位操作法（Bit manipulation），一共2^n种情况，正好对应n位二进制数的所有0、1组合情况
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        int elemNum = nums.size(), setNum = pow(2, elemNum);
        vector<vector<int>> result(setNum);
        for(int i = 0; i < elemNum; i++)
        {
            for(int j = 0; j < setNum; j++)
            {
                if((j >> i) & 1)
                    result[j].push_back(nums[i]);
            }
        }
        return result;
    }
};

/**
迭代法
This problem can also be solved iteratively. Take [1, 2, 3] in the problem statement as an example. 
The process of generating all the subsets is like:

Initially: [[]]
Adding the first number to all the existed subsets: [[], [1]];
Adding the second number to all the existed subsets: [[], [1], [2], [1, 2]];
Adding the third number to all the existed subsets: [[], [1], [2], [1, 2], [3], [1, 3], [2, 3], [1, 2, 3]].
*/
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> result(1, vector<int>());
        for(int i = 0; i < nums.size(); i++)
        {
            int n = result.size();  //注意size()在下面循环中是变化的，不能直接放到终止条件中
            for(int j = 0; j < n; j++)
            {
                result.push_back(result[j]);
                result.back().push_back(nums[i]);
            }
        }
        return result;
    }
};
```

### Subsets II
[Description](https://leetcode.com/problems/subsets-ii/description/): Given a collection of integers that might contain duplicates, nums, return all possible subsets (the power set).
Note: The solution set must not contain duplicate subsets.
For example,If nums = [1,2,2], a solution is:
[
  [2],
  [1],
  [1,2,2],
  [2,2],
  [1,2],
  []
]
```cpp
/****************回溯法***************/
class Solution {
public:
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> result;
        vector<int> subset;
        backtrack(result, subset, nums, 0);
        return result;
    }
private:
    void backtrack(vector<vector<int>>& result, vector<int>& subset, vector<int>& nums, int begin)
    {
        result.push_back(subset);
        for(int i = begin; i < nums.size(); i++)
        {
            if(i != begin && nums[i - 1] == nums[i])    //跳过重复元素
                continue;
            subset.push_back(nums[i]);
            backtrack(result, subset, nums, i + 1);
            subset.pop_back();
        }
    }
};

/**
迭代法，对于连续k个重复元素，加入子集中的数量可以为[0, 1, ..., k]
*/
class Solution {
public:
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> result(1, vector<int>());
        for(int i = 0; i < nums.size();)
        {
            int count = 0;
            while(i + count < nums.size() && nums[i] == nums[i + count])
                count++;
            int previousN = result.size();
            for(int j = 0; j < previousN; j++)
            {
                vector<int> instance = result[j];
                for(int k = 0; k < count; k++)
                {
                    instance.push_back(nums[i]);
                    result.push_back(instance);
                }
            }
            i += count;
        }
        return result;
    }
};

/**
迭代法，如果存在重复元素，则从上一次的size处开始复制子集，添加新元素，直至这一次的size处
*/
class Solution {
public:
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> result(1, vector<int>());
        int start = 0, size = 0;
        for(int i = 0; i < nums.size(); i++)
        {
            if(i > 0 && nums[i] == nums[i - 1])
                start = size;
            else
                start = 0;
            size = result.size();
            for(int j = start; j < size; j++)
            {
                result.push_back(result[j]);
                result.back().push_back(nums[i]);
            }
        }
        return result;
    }
};
```

