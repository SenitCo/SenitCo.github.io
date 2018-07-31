---
title: 数组(Array)和动态规划
date: 2018-03-26
categories: Data Structure
tags: [code, array, dp, stack]
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

### Largest Rectangle in Histogram
[Description](https://leetcode.com/problems/largest-rectangle-in-histogram/description/): Given n non-negative integers representing the histogram's bar height where the width of each bar is 1, find the area of largest rectangle in the histogram. Above is a histogram where width of each bar is 1, given height = [2,1,5,6,2,3]. The largest rectangle is shown in the shaded area, which has area = 10 unit. For example, Given heights = [2,1,5,6,2,3], return 10.

<img src="https://github.com/SenitCo/Algorithm/blob/master/images/84_histogram_area.png" align="center"/>

```cpp
/**
定义一个存储索引的数组(stack)——index，如果原数组中高度依次递增，则将对应索引push到index中，否则从栈顶
依次取出元素计算面积
*/
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int area = 0, maxArea = 0;
        vector<int> index;
        heights.push_back(0);
        for(int i = 0; i < heights.size(); i++)
        {   //index中元素对应高度必定是递增的
            while(index.size() > 0 && heights[index.back()] >= heights[i])
            {
                int h = heights[index.back()];  //h必定是当前取出的高度最小值
                index.pop_back();
                //begin表示计算面积区间的前一个索引，i是区间的后一个索引，整个区间范围为(i-begin-1)
                int begin = index.size() > 0 ? index.back() : -1;   
                area = (i - begin - 1) * h;
                if(area > maxArea)
                    maxArea = area;
            }
            index.push_back(i);
        }
        return maxArea;
    }
};
```

### Maximal Rectangle
[Description](https://leetcode.com/problems/maximal-rectangle/discuss/29054): Given a 2D binary matrix filled with 0's and 1's, find the largest rectangle containing only 1's and return its area.
```cpp
/**
For example, given the following matrix:
1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
Return 6.

See https://leetcode.com/problems/maximal-rectangle/discuss/29054
*/
class Solution {
public:
    int maximalRectangle(vector<vector<char>>& matrix) {
        if(matrix.empty())  return 0;
        const int m = matrix.size();
        const int n = matrix[0].size();
        vector<int> left(n, 0), right(n, n), height(n, 0);
        int maxArea = 0;
        for(int i = 0; i < m; i++)
        {
            int curLeft = 0, curRight = n;
            for(int j = 0; j < n; j++)
            {
                if(matrix[i][j] == '1')
                    height[j]++;
                else
                    height[j] = 0;
            }
            
            for(int j = 0; j < n; j++)
            {
                if(matrix[i][j] == '1')
                    left[j] = max(left[j], curLeft);
                else
                {
                    left[j] = 0;
                    curLeft = j + 1;
                }
            }
            
            for(int j = n - 1; j >= 0; j--)
            {
                if(matrix[i][j] == '1')
                    right[j] = min(right[j], curRight);
                else
                {
                    right[j] = n;
                    curRight = j;
                }
            }
            
            for(int j = 0; j < n; j++)
            {
                maxArea = max(maxArea, (right[j] - left[j]) * height[j]);
            }
            
        }
        return maxArea;
    }
};

/**
采用Largest Rectangle in Histogram中的方法：逐行遍历，计算每一行中元素的高度（纵向连续为'1'的个数），
然后将其视为一个计算直方图中最大矩形面积的问题求解
See https://leetcode.com/problems/maximal-rectangle/discuss/29064
*/
class Solution {
public:
    int maximalRectangle(vector<vector<char>>& matrix) {
        if(matrix.empty())  return 0;
        const int m = matrix.size();
        const int n = matrix[0].size();
        vector<int> height(n + 1, 0);
        int rectWidth = 0, rectHeight = 0, maxArea = 0;
        
        for(int i = 0; i < m; i++)
        {
            stack<int> index;
            for(int j = 0; j < n + 1; j++)
            {
                if(j < n)
                {
                    if(matrix[i][j] == '1')
                        height[j]++;
                    else
                        height[j] = 0;
                }
                
                while(!index.empty() && height[index.top()] >= height[j])   
                {
                    rectHeight = height[index.top()];
                    index.pop();
                    rectWidth = index.empty() ? j : j - index.top() - 1;
                    if(rectWidth * rectHeight > maxArea)
                        maxArea = rectWidth * rectHeight;
                }
                index.push(j);
            }
        }
        
        return maxArea;
    }
};
```

