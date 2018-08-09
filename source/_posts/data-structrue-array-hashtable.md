---
title: 数组(Array)与哈希表
date: 2018-04-12
categories: Data Structure
tags: [code, array, hash table]
---
数据结构与算法中数组（Array）问题总结归纳。
<!--more-->

### Valid Sudoku
[Description](https://leetcode.com/problems/valid-sudoku/description/): The Sudoku board could be partially filled, where empty cells are filled with the character '.'. A partially filled sudoku which is valid.
Note: A valid Sudoku board (partially filled) is not necessarily solvable. Only the filled cells need to be validated.
```cpp
class Solution {
public:
    bool isValidSudoku(vector<vector<char>>& board) {
        int m = board.size(), n = board[0].size();
        vector<vector<bool>> row(m, vector<bool>(n, false));
        vector<vector<bool>> col(m, vector<bool>(n, false));
        vector<vector<bool>> cell(m, vector<bool>(n, false));
        
        for(int i = 0; i < m; i++)
        {
            for(int j = 0; j < n; j++)
            {
                if(board[i][j] != '.')
                {
                    int num = board[i][j] - '1';
                    int k = i / 3 * 3 + j / 3;
                    if(row[i][num] || col[num][j] || cell[k][num])
                        return false;
                    row[i][num] = col[num][j] = cell[k][num] = true;
                }
            }
        }
        return true;
    }
};
```