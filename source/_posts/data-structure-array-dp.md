---
title: 数组(Array)和动态规划
date: 2018-03-26
categories: Data Structure
tags: [code, array, dp]
---
数据结构与算法中数组（Array）问题总结归纳。
<!--more-->

### Unique Paths
[Description](https://leetcode.com/problems/unique-paths/description/): A robot is located at the top-left corner of a m x n grid (marked 'Start' in the diagram below). The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below). How many possible unique paths are there?
```cpp
/**
DP解法：由于只能向下或向右移动，第一行第一列均为1，且满足关系式dp[i][j] = dp[i - 1][j] + dp[i][j - 1]
*/
int uniquePaths(int m, int n) 
{
    vector<vector<int> > path(m, vector<int> (n, 1));
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            path[i][j] = path[i - 1][j] + path[i][j - 1];
    return path[m - 1][n - 1];
}

/**
公式解法：从(1, 1)到(m, n)一共要走 N = (m-1)+(n-1) = m+n-2 步，其中往下走 k = (m-1) 步，
因此一共有Combination(N, k) = n! / (k!(n - k)!)种组合
*/
class Solution {
public:
    int uniquePaths(int m, int n) {
        int N = n + m - 2;// how much steps we need to do
        int k = m - 1; // number of steps that need to go down
        double res = 1;
        // here we calculate the total possible path number 
        // Combination(N, k) = n! / (k!(n - k)!)
        // reduce the numerator and denominator and get
        // C = ( (n - k + 1) * (n - k + 2) * ... * n ) / k!
        for (int i = 1; i <= k; i++)
            res = res * (N - k + i) / i;
        return (int)res;
    }
};
```

### Unique Paths II
[Description](https://leetcode.com/problems/unique-paths-ii/description/): Follow up for "Unique Paths": Now consider if some obstacles are added to the grids. How many unique paths would there be? An obstacle and empty space is marked as 1 and 0 respectively in the grid.
For example, There is one obstacle in the middle of a 3x3 grid as illustrated below.
[
  [0,0,0],
  [0,1,0],
  [0,0,0]
]
The total number of unique paths is 2. Note: m and n will be at most 100.
```cpp
/**
有障碍物的位置有效路径数为0，第一行第一列需特殊处理，只要某点存在障碍物，则该点及该点之后取值均为0
*/
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int rows = obstacleGrid.size(), cols = obstacleGrid[0].size();
        vector<vector<int>> paths(rows, vector<int>(cols, 0));
        for(int i = 0; i < rows && obstacleGrid[i][0] != 1; i++)
            paths[i][0] = 1;
        for(int j = 0; j < cols && obstacleGrid[0][j] != 1; j++)
            paths[0][j] = 1;
        for(int i = 1; i < rows; i++)
        {
            for(int j = 1; j < cols; j++)
            {
                if(obstacleGrid[i][j] == 1)
                    paths[i][j] = 0;
                else
                    paths[i][j] = paths[i - 1][j] + paths[i][j - 1];
            }
        }
        return paths[rows - 1][cols - 1];
    }
};

/**
每次考虑一行的dp取值，并累加更新
*/
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        if(obstacleGrid.empty())
            return 0;
        int m = obstacleGrid.size(), n = obstacleGrid[0].size();
        vector<int> path(n, 0);
        path[0] = 1;
        for(int i = 0; i < m; i++)
        {
            for(int j = 0; j < n; j++)
            {
                if(obstacleGrid[i][j] == 1)
                    path[j] = 0;
                else if(j > 0)
                    path[j] += path[j - 1];
            }
        }
        return path[n - 1];
    }
};

/**
增加额外的第0行第0列，并使dp[0][1]取值为1，然后判断有无障碍物来更新dp值
*/
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        if(obstacleGrid.empty())
            return 0;
        int m = obstacleGrid.size(), n = obstacleGrid[0].size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        dp[0][1] = 1;
        for(int i = 1 ; i < m + 1; ++i)
            for(int j = 1 ; j < n + 1; ++j)
                if(!obstacleGrid[i - 1][j - 1])
                    dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
        return dp[m][n];
    }
};
```

### Minimum Path Sum
[Description](https://leetcode.com/problems/minimum-path-sum/description/): Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right which minimizes the sum of all numbers along its path. Note: You can only move either down or right at any point in time.
Example 1:
[[1,3,1],
 [1,5,1],
 [4,2,1]]
Given the above grid map, return 7. Because the path 1→3→1→1→1 minimizes the sum.
```cpp
/**
DP方法：由于只能向右或向下移动，最短路径满足关系式path[i][j] = min(path[i - 1][j], path[i][j - 1]) + grid[i - 1][j - 1]
*/
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        if(grid.size() == 0)    return 0;
        int rows = grid.size(), cols = grid[0].size();
        vector<vector<int>> dp(rows + 1, vector<int>(cols + 1, INT_MAX));
        dp[0][1] = 0;
        for(int i = 1; i < rows + 1; i++)
        {
            for(int j = 1; j < cols + 1; j++)
            {
                dp[i][j] = min(dp[i - 1][j] , dp[i][j - 1]) + grid[i - 1][j - 1];
            }
        }
        return dp[rows][cols];
    }
};

/**
上述方法的空间简化版本，只需要O(m)的线性空间
*/
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        if(grid.empty())    return 0;
        int m = grid.size(), n = grid[0].size();
        vector<int> cur(m, grid[0][0]);
        for (int i = 1; i < m; i++)
            cur[i] = cur[i - 1] + grid[i][0]; 
        for (int j = 1; j < n; j++) {
            cur[0] += grid[0][j]; 
            for (int i = 1; i < m; i++)
                cur[i] = min(cur[i - 1], cur[i]) + grid[i][j];
        }
        return cur[m - 1];
    }
};
```



