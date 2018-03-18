---
title: 回溯(Backtracking)问题集锦（一）
date: 2018-03-08
categories: Data Structure
tags: [code, recursive, backtracking]
---
数据结构与算法中回溯(Backtracking)问题的总结归纳。
<!--more-->

### Palindrome Partitioning
题目描述：[LeetCode](https://leetcode.com/problems/palindrome-partitioning/description/)
Given a string s, partition s such that every substring of the partition is a palindrome.
Return all possible palindrome partitioning of s.
For example, given s = "aab", return
[
  ["aa","b"],
  ["a","a","b"]
]

```cpp
vector<vector<string>> partition(string s) 
{
    vector<vector<string>> result;
    vector<string> cut;
    backtrack(result, cut, s, 0);
    return result;
}

void backtrack(vector<vector<string>>& result, vector<string>& cut, string& s, int begin)
{
    if(begin == s.length())
    {
        result.push_back(cut);
        return;
    }
    for(int i = begin; i < s.size(); i++)
    {
        if(isPalindrome(s, begin, i))
        {
            cut.push_back(s.substr(begin, i - begin + 1));
            backtrack(result, cut, s, i + 1);
            cut.pop_back();
        }
        
    }
}

bool isPalindrome(string& s, int start, int end)
{
    while(start < end)
    {
        if(s[start++] != s[end--])
            return false;
    }
    return true;
}
```