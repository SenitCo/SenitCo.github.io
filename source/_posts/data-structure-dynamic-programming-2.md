---
title: 动态规划(Dynamic Programming)问题集锦（二）
date: 2018-02-06
categories: Data Structure
tags: [code, dp, buy-and-sell]
---
数据结构与算法中动态规划问题的总结归纳。
<!--more-->

### Maximum Subarray
题目描述：Find the contiguous subarray within an array (containing at least one number) which has the largest sum.
For example, given the array [-2,1,-3,4,-1,2,1,-5,4],the contiguous subarray [4,-1,2,1] has the largest sum = 6.[LeetCode](https://leetcode.com/problems/maximum-subarray/description/)
```cpp 
int maxSubArray(vector<int>& nums) 
{
    int maxSoFar = nums[0], maxEndingHere = nums[0];
    for (int i = 1;i < nums.size(); i++)
    {
        maxEndingHere = max(maxEndingHere + nums[i], nums[i]);
        maxSoFar = max(maxSoFar, maxEndingHere);    
    }
    return maxSoFar;
}
```

### Best Time to Buy and Sell Stock
题目描述：Say you have an array for which the ith element is the price of a given stock on day i.If you were only permitted to complete at most one transaction (ie, buy one and sell one share of the stock), design an algorithm to find the maximum profit.
Example 1: Input: [7, 1, 5, 3, 6, 4], Output: 5
max. difference = 6-1 = 5 (not 7-1 = 6, as selling price needs to be larger than buying price)
Example 2: Input: [7, 6, 4, 3, 1], Output: 0
In this case, no transaction is done, i.e. max profit = 0.[LeetCode](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/)

```cpp
//minVal表示已遍历元素中的最小值，profit表示当前收益的最大值
int maxProfit(vector<int>& prices) 
{
    int minVal = INT_MAX, profit = 0, size = prices.size();
    for(int i = 0; i < size; i++)
    {
        minVal = min(prices[i], minVal);
        profit = max(profit, prices[i] - minVal);
    }
    return profit;
}

/**
转化为最大子数列问题（Kadane算法），从动态规划的角度考虑：
maxEndingHere[i] = max(0, maxEndingHere[i - 1] + prices[i] - prices[i - 1]),
maxEndingHere[i - 1]表示第i-1天售出股票的最大利润，maxEndingHere[i]表示第i天售出股票的最大利润，
因此需要加上前后两天的股票差价，maxSoFor表示到目前为止的最大利益（不一定在当天售出）
*/
int maxProfit(vector<int>& prices) 
{
    int maxEndingHere = 0, maxSoFar = 0, size = prices.size();
    for(int i = 1; i < size; i++)
    {
        maxEndingHere = max(0, maxEndingHere += prices[i] - prices[i - 1]);
        maxSoFar = max(maxEndingHere, maxSoFar);
    }
    return maxSoFar;
}
```

### Best Time to Buy and Sell Stock II
题目描述：Say you have an array for which the ith element is the price of a given stock on day i.Design an algorithm to find the maximum profit. You may complete as many transactions as you like(ie, buy one and sell one share of the stock multiple times). However, you may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).[LeetCode](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/description/)
```cpp
//累加相邻极小值点与极大值点之间的差值
int maxProfit(vector<int>& prices) 
{
    int minVal = INT_MAX, maxPro = 0, sumPro = 0, size = prices.size();
    if(size == 0)   return 0;
    for(int i = 0; i < prices.size(); i++)
    {
        minVal = min(minVal, prices[i]);
        if(maxPro < prices[i] - minVal)
            maxPro = prices[i] - minVal;
        else
        {
            sumPro += maxPro;   //maxPro没有更新，说明前一个点为极值点，累加收益值，并更新最小值和单次最大收益
            minVal = prices[i];
            maxPro = 0;
        }
    }
    if(minVal != prices[size - 1])  //考虑最后一个点为极大值的情况
        sumPro += maxPro;
    return sumPro;
}

//累加递增的股票差值
int maxProfit(vector<int>& prices) 
{
    int profit = 0;
    for(int i = 1; i < prices.size(); i++)
    {
        if(prices[i - 1] < prices[i])
            profit += prices[i] - prices[i - 1];
    }
    return profit;
}
```

