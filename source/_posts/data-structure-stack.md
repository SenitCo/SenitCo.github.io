---
title: 栈(Stack)相关问题集锦
date: 2018-04-10
categories: Data Structure
tags: [code, stack]
---
数据结构与算法中栈相关问题总结归纳。
<!--more-->

### Valid Parentheses
[Description](https://leetcode.com/problems/valid-parentheses/description/): Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid. The brackets must close in the correct order, "()" and "()[]{}" are all valid but "(]" and "([)]" are not.
```cpp
bool isValid(string s) 
{
    unordered_map<char, char> mp({{']', '['}, {')', '('}, {'}', '{'}});
    stack<char> st;
    for(int i = 0; i < s.length(); i++)
    {
        if(mp.find(s[i]) == mp.end())
            st.push(s[i]);
        else if(!st.empty() && mp[s[i]] == st.top())
            st.pop();
        else
            return false;
    }
    return st.empty() ? true : false;
}

bool isValid(string s) 
{
    typedef pair<char, char> p2Char;
    unordered_map<char, char> parenthese{ p2Char('(', ')'), p2Char('{', '}'), p2Char('[', ']') };
    int len = s.length();
    stack<char> ss;
    for (int i = 0; i < len; i++)
    {
        if (parenthese.find(s[i]) != parenthese.end())
        {
            ss.push(s[i]);
        }
        else 
        {
            if(ss.empty() || parenthese[ss.top()] != s[i])
                return false;
            else
                ss.pop();
        }
    }
    return ss.empty();
}

/*压入栈中的为右边括号*/
bool isValid(string s) 
{
    stack<char> ss;
    for(int i = 0; i < s.length(); i++)
    {
        switch(s[i])
        {
            case '(': ss.push(')'); break;
            case '{': ss.push('}'); break;
            case '[': ss.push(']'); break;
            default:
                if(ss.empty() || ss.top() != s[i])
                    return false;
                else
                    ss.pop();
        }
    }
    return ss.empty();
}
```

### Longest Valid Parentheses
[Description](https://leetcode.com/problems/longest-valid-parentheses/description/): Given a string containing just the characters '(' and ')', find the length of the longest valid (well-formed) parentheses substring. For "(()", the longest valid parentheses substring is "()", which has length = 2. Another example is ")()())", where the longest valid parentheses substring is "()()", which has length = 4.
```cpp
/**
遍历字符串中的元素，如果是'('，则将索引 i 压入栈中；如果是')'，则判断栈是否为空，若不为空切栈顶元素为'('，
pop栈顶元素，否则将索引压入栈中。然后计算栈中相邻元素(字符串索引)之间的距离，取最大值即为最大匹配长度。
*/
class Solution {
public:
    int longestValidParentheses(string s) {
        int len = s.length(), maxLen = 0;
        stack<int> st;
        for(int i = 0; i < len; i++)
        {
            if(s[i] == '(')
                st.push(i);
            else
            {
                if(!st.empty())  
                {
                    if(s[st.top()] == '(')
                        st.pop();
                    else
                        st.push(i);
                }
                else
                    st.push(i);
            }
        }
        if(st.empty())
            return len;
        
        int i = len, j = 0;
        while(!st.empty())
        {
            j = st.top();
            st.pop();
            maxLen = max(maxLen, i - j - 1);
            i = j;
        }
        maxLen = max(maxLen, i);
        return maxLen;
    }
};

/**
简化版本，遍历字符串元素的同时，计算长度，并记录最大值
*/
int longestValidParentheses(string s) 
{
    int len = s.length(), maxLen = 0;
    stack<int> st;
    st.push(-1);
    int topVal = 0;
    for(int i = 0; i < len; i++)
    {
        topVal = st.top();
        if(topVal != -1 && s[i] == ')' && s[topVal] == '(')
        {
            st.pop();
            maxLen = max(maxLen, i - st.top());
        }
        else
            st.push(i);
    }
    return maxLen;
}
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