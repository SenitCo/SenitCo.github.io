---
title: 动态规划(Dynamic Programming)问题集锦（一）
date: 2018-02-04
categories: Data Structure
tags: [code, dp]
---
数据结构与算法中动态规划问题的总结归纳。
<!--more-->

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

### Malloc Candy
题目描述：[LeetCode](https://leetcode.com/problems/candy/description/)
There are N children standing in a line. Each child is assigned a rating value.
You are giving candies to these children subjected to the following requirements:
Each child must have at least one candy.
Children with a higher rating get more candies than their neighbors.
What is the minimum candies you must give?
分析：分别前向、后向遍历一次，更新每个人应该分配的最小数量
```cpp
int mallocCandy(vector<int>& ratings) 
{
    int size = ratings.size();
    if(size < 2)
        return size;
    int result = 0;
    vector<int> nums(size, 1);
    for(int i = 1; i < size; i++)
    {
        if(ratings[i] > ratings[i - 1])
            nums[i] = nums[i - 1] + 1;
    }
    for(int i = size - 1; i > 0; i--)
    {
        if(ratings[i - 1] > ratings[i])
            nums[i - 1] = max(nums[i - 1], nums[i] + 1);
        result += nums[i];  //在第二次遍历(反向)时直接累加，避免再循环一次
    }
    result += nums[0];
    return result;
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