### Best Time to Buy and Sell Stock III
题目描述：Say you have an array for which the ith element is the price of a given stock on day i. Design an algorithm to find the maximum profit. You may complete at most two transactions.
Note: You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).
```cpp
/**DP算法：可扩展到最多交易k次
定义local[i][j]为在到达第i天时最多可进行j次交易并且最后一次交易在最后一天卖出的最大利润，此为局部最优。
定义global[i][j]为在到达第i天时最多可进行j次交易的最大利润，此为全局最优。递推式为：
diff = prices[i] - prices[i - 1]
local[i][j] = max(global[i - 1][j - 1] + max(diff, 0), local[i - 1][j] + diff)
global[i][j] = max(local[i][j], global[i - 1][j])
其中局部最优值是比较前一天并少交易一次的全局最优加上大于0的差值，和前一天的局部最优加上差值中取较大值，
而全局最优比较局部最优和前一天的全局最优。
*/
int maxProfit(vector<int>& prices) 
{
    if(prices.empty())  return 0;
    int n = prices.size();
    vector<vector<int>> local(n, vector<int>(3, 0)), global(n, vector<int>(3, 0));
    for(int i = 1; i < n; i++)
    {
        int diff = prices[i] - prices[i - 1];
        for(int j = 1; j < 3; j++)
        {
            local[i][j] = max(global[i - 1][j - 1] + max(diff, 0), local[i - 1][j] + diff); 
            global[i][j] = max(global[i - 1][j], local[i][j]);
        }
    }
    return global[n - 1][2];
}

/**
https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/discuss/39608/A-clean-DP-solution-which-generalizes-to-k-transactions
f[k, i] = max(f[k, i-1], prices[i] - prices[j] + f[k-1, j]) { j in range of [0, i-1] }
*/
int maxProfit(vector<int>& prices) 
{
    if(prices.size() <= 1)
        return 0;
    int K = 2, n = prices.size(), maxProf = 0;
    vector<vector<int>> dp(K + 1, vector<int>(n, 0));
    for(int k = 1; k < K + 1; k++)
    {
        int tempMax = dp[k - 1][0] - prices[0];
        for(int i = 1; i < n; i++)
        {
            dp[k][i] = max(dp[k][i - 1], tempMax + prices[i]);
            tempMax = max(tempMax, dp[k - 1][i] - prices[i]);
            maxProf = max(maxProf, dp[k][i]);
        }
    }
    return maxProf;
}

/**
用一维数组来代替二维数组，可以极大地减少空间，由于覆盖的顺序关系，遍历j需要从2到1，
这样可以取到正确的g[j-1]值，而非已经被覆盖过的值
*/
int maxProfit(vector<int>& prices) 
{
    if(prices.empty())  return 0;
    int n = prices.size();
    int local[3] = { 0 }, global[3] = { 0 };
    for(int i = 1; i < n; i++)
    {
        int diff = prices[i] - prices[i - 1];
        for(int j = 2; j >= 1; j--)
        {
            local[j] = max(global[j - 1] + max(0, diff), local[j] + diff);
            global[j] = max(global[j], local[j]);
        }
    }
    return global[2];
}

/**
lowestBuyPrice1始终为输入数组中的最低价，maxProfit1记录到目前为止最高价与最低价的差价（最高价需在最低价后面），
lowestBuyPrice2为相对最小值，maxProfit2则为最多买卖两次的累积最大收益
*/
int maxProfit(vector<int>& prices) 
{
    int maxProfit1 = 0, maxProfit2 = 0, lowestBuyPrice1 = INT_MAX, lowestBuyPrice2 = INT_MAX;
    for(int p : prices)
    {
        maxProfit2 = max(maxProfit2, p - lowestBuyPrice2);
        lowestBuyPrice2 = min(lowestBuyPrice2, p - maxProfit1);
        maxProfit1 = max(maxProfit1, p - lowestBuyPrice1);
        lowestBuyPrice1 = min(lowestBuyPrice1, p);
    }
    return maxProfit2;
}
```

### Triangle
问题描述：Given a triangle, find the minimum path sum from top to bottom. Each step you may move to adjacent numbers on the row below.
Note: Bonus point if you are able to do this using only O(n) extra space, where n is the total number of rows in the triangle.[LeetCode](https://leetcode.com/problems/triangle/description/)
```cpp
/**
For example, given the following triangle
[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
The minimum path sum from top to bottom is 11 (i.e., 2 + 3 + 5 + 1 = 11).
DP算法：从下往上遍历，对当前的某一行取值，选择其下一行相邻的两个累加路径和中较小的一个，进行求和并将结果作为当前行当前位置的最短路径和。
*/
int minimumTotal(vector<vector<int>>& triangle) 
{
    if(triangle.empty())    return 0;
    int size = triangle.size(), minSum = 0;
    vector<int> pathSum(triangle[size - 1].begin(), triangle[size - 1].end());
    for(int i = size - 2; i >= 0; i--)
    {
        for(int j = 0; j < i + 1; j++)
        {
            pathSum[j] = triangle[i][j] + min(pathSum[j], pathSum[j + 1]);
        }
    }
    return pathSum[0];
}
```
