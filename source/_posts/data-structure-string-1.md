---
title: 字符串(String)问题集锦
date: 2018-03-20
categories: Data Structure
tags: [code, string]
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