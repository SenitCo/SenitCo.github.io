---
title: 二分查找（三）
date: 2018-03-15
categories: Data Structure
tags: [code, binary search, math]
---

数据结构中二分查找问题总结归纳。
<!--more-->

### Divide Two Integers
Divide two integers without using multiplication, division and mod operator. If it is overflow, return MAX_INT.[LeetCode](https://leetcode.com/problems/divide-two-integers/description/)
```cpp
/**
In this problem, we are asked to divide two integers. However, we are not allowed to use division, multiplication 
and mod operations. So, what else can we use? Yeah, bit manipulations.
Let's do an example and see how bit manipulations work.
Suppose we want to divide 15 by 3, so 15 is dividend and 3 is divisor. Well, division simply requires us to find 
how many times we can subtract the divisor from the the dividend without making the dividend negative.
Let's get started. We subtract 3 from 15 and we get 12, which is positive. Let's try to subtract more. Well, we 
shift 3 to the left by 1 bit and we get 6. Subtracting 6 from 15 still gives a positive result. Well, we shift 
again and get 12. We subtract 12 from 15 and it is still positive. We shift again, obtaining 24 and we know we 
can at most subtract 12. Well, since 12 is obtained by shifting 3 to left twice, we know it is 4 times of 3. 
How do we obtain this 4? Well, we start from 1 and shift it to left twice at the same time. We add 4 to an answer
(initialized to be 0). In fact, the above process is like 15 = 3 * 4 + 3. We now get part of the quotient (4), with 
a remainder 3.
Then we repeat the above process again. We subtract divisor = 3 from the remaining dividend = 3 and obtain 0. We know
we are done. No shift happens, so we simply add 1 << 0 to the answer.

Now we have the full algorithm to perform division.
According to the problem statement, we need to handle some exceptions, such as overflow.

Well, two cases may cause overflow:
(1) divisor = 0;
(2) dividend = INT_MIN and divisor = -1 (because abs(INT_MIN) = INT_MAX + 1).
Of course, we also need to take the sign into considerations, which is relatively easy.
Putting all these together, we have the following code.
*/

class Solution {
public:
    int divide(int dividend, int divisor) {
        if(!divisor || (dividend == INT_MIN && divisor == -1))
            return INT_MAX;
        int sign = ((dividend < 0) ^ (divisor < 0)) ? -1 : 1;
        long long dvd = labs(dividend);
        long long dvs = labs(divisor);
        long long temp = dvs, multiple = 1;
        int res = 0;
        while(dvd >= dvs)
        {
            temp = dvs;
            multiple = 1;
            while(dvd >= (temp << 1))
            {
                temp = temp << 1;
                multiple = multiple << 1;
            }
            dvd -= temp;
            res += multiple;
        }
        return sign == 1 ? res : -res;
    }
};
```

### Pow(x, n)
Implement pow(x, n).[LeetCode](https://leetcode.com/problems/powx-n/description/)
Example 1:
Input: 2.00000, 10
Output: 1024.00000
Example 2:
Input: 2.10000, 3
Output: 9.26100

```cpp
//递归计算

double myPow(double x, int n) 
{
    long long ln = n;
    if(ln == 0) return 1;
    if(ln < 0)
    {
        ln = -ln;   //如果不定义成长整型对于 n = -2147483648 会越界
        x = 1 / x;
    }
    return ln % 2 == 0 ? myPow(x * x, ln / 2) : x * myPow(x * x, ln / 2);
}

double myPow(double x, int n) 
{
    if(n < 0) return 1 / x * myPow(1 / x, -(n + 1));    //解决最小负整数反号越界的问题
    if(n == 0) return 1;
    if(n == 2) return x * x;
    if(n % 2 == 0) 
        return myPow(myPow(x, n / 2), 2);
    else
        return x * myPow(myPow(x, n / 2), 2);
}

//迭代计算
double myPow(double x, int n) 
{
    long long ln = n;
    if(ln == 0) return 1;
    if(ln < 0) 
    {
        ln = -ln;
        x = 1 / x;
    }
    double ans = 1;
    while(ln > 0)
    {
        if(ln & 1)  //最低位为1
            ans *= x;
        x *= x;
        ln >>= 1;
    }
    return ans;
}
```

### Sqrt(x)
Implement int sqrt(int x). Compute and return the square root of x.
x is guaranteed to be a non-negative integer.
Example 1:
Input: 4
Output: 2
Example 2:
Input: 8
Output: 2
Explanation: The square root of 8 is 2.82842..., and since we want to return an integer, 
the decimal part will be truncated.[LeetCode](https://leetcode.com/problems/sqrtx/description/)
```cpp
/************二分法查找*************/
class Solution {
public:
    int mySqrt(int x) {
        int left = 1, right = x;
        while(left <= right)
        {
            int mid = (left + right) / 2;
            if(mid == x / mid)
                return mid;
            else if(mid < x / mid)
                left = mid + 1;
            else
                right = mid - 1;
        }
        return right;
    }
};

/**********牛顿迭代法求解************/
class Solution {
public:
    int mySqrt(int x) {
        long r = x;
        while (r * r > x)
            r = (r + x / r) / 2;
        return r;
    }
};
```