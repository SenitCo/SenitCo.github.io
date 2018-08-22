---
title: 回溯(Backtracking)问题集锦（三）
date: 2018-03-11
categories: Data Structure
tags: [code, recursive, backtracking, dp]
---
数据结构与算法中回溯(Backtracking)问题总结归纳。
<!--more-->

### Letter Combinations of a Phone Number
Given a digit string, return all possible letter combinations that the number could represent. A mapping of digit to letters (just like on the telephone buttons) is given below.
Input:Digit string "23", Output: ["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
```cpp
class Solution {
public:
    vector<string> letterCombinations(string digits) {
        vector<string> results;
        string combination;
        if(digits.empty())
            return results;
        vector<string> table = {"abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};
        search(digits, table, results, combination, 0);
        return results;
    }
    
    void search(const string& digits, const vector<string>& table, vector<string>& results, string& combination, int i)
    {
        if(i == digits.length())
        {
            results.push_back(combination);
            return;
        }
        int index = digits[i] - '2';
        for(int j = 0; j < table[index].length(); j++)
        {
            combination.push_back(table[index][j]);
            search(digits, table, results, combination, i + 1);
            combination.pop_back();
        }
    }
};

class Solution {
public:
    vector<string> letterCombinations(string digits) {
        string map[10] = { "0", "1", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz" };
        vector<string> letters;
        if(digits.empty())
            return letters;
        letters.push_back("");
        for(int i = 0; i < digits.length(); i++)
        {
            vector<string> temp;
            string chars = map[digits[i] - '0'];
            for(int j = 0; j < chars.length(); j++)
            {
                for(int k = 0; k < letters.size(); k++)
                {
                    temp.push_back(letters[k] + chars[j]);
                }
            }
            letters = temp;
        }
        return letters;
    }
};
```

### Generate Parentheses
[Description](https://leetcode.com/problems/generate-parentheses/description/): Given n pairs of parentheses, write a function to generate all combinations of well-formed parentheses.
> For example, given n = 3, a solution set is:
[
  "((()))",
  "(()())",
  "(())()",
  "()(())",
  "()()()"
]

```cpp
/*
递归解法：left,right分别表示已插入字符串中左右括号的数量
*/
class Solution1 {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> result;
        backtrack(result, "", 0, 0, n);
        return result;
    }
    void backtrack(vector<string>& vec, string str, int left, int right, int n)
    {
        if(str.length() == n * 2)
        {
            vec.push_back(str);
            return;
        }
        if(left < n)
            backtrack(vec, str + '(', left + 1, right, n);
        if(right < left)
            backtrack(vec, str + ')', left, right + 1, n);
    }
};

/*
递归解法：n,m分别表示可以插入字符串中的左右括号的剩余数量
*/
class Solution2 {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> result;
        addPar(result, "", n, 0);
        return result;
    }
    void addPar(vector<string>& vec, string str, int n, int m)
    {
        if(n == 0 && m == 0)
        {
            vec.push_back(str);
            return;
        }
        if(m > 0)   
            addPar(vec, str + ')', n, m - 1);
        if(n > 0)
            addPar(vec, str + '(', n - 1, m + 1);
    }
};
```

### Sudoku Solver
[Description](https://leetcode.com/problems/sudoku-solver/description/): Write a program to solve a Sudoku puzzle by filling the empty cells. Empty cells are indicated by the character '.'. You may assume that there will be only one unique solution.
> A sudoku solution must satisfy all of the following rules:
Each of the digits 1-9 must occur exactly once in each row.
Each of the digits 1-9 must occur exactly once in each column.
Each of the the digits 1-9 must occur exactly once in each of the 9 3x3 sub-boxes of the grid.

```cpp
/**
对于每个需要填数字的格子带入1到9，每代入一个数字都判定其是否合法，如果合法就继续下一次递归，结束时把数字设回'.'，
判断新加入的数字是否合法时，只需要判定当前数字是否合法，不需要判定这个数组是否为数独数组，因为之前加进的数字都是
合法的，这样可以使程序更加高效一些
*/
class Solution {
public:
    void solveSudoku(vector<vector<char>>& board) {
        if(board.empty() || board.size() != 9 || board[0].size() != 9)
            return;
        search(board, 0, 0);
    }
    
    bool search(vector<vector<char>>& board, int i, int j)
    {
        if(i == 9)  return true;
        if(j >= 9)  return search(board, i + 1, 0);
        if(board[i][j] == '.')
        {
            for(int k = 0; k < 9; k++)
            {
                board[i][j] = (char)(k + '1');
                if(isValid(board, i, j))        //如果新加数字合法，则继续递归      
                {
                    if(search(board, i, j + 1)) 
                        return true;
                }
                board[i][j] = '.';  //新加数字不合法或回溯至此时，将数字设为空
            }
        }
        else
        {
            return search(board, i, j + 1);
        }
        return false;
    }
    
    //判断新加的数字是否合法，只需判断该数字所在行、列以及块，不需遍历整个数盘
    bool isValid(vector<vector<char>>& board, int i, int j)     
    {
        for(int row = 0; row < 9; row++)
        {
            if(i != row && board[i][j] == board[row][j])
                return false;
        }
        
        for(int col = 0; col < 9; col++)
        {
            if(j != col && board[i][j] == board[i][col])
                return false;
        }
        
        for(int row = i / 3 * 3; row < i / 3 * 3 + 3; row++)
        {
            for(int col = j / 3 * 3; col < j / 3 * 3 + 3; col++)
            {
                if((row != i || col != j) && board[i][j] == board[row][col])
                    return false;
            }
        }
        return true;
    }
};

class Solution {
public:
    void solveSudoku(vector<vector<char>>& board) 
    {
        if(board.empty() || board.size() != 9 || board[0].size() != 9)
            return;
        search(board);
    }

    bool search(vector<vector<char> > &board) 
    {
        for (int row = 0; row < 9; ++row) 
        {
            for (int col = 0; col < 9; ++col) 
            {
                if (board[row][col] == '.') 
                {
                    for (int i = 1; i <= 9; ++i)
                    {
                        board[row][col] = '0' + i;

                        if (isValid(board, row, col)) 
                            if (search(board)) 
                                return true;                    
                        board[row][col] = '.';
                    }
                    return false;
                }
            }
        }
        return true;
    }

    bool isValid(vector<vector<char>>& board, int i, int j)     
    {
        for(int row = 0; row < 9; row++)
        {
            if(i != row && board[i][j] == board[row][j])
                return false;
        }
        
        for(int col = 0; col < 9; col++)
        {
            if(j != col && board[i][j] == board[i][col])
                return false;
        }
        
        for(int row = i / 3 * 3; row < i / 3 * 3 + 3; row++)
        {
            for(int col = j / 3 * 3; col < j / 3 * 3 + 3; col++)
            {
                if((row != i || col != j) && board[i][j] == board[row][col])
                    return false;
            }
        }
        return true;
    }

};
```

### Regular Expression Matching
[Description](https://leetcode.com/problems/regular-expression-matching/description/): Implement regular expression matching with support for '.' and '\*'.
'.' Matches any single character.
'\*' Matches zero or more of the preceding element.
The matching should cover the entire input string (not partial).
The function prototype should be: bool isMatch(const char *s, const char *p)
> Some examples:
isMatch("aa","a") → false
isMatch("aa","aa") → true
isMatch("aaa","aa") → false
isMatch("aa", "a\*") → true
isMatch("aa", ".\*") → true
isMatch("ab", ".\*") → true
isMatch("aab", "c\*a\*b") → true

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
[Description](https://leetcode.com/problems/wildcard-matching/description/): Implement wildcard pattern matching with support for '?' and '\*'.
> '?' Matches any single character. 
'\*' Matches any sequence of characters (including the empty sequence).
The matching should cover the entire input string (not partial). 
The function prototype should be: bool isMatch(const char \*s, const char \*p)
Some examples:
isMatch("aa","a") → false
isMatch("aa","aa") → true
isMatch("aaa","aa") → false
isMatch("aa", "\*") → true
isMatch("aa", "a\*") → true
isMatch("ab", "?\*") → true
isMatch("aab", "c\*a\*b") → false

```cpp
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

