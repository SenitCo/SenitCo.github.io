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

### Simplify Path
[Description](https://leetcode.com/problems/simplify-path/description/): Given an absolute path for a file (Unix-style), simplify it.
For example, path = "/home/", => "/home", path = "/a/./b/../../c/", => "/c"

Corner Cases:
Did you consider the case where path = "/../"?
In this case, you should return "/".
Another corner case is the path might contain multiple slashes '/' together, such as "/home//foo/".
In this case, you should ignore redundant slashes and return "/home/foo".

```cpp
class Solution {
public:
    string simplifyPath(string path) {
        string result, temp;
        vector<string> vs;
        stringstream ss(path);
        while(getline(ss, temp, '/'))   //getlines可以分割(split)字符串
        {
            if(temp == "" || temp == ".")
                continue;
            if(temp == "..")
            {
                if(!vs.empty())
                    vs.pop_back();
            }
            else
                vs.push_back(temp);
        }
        for(auto s : vs)
            result += '/' + s;
        return result.empty() ? "/" : result;
    }
};
```

### Text Justification
[Description](https://leetcode.com/problems/text-justification/description/): Given an array of words and a length L, format the text such that each line has exactly L characters and is fully (left and right) justified. You should pack your words in a greedy approach; that is, pack as many words as you can in each line. Pad extra spaces ' ' when necessary so that each line has exactly L characters. Extra spaces between words should be distributed as evenly as possible. If the number of spaces on a line do not divide evenly between words, the empty slots on the left will be assigned more spaces than the slots on the right. For the last line of text, it should be left justified and no extra space is inserted between words.

For example, words: ["This", "is", "an", "example", "of", "text", "justification."], L: 16.
Return the formatted lines as:
[
   "This    is    an",
   "example  of text",
   "justification.  "
]
Note: Each word is guaranteed not to exceed L in length.
Corner Cases:
A line other than the last line might contain only one word. What should you do in this case?
In this case, that line should be left-justified.

```cpp
vector<string> fullJustify(vector<string> &words, int L) 
{
    vector<string> res;
    int i = 0, k = 0, l = 0;
    for(i = 0; i < words.size(); i += k) 
    {
        for(k = l = 0; i + k < words.size() && l + words[i + k].size() <= L - k; k++) 
        {
            l += words[i+k].size();
        }
        string tmp = words[i];
        for(int j = 0; j < k - 1; j++) 
        {
            if(i + k >= words.size()) 
                tmp += " ";
            else 
                tmp += string((L - l) / (k - 1) + (j < (L - l) % (k - 1)), ' ');
            tmp += words[i + j + 1];
        }
        tmp += string(L - tmp.size(), ' ');
        res.push_back(tmp);
    }
    return res;
}
```