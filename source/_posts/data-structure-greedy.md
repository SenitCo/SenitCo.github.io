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

### Jump Game
[Description](https://leetcode.com/problems/jump-game/description/): Given an array of non-negative integers, you are initially positioned at the first index of the array. Each element in the array represents your maximum jump length at that position. Determine if you are able to reach the last index.

For example:
A = [2,3,1,1,4], return true.
A = [3,2,1,0,4], return false.

```cpp
/**
1. 能跳到位置i的条件：i<=maxIndex。
2. 一旦跳到i，则maxIndex = max(maxIndex, i+nums[i])。
3. 能跳到最后一个位置n-1的条件是：maxIndex >= n-1
*/
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int maxIndex = 0, size = nums.size();
        for(int i = 0; i < size; i++)
        {
            if(i > maxIndex || maxIndex >= size - 1)
                break;
            maxIndex = max(maxIndex, nums[i] + i);
        }
        return maxIndex >= size - 1; 
    }
};

class Solution {
public:
    bool canJump(vector<int>& nums) {
        int i = 0, n = nums.size();
        for (int reach = 0; i < n && i <= reach; ++i)
            reach = max(i + nums[i], reach);
        return i == n;
    }
};
```

### Jump Game II
[Description](https://leetcode.com/problems/jump-game-ii/description/): Given an array of non-negative integers, you are initially positioned at the first index of the array. Each element in the array represents your maximum jump length at that position. Your goal is to reach the last index in the minimum number of jumps.
For example: Given array A = [2,3,1,1,4]
The minimum number of jumps to reach the last index is 2. (Jump 1 step from index 0 to 1, then 3 steps to the last index.)
Note: You can assume that you can always reach the last index.

```cpp
/**
currentMax表示当前能够到达的最大长度，nextMax表示下一步可以到达的最大长度，在currentMax范围内遍历，
确定下一步的最长范围nextMax，超过currentMax则跳数递增，并让 currentMax = nextMax，继续下一轮遍历，
时间复杂度为O(N)
*/
class Solution {
public:
    int jump(vector<int>& nums) {
        if(nums.size() < 2)
            return 0;
        int currentMax = 0, nextMax = 0, i = 0, step = 0;
        while(currentMax >= i)
        {
            step++;
            for(; i <= currentMax; i++)
            {
                nextMax = max(nextMax, i + nums[i]);
                if(nextMax >= nums.size() - 1)
                    return step;
            }
            currentMax = nextMax;
        }
    }
};


class Solution {
public:
    int jump(vector<int>& nums) {
        int cnt = 0, currentMax = 0, nextMax = 0;
        for(int i = 0; i < nums.size(); i++)
        {
            if(i > currentMax)
            {
                currentMax = nextMax;
                cnt++;
            }
            nextMax = max(nextMax, nums[i] + i);
        }
        return cnt;
    }
};
```

### Malloc Candy
题目描述：[LeetCode](https://leetcode.com/problems/candy/description/)
There are N children standing in a line. Each child is assigned a rating value.
You are giving candies to these children subjected to the following requirements:
Each child must have at least one candy.
Children with a higher rating get more candies than their neighbors.
What is the minimum candies you must give?
分析：分别前向、后向遍历一次，更新每个人应该分配的最小数量
```cpp
int mallocCandy(vector<int>& ratings) 
{
    int size = ratings.size();
    if(size < 2)
        return size;
    int result = 0;
    vector<int> nums(size, 1);
    for(int i = 1; i < size; i++)
    {
        if(ratings[i] > ratings[i - 1])
            nums[i] = nums[i - 1] + 1;
    }
    for(int i = size - 1; i > 0; i--)
    {
        if(ratings[i - 1] > ratings[i])
            nums[i - 1] = max(nums[i - 1], nums[i] + 1);
        result += nums[i];  //在第二次遍历(反向)时直接累加，避免再循环一次
    }
    result += nums[0];
    return result;
}
```