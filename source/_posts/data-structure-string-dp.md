---
title: 字符串(String)与DP问题
date: 2018-03-20
categories: Data Structure
tags: [code, string, recursive, dp]
---
数据结构与算法中字符串问题的总结归纳。
<!--more-->

### Regular Expression Matching
[Description](https://leetcode.com/problems/regular-expression-matching/description/): Implement regular expression matching with support for '.' and '*'.
'.' Matches any single character.
'*' Matches zero or more of the preceding element.
The matching should cover the entire input string (not partial).
The function prototype should be: bool isMatch(const char *s, const char *p)
> Some examples:
isMatch("aa","a") → false
isMatch("aa","aa") → true
isMatch("aaa","aa") → false
isMatch("aa", "a*") → true
isMatch("aa", ".*") → true
isMatch("ab", ".*") → true
isMatch("aab", "c*a*b") → true

```cpp
/*递归解法一
- 若p为空，若s也为空，返回true，反之返回false
- 若p的长度为1，若s长度也为1，且相同或是p为'.'则返回true，反之返回false
- 若p的第二个字符不为*，若此时s为空返回false，否则判断首字符是否匹配，且从各自的第二个字符开始调用递归函数匹配
- 若p的第二个字符为*，若s不为空且字符匹配，调用递归函数匹配s和去掉前两个字符的p，若匹配返回true，否则s去掉首字母
- 返回调用递归函数匹配s和去掉前两个字符的p的结果
@reference:
http://www.cnblogs.com/grandyang/p/4461713.html
*/
class Solution1 {
public:
    bool isMatch(string s, string p) {
        if (p.empty()) return s.empty();
        if (p.size() == 1) {
            return (s.size() == 1 && (s[0] == p[0] || p[0] == '.'));
        }
        if (p[1] != '*') {
            if (s.empty()) return false;
            return (s[0] == p[0] || p[0] == '.') && isMatch(s.substr(1), p.substr(1));
        }
        while (!s.empty() && (s[0] == p[0] || p[0] == '.')) {
            if (isMatch(s, p.substr(2))) return true;
            s = s.substr(1);
        }
        return isMatch(s, p.substr(2));
    }
};

/*递归解法二
- 先判断p是否为空，若为空则根据s的为空的情况返回结果。
- 当p的第二个字符为*号时，由于*号前面的字符的个数可以任意，可以为0，
    - 那么先用递归来调用为0的情况，就是直接把这两个字符去掉再比较
    - 或者当s不为空，且第一个字符和p的第一个字符相同时，再对去掉首字符的s和p调用递归，注意p不能去掉首字符，
        因为*号前面的字符可以有无限个；
- 如果第二个字符不为*号，那么就比较第一个字符，然后对后面的字符串调用递归
*/
class Solution2 {
public:
    bool isMatch(string s, string p) {
        if (p.empty()) 
            return s.empty();
        if (p.size() > 1 && p[1] == '*') 
        {
            return isMatch(s, p.substr(2)) || (!s.empty() && (s[0] == p[0] || p[0] == '.') && isMatch(s.substr(1), p));
        } 
        else 
        {
            return !s.empty() && (s[0] == p[0] || p[0] == '.') && isMatch(s.substr(1), p.substr(1));
        }
    }
};

/*动态规划法

dp[i][j]表示s[0:i-1]是否能和p[0:j-1]匹配。

递推公式：由于只有p中会含有regular expression，所以以p[j-1]来进行分类。
p[j-1] != '.' && p[j-1] != '*'：dp[i][j] = dp[i-1][j-1] && (s[i-1] == p[j-1])
p[j-1] == '.'：dp[i][j] = dp[i-1][j-1]
p[j-1] == '*':

而关键的难点在于 p[j-1] = '*'。由于星号可以匹配0、1、乃至多个p[j-2]。
1. 匹配0个元素，即消去p[j-2]，此时p[0: j-1] = p[0: j-3]
dp[i][j] = dp[i][j-2]

2. 匹配1个元素，此时p[0: j-1] = p[0: j-2]
dp[i][j] = dp[i][j-1]

3. 匹配多个元素，此时p[0: j-1] = { p[0: j-2], p[j-2], ... , p[j-2] }
dp[i][j] = dp[i-1][j] && (p[j-2]=='.' || s[i-1]==p[j-2])
由于p[j-1]为'*'，而且匹配数目不确定，因此递推式中j值不能减小，而是通过递减i，使得最后匹配的数目为 0 或 1 来进行处理。

@reference:
http://bangbingsyb.blogspot.com/2014/11/leetcode-regular-expression-matching.html
http://xiaohuiliucuriosity.blogspot.com/2014/12/regular-expression-matching.html
https://discuss.leetcode.com/topic/6183/my-concise-recursive-and-dp-solutions-with-full-explanation-in-c
*/
class Solution3 {
public:
    bool isMatch(string s, string p) {
        int m = s.size(), n = p.size();
        vector<vector<bool>> dp(m + 1, vector<bool>(n + 1,false));
        dp[0][0] = true;
        
        for(int i = 0; i <= m; i++) 
        {
            for(int j = 1; j <= n; j++) 
            {
                if(p[j - 1] != '.' && p[j - 1] != '*') 
                {
                    if(i > 0 && s[i - 1] == p[j - 1] && dp[i - 1][j - 1])
                        dp[i][j] = true;
                }
                else if(p[j - 1] == '.') 
                {
                    if(i > 0 && dp[i - 1][j - 1])
                        dp[i][j] = true;
                }
                else if(j > 1)
                {  //'*' cannot be the 1st element
                    if(dp[i][j - 1] || dp[i][j - 2])  // match 0 or 1 preceding element
                        dp[i][j] = true;
                    else if(i > 0 && (p[j - 2] == s[i - 1] || p[j - 2] == '.') && dp[i - 1][j]) // match multiple preceding elements
                        dp[i][j] = true;
                }
            }
        }
        return dp[m][n];
    }
};
```

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


