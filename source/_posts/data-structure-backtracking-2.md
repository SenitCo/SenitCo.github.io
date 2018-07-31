---
title: 回溯(Backtracking)问题集锦（二）
date: 2018-03-09
categories: Data Structure
tags: [code, recursive, backtracking]
---
数据结构与算法中回溯(Backtracking)问题总结归纳。
<!--more-->

### N-Queens
[Description](https://leetcode.com/problems/n-queens/description/): The n-queens puzzle is the problem of placing n queens on an n×n chessboard such that no two queens attack each other. Given an integer n, return all distinct solutions to the n-queens puzzle. Each solution contains a distinct board configuration of the n-queens' placement, where 'Q' and '.' both indicate a queen and an empty space respectively.

<img src="https://github.com/SenitCo/Algorithm/blob/master/images/51_n-queens.png" />

> For example,
There exist two distinct solutions to the 4-queens puzzle:
[
 [".Q..",  // Solution 1
  "...Q",
  "Q...",
  "..Q."],
 ["..Q.",  // Solution 2
  "Q...",
  "...Q",
  ".Q.."]
]

```cpp
/**
利用数组tmp记录旗子在棋盘中的位置，例如tmp[i]=j表示第i行第j列放置棋子，得到所有可能的布棋组合后，
再逐一生成对应的棋盘（字符串），利用tmp[i]=j表示棋子位置而不是二维数组board[i][j]='Q'这种方式，
可避免在多方向（列、斜对角）搜索确认是否存在冲突情况，而只需要考虑是否满足一定的数学条件
*/
class Solution {
public:
    vector<vector<string>> solveNQueens(int n) {
        vector<vector<string>> result;
        vector<vector<int>> pos;
        vector<int> tmp(n, 0);
        backtrack(pos, tmp, n, 0);
        for(int i = 0; i < pos.size(); i++)
        {
            vector<string> board(n, string(n, '.'));
            for(int j = 0; j < pos[i].size(); j++)
            {
                int index = pos[i][j];
                board[j][index] = 'Q';
            }
            result.push_back(board);
        }
        return result;
    }
private:
    void backtrack(vector<vector<int>>& pos, vector<int>& tmp, int n, int t)
    {
        if(t == n)
            pos.push_back(tmp);
        else
        {
            for(int i = 0; i < n; i++)
            {
                bool flag = false;
                tmp[t] = i;
                for (int k = 0; k < t; k++)
                {
                    // (*)利用这个关系式判断是否冲突，而不需要多方向搜索
                    if (tmp[t] == tmp[k] || abs(t - k) == abs(tmp[t] - tmp[k]))     
                    {
                        flag = true;
                        break;
                    }                    
                }
                if (!flag)  //如果t = 0或者不存在冲突的情况，则继续递归
                    backtrack(pos, tmp, n, t + 1);
            }
        }
    }
};
```

### N-Queens II
[Description](https://leetcode.com/problems/n-queens-ii/description/)： The n-queens puzzle is the problem of placing n queens on an n×n chessboard such that no two queens attack each other. Given an integer n, return the number of distinct solutions to the n-queens puzzle.
```cpp
// 无需得到每一种棋盘布局情况，只需统计有效的组合数
class Solution {
public:
    int totalNQueens(int n) {
        int sum = 0;
        vector<int> tmp(n, 0);
        backtrack(tmp, n, 0, sum);
        return sum;
    }
private:
    void backtrack(vector<int>& tmp, int n, int t, int& sum)
    {
        if(t == n)
            sum++;
        else
        {
            for(int i = 0; i < n; i++)
            {
                tmp[t] = i;
                if (isValid(tmp, t))
                    backtrack(tmp, n, t + 1, sum);
            }
        }
    }
    
    bool isValid(vector<int>& tmp, int t)
    {
        for (int k = 0; k < t; k++)
        {
            if (tmp[t] == tmp[k] || abs(t - k) == abs(tmp[t] - tmp[k]))
            {
               return false;
            }                    
        }
        return true;
    }
};

class Solution {
public:
    int totalNQueens(int n) {
        int cnt = 0;
        vector<int> cols(n, 1);
        vector<int> dia1(2 * n - 1, 1);
        vector<int> dia2(2 * n - 1, 1);
        backtrack(0, cols, dia1, dia2, n, cnt);
        return cnt;
    }
private:
    void backtrack(int row, vector<int> cols, vector<int> dia1, vector<int> dia2, int n, int& cnt)
    {
        if(row == n)
        {
            cnt++;
            return;
        }
        for(int col = 0; col < n; col++)
        {
            int id1 = row + col, id2 = n - 1 + col - row;
            if(cols[col] && dia1[id1] && dia2[id2])
            {
                cols[col] = dia1[id1] = dia2[id2] = 0;
                backtrack(row + 1, cols, dia1, dia2, n, cnt);
                cols[col] = dia1[id1] = dia2[id2] = 1;
            }   
        }
    }
};
```

### Palindrome Partitioning
[Description](https://leetcode.com/problems/palindrome-partitioning/description/): Given a string s, partition s such that every substring of the partition is a palindrome.
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

### Word Break II
[Description](https://leetcode.com/problems/word-break-ii/description/): Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, add spaces in s to construct a sentence where each word is a valid dictionary word. You may assume the dictionary does not contain duplicate words. Return all such possible sentences.
> For example, given s = "catsanddog", dict = ["cat", "cats", "and", "sand", "dog"].
A solution is ["cats and dog", "cat sand dog"].

```cpp
//递归回溯
class Solution {
public:
    vector<string> wordBreak(string s, vector<string>& wordDict) {
        unordered_map<string, vector<string>> mp;
        unordered_set<string> dict;
        for(auto word : wordDict)
            dict.insert(word);
        return wordBreakRecur(s, dict, mp);
    }
private:
    vector<string> wordBreakRecur(string s, unordered_set<string>& dict, unordered_map<string, vector<string>>& mp)
    {
        if(mp.count(s)) return mp[s];
        vector<string> result;
        if(dict.count(s))
            result.push_back(s);
        
        for(int i = 1; i < s.length(); i++)
        {
            string word = s.substr(i);
            if(dict.count(word))
            {
                string remain = s.substr(0, i);
                vector<string> prev = wordBreakRecur(remain, dict, mp);
                combine(word, prev);
                result.insert(result.end(), prev.begin(), prev.end());
            }
        }
        mp[s] = result;
        return result;
    }
    
    void combine(string word, vector<string>& prev)
    {
        for(int i = 0; i < prev.size(); i++)
            prev[i] += " " + word;
    }
};

```

### Word Search
[Description](https://leetcode.com/problems/word-search/description/): Given a 2D board and a word, find if the word exists in the grid. The word can be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once.
> For example,
Given board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]
word = "ABCCED", -> returns true,
word = "SEE", -> returns true,
word = "ABCB", -> returns false.

```cpp
/**
定义一个标志数组visited，如果位置(i, j)被访问过，则令visited[i][j] = 0
*/
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        int m = board.size(), n = board[0].size();
        vector<vector<int>> visited(m, vector<int>(n, 0));
        for (int i = 0; i < board.size(); i++)
        {
            for (int j = 0; j < board[0].size(); j++)
            {   //第一个判断条件可不要
                if (board[i][j] == word[0] && backtrack(board, word, i, j, visited, 0))
                    return true;
            }
        }
        return false;
    }
private:
    bool backtrack(vector<vector<char>>& board, string& word, int i, int j, vector<vector<int>>& visited, int count)
    {
        if (count == word.size())
            return true;
        //将所有判断条件集中处理
        if (i < 0 || j < 0 || i > board.size() - 1 || j > board[i].size() - 1 || visited[i][j] || board[i][j] != word[count])
            return false;
        visited[i][j] = 1;
        if (backtrack(board, word, i - 1, j, visited, count + 1)
            || backtrack(board, word, i, j - 1, visited, count + 1)
            || backtrack(board, word, i + 1, j, visited, count + 1)
            || backtrack(board, word, i, j + 1, visited, count + 1))
            return true;
        visited[i][j] = 0;
        return false;
    }
};

/**
无需额外的存储空间表示元素是否访问过，若访问某元素，直接将其赋值为一个特殊字符，并在回溯时将其复原
*/
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        for (int i = 0; i < board.size(); i++)
        {
            for (int j = 0; j < board[0].size(); j++)
            {
                if (backtrack(board, word, i, j, 0))
                    return true;
            }
        }
        return false;
    }
private:
    bool backtrack(vector<vector<char>>& board, string& word, int i, int j, int count)
    {
        if (count == word.size())
            return true;
        if (i < 0 || j < 0 || i > board.size() - 1 || j > board[i].size() - 1 || board[i][j] != word[count])
            return false;
        board[i][j] = '?';
        if (backtrack(board, word, i - 1, j, count + 1)
            || backtrack(board, word, i, j - 1, count + 1)
            || backtrack(board, word, i + 1, j, count + 1)
            || backtrack(board, word, i, j + 1, count + 1))
            return true;
        board[i][j] = word[count];
        return false;
    }
};
```