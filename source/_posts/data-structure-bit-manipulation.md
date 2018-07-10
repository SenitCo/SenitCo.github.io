---
title: 位操作(Bit Manipulation)问题集锦
date: 2018-03-05
categories: Data Structure
tags: [code, bit manipulation]
---

数据结构与算法中位操作(Bit Manipulation)问题总结归纳。
<!--more-->

### Single Number
题目描述：[LeetCode](https://leetcode.com/problems/single-number/description/)
Given an array of integers, every element appears twice except for one. Find that single one.
Note: Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?
分析：最大次数为两次，使用1位即可表示，即采用一个int型变量存储数组的元素和，在累加过程中除以2取余，每一位的变化过程为0->1->0(2/0)，利用异或操作实现
```cpp
int singleNumber(vector<int>& nums) 
{
    int ones = 0;
    for(int i = 0; i < nums.size(); i++)
    {
        ones = ones ^ nums[i];
    }
    return ones;
}
```

### Single Number II
题目描述：[LeetCode](https://leetcode.com/problems/single-number-ii/description/)
Given an array of integers, every element appears three times except for one, which appears exactly once. Find that single one.
Note: Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?
```cpp
//hash表，用到了额外的存储空间
int singleNumber(vector<int>& nums) 
{
    unordered_map<int, int> cnt;
    for(int i = 0; i < nums.size(); i++)
    {
        cnt[nums[i]]++;
    }
    for(auto t : cnt)
    {
        if(t.second == 1)
            return t.first;
    }
    return 0;
}

//位操作，对int型数据（32位）分别进行求和除以3取余，所得数字即为所求
int singleNumber(vector<int>& nums) 
{
    vector<int> digit(32, 0);   //固定数组仍为常量存储空间
    int result = 0;
    for(int i = 0; i < nums.size(); i++)
    {
        for(int j = 0; j < 32; j++)
        {
            digit[j] += (nums[i] >> j) & 1;
        }
    }
    for(int i = 0; i < 32; i++)
    {
        result += (digit[i] % 3) << i;
    }
    return result;
}

/**
位操作：用两个int数据分别存储数组和的低位和高位，最大次数为3，因此使用两位即可表示，
所以采用两个int型数据，在累加过程中需要对3取余，因此每一位的变化过程为00(0)->01(1)
->10(2)->00(3/0)。代码中的表达式可由数字逻辑的真值表求得
See [here](http://blog.csdn.net/yutianzuijin/article/details/50597413)
*/
int singleNumber(vector<int>& nums) 
{
    int low = 0, high = 0;
    for(int i = 0; i < nums.size(); i++)
    {
        low = (low ^ nums[i]) & ~high;
        high = (high ^ nums[i]) & ~low;
    }
    return low;
}
```

### Gray Code
[Description](https://leetcode.com/problems/gray-code/description/): The gray code is a binary numeral system where two successive values differ in only one bit.
> Given a non-negative integer n representing the total number of bits in the code, print the 
sequence of gray code. A gray code sequence must begin with 0.
For example, given n = 2, return [0,1,3,2]. Its gray code sequence is:
00 - 0
01 - 1
11 - 3
10 - 2
For a given n, a gray code sequence is not uniquely defined.
For example, [0,2,3,1] is also a valid gray code sequence according to the above definition.

```cpp
/**
Adding one to a number results in flipping all the bits from the rightmost zero bit to the end, e.g. 
110011 + 1 = 110100. In the general form, i = ...?01...1, i+1 = ...?10...0, ? represents the left bit 
of the rightmost zero bit, the length of tailing one bits of i is the same as the length of tailing 
0 bits of i+1, and the bits from the beginning to the ? are the same.

Then i^(i>>1) = xxx(?^0)10...0, (i+1)^((i+1)>>1) = xxx(?^1)10...0. Since the bits from the beginning 
to the ? are the same, xxx part of both results must be same, it's obvious the tailing parts of 10...0 
must be same, and its length is the same as the length of tailing one bits of i, so there is only one 
bit difference comes from (?^0) and (?^1).

See https://discuss.leetcode.com/topic/8557/an-accepted-three-line-solution-in-java

简单翻译一下：对于i和i+1，i的二进制形式(i = ...?01...1)的最右边的连续为1和数量，和i+1的二进制形式(i+1 = ...?10...0)
的最右边连续为0的数量是一样的，？表示i的二进制形式的最右边的0比特和左边一个比特，同样的，i的起始比特位到？处与i+1的起始比特
位到？处是一致的。
考虑 i^(i>>1) = xxx(?^0)10...0，以及 (i+1)^((i+1)>>1) = xxx(?^1)10...0，由于起始比特位到？处是一致的，因此两个二进
制序列xxx部分是一致的，也就是说上述两个表达式只有 (?^0) 和 (?^1) 对应的比特位不一样
*/
class Solution {
public:
    vector<int> grayCode(int n) {
        vector<int> result;
        unsigned int num = 1 << n;
        for(unsigned int i = 0; i < num; i++)
        {
            result.push_back(i ^ (i >> 1));
        }
        return result;
    }
};

/**
Generate the sequence iteratively. For example, when n=3, we can get the result based on n=2.
00,01,11,10 -> (000,001,011,010 ) (110,111,101,100). The middle two numbers only differ at 
their highest bit, while the rest numbers of part two are exactly symmetric of part one. It 
is easy to see its correctness.
*/
class Solution {
public:
    vector<int> grayCode(int n) {
        vector<int> result;
        result.push_back(0);
        for(int i = 0; i < n; i++)
        {
            int size = result.size();
            for(int j = size - 1; j >= 0; j--)
            {
                result.push_back(result[j] | 1 << i);
            }
        }
        return result;
    }
};

```
