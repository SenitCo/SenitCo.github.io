---
title: 动态规划(Dynamic Programming)问题集锦（三）
date: 2018-02-09
categories: Data Structure
tags: [code, dp]
---
数据结构与算法中动态规划问题的总结归纳。
<!--more-->

### Distinct Subsequences
题目描述：Given a string S and a string T, count the number of distinct subsequences of S which equals T.A subsequence of a string is a new string which is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (ie, "ACE" is a subsequence of "ABCDE" while "AEC" is not).
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