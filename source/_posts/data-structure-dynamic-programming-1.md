---
title: 动态规划(Dynamic Programming)问题集锦（一）
date: 2018-02-04
categories: Data Structure
tags: [code, dp]
---
数据结构与算法中动态规划问题的总结归纳。
<!--more-->

###  Climbing Stairs
You are climbing a stair case. It takes n steps to reach to the top. Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top? Note: Given n will be a positive integer.
```cpp
/**
1. n = 1: steps = 1
2. n = 2: steps = 2
3. n > 2: steps[i] = steps[i - 1] + steps[i - 2]
对于第 i 个台阶，可以先走到 i - 1 个台阶，再走一步；或者先走到第 i - 2 个台阶，再走一个步（跨两个台阶），
而且两种走法没有重叠，因为最后一步不一样，所以存在以上迭代关系。本质上就是个 fibonacci 数列
*/

class Solution {
public:
    int climbStairs(int n) {
        vector<int> path(n);
        if(n == 1)
            return 1;
        if(n == 2)
            return 2;
        path[0] = 1;
        path[1] = 2;
        for(int i = 2; i < n; i++)
        {
            path[i] = path[i - 1] + path[i - 2];
        }
        return path[n - 1];
    }
};

/*******上述方法的简化版本，同样是迭代，而无需 n 个存储空间*******/
class Solution {
public:
    int climbStairs(int n) {
        int a = 1, b = 1;
        while (n--)
        {   
            b = b + a;
            a = b - a;
        }
        return a;
    }
};
```

### Word Break
题目描述：[LeetCode](https://leetcode.com/problems/word-break/description/)
Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, 
determine if s can be segmented into a space-separated sequence of one or more dictionary 
words. You may assume the dictionary does not contain duplicate words.
For example, given s = "leetcode", dict = ["leet", "code"].
Return true because "leetcode" can be segmented as "leet code".
分析：DP解法：定义一个数组dp[]，dp[i]为true表示一个有效的单词或单词序列和字符串s的前i个字符匹配
```cpp
bool wordBreak(string s, vector<string>& wordDict) 
{
    unordered_set<string> wordSet;
    for(auto word : wordDict)
        wordSet.insert(word);   //将所有元素移至hashset中，这样在一轮搜索过程中时间复杂度为O(1)
    vector<bool> dp(s.length() + 1, false);
    dp[0] = true;
    for(int i = 1; i <= s.length(); i++)
    {
        for(int j = i - 1; j >= 0; j--) //反向遍历可能会更快
        {
            if(dp[j])   //dp[j]为true且子串s.substr(j, i-j)在word集合中，则dp[i]为true
            {
                string str = s.substr(j, i - j);
                if(wordSet.find(str) != wordSet.end())
                {
                    dp[i] = true;
                    break;
                }
            }
        }
    }
    return dp[s.length()];
}
```

### Palindrome Partitioning II
题目描述：[LeetCode](https://leetcode.com/problems/palindrome-partitioning-ii/description/)
Given a string s, partition s such that every substring of the partition is a palindrome.
Return the minimum cuts needed for a palindrome partitioning of s.
For example, given s = "aab",
Return 1 since the palindrome partitioning ["aa","b"] could be produced using 1 cut.
```cpp
//法一
int minCut(string s) 
{
    int n = s.size();
    vector<int> cut(n + 1, 0);
    for(int i = 0; i < n + 1; i++)
        cut[i] = i - 1;     //初始化dp值
    for(int i = 0; i < n; i++)
    {
        for(int j = 0; i - j >= 0 && i + j < n && s[i - j] == s[i + j]; j++)
            cut[i + j + 1] = min(cut[i + j + 1], cut[i - j] + 1);   //奇数长度的回文序列
        for(int j = 1; i - j + 1 >= 0 && i + j < n && s[i - j + 1] == s[i + j]; j++)
            cut[i + j + 1] = min(cut[i + j + 1], cut[i - j + 1] + 1);   //偶数长度的回文序列
    }
    return cut[n];
}

//法二：dp[i + 1]记录长度为i的序列的最小分割次数，isPal[i][j]表示s[i...j]是否为回文序列
int minCut(string s) 
{
    int n = s.size();
    vector<int> dp(n + 1, 0);
    for(int i = 0; i < n + 1; i++)
        dp[i] = i - 1;
    vector<vector<bool>> isPal(n, vector<bool>(n, false));
    for(int right = 0; right < n; right++)
    {
        for(int left = 0; left <= right; left++)
        {
            if(s[left] == s[right] && (right - left < 2 || isPal[left + 1][right - 1]))
            {
                isPal[left][right] = true;
                if(left == 0)
                    dp[right + 1] = 0;
                else
                    dp[right + 1] = min(dp[right + 1], dp[left] + 1);
            }
        }
    }
    return dp[n];
}
```