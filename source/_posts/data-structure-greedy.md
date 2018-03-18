---
title: 贪心(Greedy)问题集锦
date: 2018-03-02
categories: Data Structure
tags: [code, greedy]
---
数据结构与算法中贪心问题的总结归纳。
<!--more-->

### Gas Station
题目描述：[LeetCode](https://leetcode.com/problems/gas-station/description/)
There are N gas stations along a circular route, where the amount of gas at station i is gas[i].You have a car with an unlimited gas tank and it costs cost[i] of gas to travel from station i to its next station (i+1). You begin the journey with an empty tank at one of the gas stations. Return the starting gas station's index if you can travel around the circuit once, otherwise return -1.
Note: The solution is guaranteed to be unique.
分析：（1）如果gas总储存量不小于总消耗量，那么一定存在一条循环路线
（2）如果某一点的累积存储量小于累积消耗量，那么将下一点作为起始点（如果A点不能到达B点，那么A、B之间的任一点都无法到B点，因此直接选取B的下一点作为起始点）
```cpp
int canCompleteCircuit(vector<int>& gas, vector<int>& cost) 
{
    int size = gas.size();
    int sumGas = 0, sumCost = 0, tank = 0, start = 0;
    for(int i = 0; i < size; i++)
    {
        sumGas += gas[i];
        sumCost += cost[i];
        tank += gas[i] - cost[i];
        if(tank < 0)  
        {
            start = i + 1;
            tank = 0;
        }
    }
    if(sumGas < sumCost)
        return -1;
    else
        return start;
}
```