---
title: 字符串(String)与DP问题
date: 2018-03-20
categories: Data Structure
tags: [code, string, recursive, dp]
---
数据结构与算法中字符串问题的总结归纳。
<!--more-->

### Wildcard Matching
[Description](https://leetcode.com/problems/wildcard-matching/description/): Implement wildcard pattern matching.

```cpp
/**
Implement wildcard pattern matching with support for '?' and '*'.
'?' Matches any single character.
'*' Matches any sequence of characters (including the empty sequence).

The matching should cover the entire input string (not partial).

The function prototype should be:
bool isMatch(const char *s, const char *p)

Some examples:
isMatch("aa","a") → false
isMatch("aa","aa") → true
isMatch("aaa","aa") → false
isMatch("aa", "*") → true
isMatch("aa", "a*") → true
isMatch("ab", "?*") → true
isMatch("aab", "c*a*b") → false
*/

/**
（1）若两字符串中的字符匹配则索引均前进；
（2）若模式串中出现'*'，则记录'*'号位置 starIdx=pIdx，以及原串的当前位置 match=sIdx；
（3）若当前字符不匹配，且模式串当前字符不为'*'，而上一个字符为'*'，则模式串中待比较字符为'*'号下一个，
    原串字符索引则相对match位置继续前进；
（4）若以上条件均不满足，则两字符串不匹配，返回false；
（5）原串字符遍历结束后，判断模式串是否结束或者剩余字符是否均为'*'
*/
class Solution {
public:
    bool isMatch(string s, string p) {
        int sIdx = 0, pIdx = 0, starIdx = -1, match = 0;
        while(sIdx < s.length())
        {
            if(pIdx < p.length() && (s[sIdx] == p[pIdx] || p[pIdx] == '?'))     // (1)  
            {
                sIdx++;
                pIdx++;
            }
            else if(pIdx < p.length() && p[pIdx] == '*')    // (2)
            {
                starIdx = pIdx;
                pIdx++;
                match = sIdx;
            }
            else if(starIdx != -1)      // (3)
            {
                pIdx = starIdx + 1;
                match++;
                sIdx = match;
            }
            else                        // (4)
                return false;
        }
        while(pIdx < p.length() && p[pIdx] == '*')      // (5)
            pIdx++;
        return pIdx == p.length();
    }
};


/**
DP方法：如果s[0:i)匹配p[0:j)，DP[i][j]=true，否则DP[i][j]=false
(1) DP[i][j] = DP[i - 1][j - 1] && (s[i - 1] == p[j - 1] || p[j - 1] == '?'), if p[j - 1] != '*';
(2) DP[i][j] = DP[i][j - 1] || DP[i - 1][j], if p[j - 1] == '*'.
*/
class Solution {
public:
    bool isMatch(string s, string p) {
        int lenS = s.length(), lenP = p.length();
        vector<vector<bool>> match(lenS + 1, vector<bool>(lenP + 1, false));
        match[0][0] = true;
        for(int j = 0; j < lenP && p[j] == '*'; j++)    //模式串中起始字符为连续的'*'
        {
            match[0][j + 1] = true;
        }
        for(int i = 0; i < lenS; i++)
        {
            for(int j = 0; j < lenP; j++)
            {
                if(s[i] == p[j] || p[j] == '?')             // (1)
                    match[i + 1][j + 1] = match[i][j];
                else if(p[j] == '*')                        // (2)
                    match[i + 1][j + 1] = match[i][j + 1] || match[i + 1][j];
                else
                    match[i + 1][j + 1] = false;
            }
        }
        
        return match[lenS][lenP];
    }
};
```

### Scramble String
[Description](https://leetcode.com/problems/scramble-string/description/)
```cpp
Given a string s1, we may represent it as a binary tree by partitioning it to two non-empty substrings recursively.
Below is one possible representation of s1 = "great":
    great
   /    \
  gr    eat
 / \    /  \
g   r  e   at
           / \
          a   t
To scramble the string, we may choose any non-leaf node and swap its two children.
For example, if we choose the node "gr" and swap its two children, it produces a scrambled string "rgeat".
    rgeat
   /    \
  rg    eat
 / \    /  \
r   g  e   at
           / \
          a   t
We say that "rgeat" is a scrambled string of "great".
Similarly, if we continue to swap the children of nodes "eat" and "at", it produces a scrambled string "rgtae".
    rgtae
   /    \
  rg    tae
 / \    /  \
r   g  ta  e
       / \
      t   a
We say that "rgtae" is a scrambled string of "great".
Given two strings s1 and s2 of the same length, determine if s2 is a scrambled string of s1.
*/

/**
递归解法，并采用预处理方法。首先判断两个字符串是否相等，相等直接返回true；否则判断两个字符串中每个字符出现的次数是否相等，
若不相等，则直接返回false；否则将每个字符串依次划分为两个字串，递归比较子串，子串的比较包括两种：
（1）s1[0...i)和s2[0...i)，s1[i...len)和s2[i...len)两组子串是否相等；
（2）s1[0...i)和s2[len - i ... len)，s1[i...len)和s2[0... len - i)两组子串是否相等
满足以上两个条件之一就返回true，若均不满足则返回false
*/
class Solution {
public:
    bool isScramble(string s1, string s2) {
        if(s1 == s2)
            return true;
        int len = s1.size();
        int count[26] = {0};
        for(int i = 0; i < s1.size(); i++)
        {
            count[s1[i] - 'a']++;
            count[s2[i] - 'a']--;
        }
        for(int i = 0; i < 26; i++)
        {
            if(count[i] != 0)
                return false;
        }
        for(int i = 1; i < len; i++)
        {
            if(isScramble(s1.substr(0, i), s2.substr(0, i)) && isScramble(s1.substr(i), s2.substr(i)))
                return true;
            if(isScramble(s1.substr(0, i), s2.substr(len - i)) && isScramble(s1.substr(i), s2.substr(0, len - i)))
                return true;
        }
        return false;
    }
};

/**
DP算法：dp[len][i][j]表示s1[i... i + len - 1]和s2[j... j + len - 1]是否为Scramble String
*/
class Solution {
public:
    bool isScramble(string s1, string s2) {
        if(s1 == s2)
            return true;
        const int size = s1.size();
        bool dp[size + 1][size][size];
        for(int i = 0; i < size; i++)
        {
            for(int j = 0; j < size; j++)
            {
                dp[1][i][j] = s1[i] == s2[j];
            }
        }
        
        for(int len = 2; len < size + 1; len++)
        {
            for(int i = 0; i <= size - len; i++)   
            {
                for(int j = 0; j <= size - len; j++)
                {
                    dp[len][i][j] = false;
                    for(int k = 1; k < len && !dp[len][i][j]; k++)
                    {
                        dp[len][i][j] = dp[len][i][j] || (dp[k][i][j] && dp[len - k][i + k][j + k]);
                        dp[len][i][j] = dp[len][i][j] || (dp[k][i + len - k][j] && dp[len - k][i][j + k]);
                    }
                }
            }
        }
        return dp[size][0][0];
    }
};

```

### Longest Palindromic Subsequence
[Description](https://leetcode.com/problems/longest-palindromic-subsequence/description/): Given a string s, find the longest palindromic subsequence's length in s. You may assume that the maximum length of s is 1000.
Example 1: Input: "bbbab". Output: 4. One possible longest palindromic subsequence is "bbbb".
Example 2: Input: "cbbd". Output: 2. One possible longest palindromic subsequence is "bb".
```cpp
/***
dp[i][j]: 区间[i, j]的最长回文子序列
s[i] == s[j]: dp[i][j] = dp[i+1][j-1] + 2 
otherwise, dp[i][j] = max(dp[i+1][j], dp[i][j-1])
Initialization: dp[i][i] = 1
*/
int longestPalindromeSubseq(string s) 
{
    int length = s.length();
    vector<vector<int>> dp(length, vector<int>(length, 0));
    for(int i = length - 1; i >= 0; i--)
    {
        dp[i][i] = 1;
        for(int j = i + 1; j < length; j++)
        {
            if(s[i] == s[j])
                dp[i][j] = dp[i+1][j-1] + 2;
            else
                dp[i][j] = max(dp[i][j-1], dp[i+1][j]);
        }
    }
    return dp[0][length-1];
}
```

### Edit Distance
[Description](https://leetcode.com/problems/edit-distance/description/): Given two words word1 and word2, find the minimum number of steps required to convert word1 to word2. 
(each operation is counted as 1 step.)
You have the following 3 operations permitted on a word:
a) Insert a character
b) Delete a character
c) Replace a character

```cpp
/**
DP算法：令dp[i][j]表示将word1[0...i-1]转换为word2[0...j-1]所需的最小转换次数
1. i = 0, dp[0][j] = j; j = 0, dp[i][0] = i
2. word1[i - 1] = word2[j - 1]: dp[i][j] = dp[i - 1][j - 1]
3. word1[i - 1] != word2[j - 1]，则考虑以下三种情况：
(1) word1[i - 1]替换为word2[j - 1]: dp[i][j] = dp[i - 1][j - 1] + 1
(2) 删除word1[i - 1], word1[0...i - 2] = word2[0...j - 1]: dp[i][j] = dp[i - 1][j] + 1
(3) 插入Word2[j - 1], word1[0...i - 1] + word2[j - 1] = word2[0...j - 1]: dp[i][j] = dp[i][j - 1] + 1
*/
int minDistance(string word1, string word2) 
{
    int m = word1.size(), n = word2.size();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
    for(int i = 1; i < m + 1; i++)
        dp[i][0] = i;
    for(int j = 1; j < n + 1; j++)
        dp[0][j] = j;
    for(int i = 1; i < m + 1; i++)
    {
        for(int j = 1; j < n + 1; j++)
        {
            if(word1[i - 1] == word2[j - 1])
                dp[i][j] = dp[i - 1][j - 1];
            else
                dp[i][j] = min(dp[i - 1][j - 1] + 1, min(dp[i - 1][j] + 1, dp[i][j - 1] + 1));
        }
    }
    return dp[m][n];
}
```


