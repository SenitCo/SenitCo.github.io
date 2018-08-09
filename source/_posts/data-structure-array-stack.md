---
title: 数组(Array)和栈(Stack)
date: 2018-04-10
categories: Data Structure
tags: [code, array, stack]
---
数据结构与算法中数组（Array）和栈问题总结归纳。
<!--more-->

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