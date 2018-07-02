---
title: 动态规划(Dynamic Programming)问题集锦（三）
date: 2018-02-09
categories: Data Structure
tags: [code, dp]
---
数据结构与算法中动态规划问题的总结归纳。
<!--more-->

### Distinct Subsequences
[Description](https://leetcode.com/problems/distinct-subsequences/description/): Given a string S and a string T, count the number of distinct subsequences of S which equals T.A subsequence of a string is a new string which is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (ie, "ACE" is a subsequence of "ABCDE" while "AEC" is not).
Here is an example: S = "rabbbit", T = "rabbit", Return 3.
```cpp
/**
An example:
S: [acdabefbc] and T: [ab]
first we check with a:
           *  *
      S = [acdabefbc]
mem[1] = [0111222222]

then we check with ab:
               *  * 
      S = [acdabefbc]
mem[1] = [0111222222]
mem[2] = [0000022244]
And the result is 4, as the distinct subsequences are:
      S = [a   b    ]
      S = [a      b ]
      S = [   ab    ]
      S = [   a   b ]

dp[i][j]表示s[0...j-1]中包含t[0...i-1]的序列数，递推式满足：
如果t[i - 1] == s[j - 1]，则dp[i][j] = dp[i][j - 1] + dp[i - 1][j - 1];
否则dp[i][j] = dp[i][j - 1];
*/

int numDistinct(string s, string t) 
{
    int lenS = s.length(), lenT = t.length();
    vector<vector<int>> dp(lenT + 1, vector<int>(lenS + 1, 0));
    for(int j = 0; j < lenS + 1; j++)
        dp[0][j] = 1;
    for(int i = 1; i < lenT + 1; i++)
    {
        for(int j = 1; j < lenS + 1; j++)
        {
            if(t[i - 1] == s[j - 1])
                dp[i][j] = dp[i][j - 1] + dp[i - 1][j - 1];
            else
                dp[i][j] = dp[i][j - 1];
        }
    }
    return dp[lenT][lenS];
}
```

### Interleaving String
[Description](https://leetcode.com/problems/interleaving-string/description/): Given s1, s2, s3, find whether s3 is formed by the interleaving of s1 and s2.
For example, Given: s1 = "aabcc", s2 = "dbbca",
When s3 = "aadbbcbcac", return true.
When s3 = "aadbbbaccc", return false.
```cpp
/** DP方法
See https://leetcode.com/problems/interleaving-string/discuss/31879
DP table represents if s3 is interleaving at (i+j)th position when s1 is at ith position, 
and s2 is at jth position. 0th position means empty string.
So if both s1 and s2 is currently empty, s3 is empty too, and it is considered interleaving. 
If only s1 is empty, then if previous s2 position is interleaving and current s2 position 
char is equal to s3 current position char, it is considered interleaving. similar idea applies 
to when s2 is empty. when both s1 and s2 is not empty, then if we arrive i, j from i-1, j, then 
if i-1,j is already interleaving and i and current s3 position equal, it s interleaving. If we 
arrive i,j from i, j-1, then if i, j-1 is already interleaving and j and current s3 position 
equal. it is interleaving.
*/

bool isInterleave(string s1, string s2, string s3) 
{
    int m = s1.length(), n = s2.length();
    if(m + n != s3.length())    return false;
    vector<vector<bool>> dp(m + 1, vector<bool>(n + 1));
    for(int i = 0; i < m + 1; i++)
    {
        for(int j = 0; j < n + 1; j++)
        {
            if(i == 0 && j == 0)
                dp[i][j] = true;
            else if(i == 0)
                dp[i][j] = (dp[i][j - 1] && s2[j - 1] == s3[i + j - 1]);
            else if(j == 0)
                dp[i][j] = (dp[i - 1][j] && s1[i - 1] == s3[i + j - 1]);
            else
                dp[i][j] = (dp[i][j - 1] && s2[j - 1] == s3[i + j - 1]) || (dp[i - 1][j] && s1[i - 1] == s3[i + j - 1]);
        }
    }
    return dp[m][n];
}
```


### House Robber
[Description](https://leetcode.com/problems/house-robber/description/): You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security system connected and it will automatically contact the police if two adjacent houses were broken into on the same night.

Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight without alerting the police.
```cpp
int rob(vector<int>& nums) 
{
    int n = nums.size();
    int a = 0, b = 0;
    for(int i = 0; i < n; i++)
    {
        if(i % 2 == 0)
            a = max(a + nums[i], b);
        else
            b = max(a, b + nums[i]);
    }
    return max(a, b);
}

//DP解法
int rob(vector<int>& nums) 
{
    int n = nums.size();
    vector<vector<int>> dp(n + 1, vector<int>(2, 0));
    for(int i = 0; i < n; i++)
    {
        dp[i + 1][0] = max(dp[i][0], dp[i][1]);
        dp[i + 1][1] = dp[i][0] + nums[i];
    }
    return max(dp[n][0], dp[n][1]);
}

//迭代法：常量空间
int rob(vector<int>& nums) 
{
    int n = nums.size();
    int prevNo = 0, prevYes = 0;
    for(int i = 0; i < n; i++)
    {
        int temp = prevNo;
        prevNo = max(prevNo, prevYes);
        prevYes = temp + nums[i];
    }
    return max(prevNo, prevYes);
}
```

### House Robber II
[Description](https://leetcode.com/problems/house-robber-ii/description/): After robbing those houses on that street, the thief has found himself a new place for his thievery so that he will not get too much attention. This time, all houses at this place are arranged in a circle. That means the first house is the neighbor of the last one. Meanwhile, the security system for these houses remain the same as for those in the previous street.

Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight without alerting the police.
```cpp
/*
考虑成环的情况：分两种情况考虑
（1）考虑元素区间[0, n - 1)
（2）考虑元素区间[1, n)
*/

int rob(vector<int>& nums) 
{
    int n = nums.size();
    if(n == 1)
        return nums[0];
    return max(helper(nums, 0, n - 1), helper(nums, 1, n));
}

int helper(vector<int>& nums, int left, int right)
{
    int prevNo = 0, prevYes = 0;
    for(int i = left; i < right; i++)
    {
        int temp = prevNo;
        prevNo = max(prevNo, prevYes);
        prevYes = temp + nums[i];
    }
    return max(prevNo, prevYes);
}

int helper(vector<int>& nums, int left, int right)
{
    int pre = 0, cur = 0;
    for (int i = left; i < right; i++) 
    {
        int temp = max(pre + nums[i], cur);
        pre = cur;
        cur = temp;
    }
    return cur;
}
```
