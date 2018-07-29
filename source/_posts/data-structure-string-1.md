---
title: 字符串(String)问题集锦
date: 2018-03-20
categories: Data Structure
tags: [code, string, recursive, dp]
---
数据结构与算法中字符串问题的总结归纳。
<!--more-->

### Restore IP Addresses
[Description](https://leetcode.com/problems/restore-ip-addresses/description/): Given a string containing only digits, restore it by returning all possible valid IP address combinations. For example: Given "25525511135", return ["255.255.11.135", "255.255.111.35"]. (Order does not matter)

```cpp
/**
迭代组合
*/
class Solution {
public:
    vector<string> restoreIpAddresses(string s) {
        int len = s.length();
        vector<string> result;
        for(int i = 1; i < 4 && i < len - 2; i++)
        {
            for(int j = i + 1; j < i + 4 && j < len - 1; j++)
            {
                for(int k = j + 1; k < j + 4 && k < len; k++)
                {
                    if(len - k > 3) continue;   //如不满足条件，提前终止
                    string s1 = s.substr(0, i);
                    string s2 = s.substr(i, j - i);
                    string s3 = s.substr(j, k - j);
                    string s4 = s.substr(k, len - k);
                    if(isValid(s1) && isValid(s2) && isValid(s3) && isValid(s4))
                        result.push_back(s1 + '.' + s2 + '.' + s3 + '.' + s4);
                }
            }
        }
        return result;
    }
private:
    bool isValid(string s)
    {
        if(std::stoi(s) > 255 || (s[0] == '0' && s.length() > 1))
            return false;
        return true;
    }
};

/**
递归法
*/
class Solution {
public:
    vector<string> restoreIpAddresses(string s) {
        int len = s.length();
        vector<string> result;
        string restored;
        restoreIP(s, result, 0, restored, 0);
        return result;
    }
private:
    void restoreIP(string s, vector<string>& result, int index, string restored, int cnt)
    {
        if (cnt > 4) return;
        if (cnt == 4 && index == s.length())
            result.push_back(restored);
        for (int i = 1; i < 4; i++)
        {
            if (index + i > s.length())  break;
            string str = s.substr(index, i);
            if ((str[0] == '0' && str.length() > 1) || (i == 3 && stoi(str) > 255))
                continue;
            restoreIP(s, result, index + i, restored + str + (cnt == 3 ? "" : "."), cnt + 1);
        }
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
