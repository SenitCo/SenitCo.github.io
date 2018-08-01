---
title: 数组(Array)问题集锦（一）
date: 2018-03-22
categories: Data Structure
tags: [code, array]
---
数据结构与算法中数组（Array）问题总结归纳。
<!--more-->

### Next Permutation
[Description](https://leetcode.com/problems/next-permutation/description/): Implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers. If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order). The replacement must be in-place, do not allocate extra memory.
> Here are some examples. Inputs are in the left-hand column and its corresponding outputs are in the right-hand column.
1,2,3 → 1,3,2
3,2,1 → 1,2,3
1,1,5 → 1,5,1

```cpp
/**
实现数组的下一个排列：先找到第一个降序的数字，nums[i+1] > nums[i]，然后从后往前找到第一个大于nums[i]的数，记为nums[j]，
交换nums[i]与nums[j]的值，并翻转 i 后面的元素。
*/
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int size = nums.size();
        if(size < 2)
            return;
        int i = 0, j = 0;
        for(i = size - 1; i > 0; i--)
            if(nums[i] > nums[i - 1])   break;
        if(i == 0)
        {
            reverse(nums.begin(), nums.end());
            return;
        }
        for(j = size - 1; j >= i; j--)
            if(nums[j] > nums[i - 1])   break;
        
        swap(nums[i - 1], nums[j]);
        reverse(nums.begin() + i, nums.end());       
    }
};

void nextPermutation(vector<int>& nums) 
{
    next_permutation(begin(nums), end(nums));
}
```

### Permutation Sequence
[Description](https://leetcode.com/problems/permutation-sequence/description/): The set [1,2,3,…,n] contains a total of n! unique permutations. By listing and labeling all of the permutations in order.
> We get the following sequence (ie, for n = 3):
"123"
"132"
"213"
"231"
"312"
"321"
Given n and k, return the kth permutation sequence.
Note: Given n will be between 1 and 9 inclusive.

```cpp
/**
定义两个数组，一个nums存储原始序列{1,2,...n}，一个fact用于存储对应数的阶乘{1!,2!,...n!}，对于n个数组组成的排列，
共有fact[n]=n!种情况，每个数i为起始的情况共有batch=fact[n]/n=fact[n-1]=(n-1)!，利用(k-1)/batch可求得第一个元素
的索引为idx=(k-1)/batch+1，即result[0]=nums[(k-1)/batch+1]，将其赋值后，在数组nums中移除对应元素，然后更新相关值，
继续下一轮迭代
*/
class Solution {
public:
    string getPermutation(int n, int k) {
        vector<int> nums(n + 1, 0);
        vector<int> fact(n + 1, 1);
        string result;
        for (int i = 1; i < n + 1; i++)
        {
            nums[i] = i;
            fact[i] = fact[i - 1] * i;
        }
        int idx = k - 1;
        for (int i = n; i > 0; i--)
        {
            int batch = fact[i] / n;
            int quo = idx / batch;
            idx = idx % batch;
            result.push_back(nums[quo + 1] + '0');
            nums.erase(nums.begin() + quo + 1);
            n--;
        }
        return result;
    }
};

class Solution {
public:
    string getPermutation(int n, int k) {
        vector<int> nums(n + 1, 0);
        vector<int> fact(n + 1, 1);
        string result;
        for (int i = 1; i < n + 1; i++)
        {
            nums[i] = i;
            fact[i] = fact[i - 1] * i;
        }
        int idx = k - 1, quo = 0;
        for (int i = n; i > 0; i--)
        {
            quo = idx / fact[i - 1];    //将batch直接替换为fact[i-1]
            idx = idx % fact[i - 1];
            result.push_back(nums[quo + 1] + '0');
            nums.erase(nums.begin() + quo + 1);
        }
        return result;
    }
};

```

### Plus One
[Description](https://leetcode.com/problems/plus-one/description/): Given a non-negative integer represented as a non-empty array of digits, plus one to the integer. You may assume the integer do not contain any leading zero, except the number 0 itself. The digits are stored such that the most significant digit is at the head of the list.
```cpp
vector<int> plusOne(vector<int>& digits) 
{
    int size = digits.size();
    int step = 1, temp = 0;
    for(int i = digits.size() - 1; i >= 0; i--)
    {
        temp = digits[i];
        digits[i] = (temp + step) % 10;
        step = (temp + step) / 10;
    }
    if(step > 0)
        digits.insert(digits.begin(), 1);
    return digits;
}
```

### Spiral Matrix
[Description](https://leetcode.com/problems/spiral-matrix/description/): Given a matrix of m x n elements (m rows, n columns), return all elements of the matrix in spiral order.
For example, Given the following matrix:
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]
You should return [1,2,3,6,9,8,7,4,5].
```cpp
/**
向右遍历，向下遍历，向左遍历，向上遍历，循环往复，每次遍历结束，修改边界条件；在向左、向上遍历的时候
需要重新判断一次边界条件，因为在前两次遍历过程中，边界条件已被修改
*/
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        vector<int> result;
        if(matrix.empty())
            return result;
        int rowBegin = 0, rowEnd = matrix.size() - 1, colBegin = 0, colEnd = matrix[0].size() - 1;
        while(rowBegin <= rowEnd && colBegin <= colEnd)
        {
            for(int j = colBegin; j <= colEnd; j++)
                result.push_back(matrix[rowBegin][j]);
            rowBegin++;
            for(int i = rowBegin; i <= rowEnd; i++)
                result.push_back(matrix[i][colEnd]);
            colEnd--;
            if(rowBegin <= rowEnd)      //重新判断条件
            {
                for(int j = colEnd; j >= colBegin; j--)
                    result.push_back(matrix[rowEnd][j]);
            }
            rowEnd--;
            if(colBegin <= colEnd)
            {
                for(int i = rowEnd; i >= rowBegin; i--)
                    result.push_back(matrix[i][colBegin]);
            }
            colBegin++;
        }
        return result;
    }
};

/**
When traversing the matrix in the spiral order, at any time we follow one out of the following four directions: 
RIGHT DOWN LEFT UP. Suppose we are working on a 5 x 3 matrix as such:

0 1 2 3 4 5
6 7 8 9 10
11 12 13 14 15

Imagine a cursor starts off at (0, -1), i.e. the position at ‘0’, then we can achieve the spiral order by doing
the following:

Go right 5 times
Go down 2 times
Go left 4 times
Go up 1 times.
Go right 3 times
Go down 0 times -> quit

Notice that the directions we choose always follow the order ‘right->down->left->up’, and for horizontal movements, 
the number of shifts follows:{5, 4, 3}, and vertical movements follows {2, 1, 0}. Thus, we can make use of a direction 
matrix that records the offset for all directions, then an array of two elements that stores the number of shifts for 
horizontal and vertical movements, respectively. This way, we really just need one for loop instead of four.

Another good thing about this implementation is that: If later we decided to do spiral traversal on a different direction 
(e.g. Counterclockwise), then we only need to change the Direction matrix; the main loop does not need to be touched.

通过定义四组方向参数，以及每次遍历时迭代的次数，将四次遍历统一起来（无需写四组循环）

*/

class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        if(matrix.size() == 0)  return {};
        int m = matrix.size(), n = matrix[0].size();
        vector<int> results(m * n, 0);
        vector<vector<int>> dirs({{0, 1}, {1, 0}, {0, -1}, {-1, 0}});
        vector<int> step({n, m - 1});
        int ir = 0, ic = -1, index = 0, num = m * n;
        for(int i = 0; i < num; )
        {
            for(int j = 0; j < step[index % 2]; j++)
            {
                ir += dirs[index % 4][0];
                ic += dirs[index % 4][1];
                results[i] = matrix[ir][ic];
                i++;
            }
            step[index % 2]--;
            index = (index + 1) % 4;
        }
        return results;
    }
};
```

### Spiral Matrix II
[Description](https://leetcode.com/problems/spiral-matrix-ii/description/): Given an integer n, generate a square matrix filled with elements from 1 to n2 in spiral order.
For example, Given n = 3, You should return the following matrix:
[
 [ 1, 2, 3 ],
 [ 8, 9, 4 ],
 [ 7, 6, 5 ]
]
```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> result(n, vector<int>(n, 0));
        vector<vector<int>> dirs({{0, 1}, {1, 0}, {0, -1}, {-1, 0}});   //表示遍历方向
        int num = n * n + 1;
        vector<int> step({n, n - 1});   //横纵方向遍历的步长（依次递减）
        int ir = 0, ic = -1, idx = 0;
        for(int i = 1; i < num;)
        {
            for(int j = 0; j < step[idx % 2]; j++)
            {
                ir += dirs[idx][0];
                ic += dirs[idx][1];
                result[ir][ic] = i;
                i++;
            }
            step[idx % 2]--;
            idx = (idx + 1) % 4;
        }
        return result;
    }
};

class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> matrix(n, vector<int>(n));
        
        int rowStart = 0, rowEnd = n - 1, colStart = 0, colEnd = n - 1, num = 1;
        
        //通过横纵边界作为循环条件
        while (rowStart <= rowEnd && colStart <= colEnd) {  
            for (int j = colStart; j <= colEnd; j++) {
                matrix[rowStart][j] = num++; 
            }
            rowStart++;
            
            for (int i = rowStart; i <= rowEnd; i++) {
                matrix[i][colEnd] = num++; 
            }
            colEnd--;
            
            if (rowStart <= rowEnd) {
                for (int j = colEnd; j >= colStart; j--) {
                        matrix[rowEnd][j] = num++; 
                }
                rowEnd--;
            }
            
            if (colStart <= colEnd) {
                for (int i = rowEnd; i >= rowStart; i--) {
                        matrix[i][colStart] = num++; 
                }
                colStart ++;
            }
        }
        
        return matrix;
    }
};

class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> matrix(n, vector<int>(n));
        
        int rowStart = 0, rowEnd = n - 1, colStart = 0, colEnd = n - 1, num = 1;
        //元素总数作为循环条件
        while (num < n * n + 1) {
            for (int j = colStart; j <= colEnd; j++) {
                matrix[rowStart][j] = num++; 
            }
            rowStart++;
            
            for (int i = rowStart; i <= rowEnd; i++) {
                matrix[i][colEnd] = num++; 
            }
            colEnd--;
            
            for (int j = colEnd; j >= colStart; j--) {
                    matrix[rowEnd][j] = num++; 
            }
            rowEnd--;

            for (int i = rowEnd; i >= rowStart; i--) {
                    matrix[i][colStart] = num++; 
            }
            colStart++;
        }
        
        return matrix;
    }
};

```

### Rotate Image
[Description](https://leetcode.com/problems/group-anagrams/description/): You are given an n x n 2D matrix representing an image. Rotate the image by 90 degrees (clockwise).
Note: You have to rotate the image in-place, which means you have to modify the input 2D matrix directly. DO NOT allocate another 2D matrix and do the rotation.

>Example 1:
Given input matrix = 
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
],
rotate the input matrix in-place such that it becomes:
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
Example 2:
Given input matrix =
[
  [ 5, 1, 9,11],
  [ 2, 4, 8,10],
  [13, 3, 6, 7],
  [15,14,12,16]
], 
rotate the input matrix in-place such that it becomes:
[
  [15,13, 2, 5],
  [14, 3, 4, 1],
  [12, 6, 8, 9],
  [16, 7,10,11]
]

```cpp
/*****四个对应点坐标依次替换*****/
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        int row = n / 2 + n % 2, col = n / 2;
        int temp = 0;
        for(int i = 0; i < row; i++)
        {
            for(int j = 0; j < col; j++)
            {
                temp = matrix[i][j];
                matrix[i][j] = matrix[n - 1 - j][i];
                matrix[n - 1 - j][i] = matrix[n - 1 - i][n - 1 - j];
                matrix[n - 1 - i][n - 1 - j] = matrix[j][n - 1 - i];
                matrix[j][n - 1 - i] = temp;
            }
        }
    }
};

/*****先上下翻转，然后矩阵转置*****/
/*
 * clockwise rotate
 * first reverse up to down, then swap the symmetry 
 * 1 2 3     7 8 9     7 4 1
 * 4 5 6  => 4 5 6  => 8 5 2
 * 7 8 9     1 2 3     9 6 3
*/
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        reverse(matrix.begin(), matrix.end());
        for (int i = 0; i < matrix.size(); i++) 
        {
            for (int j = i + 1; j < matrix[i].size(); j++)
                swap(matrix[i][j], matrix[j][i]);
        }
    }
};

/*****对于逆时针旋转90°，先左右翻转，然后矩阵转置*****/
/*
 * anticlockwise rotate
 * first reverse left to right, then swap the symmetry
 * 1 2 3     3 2 1     3 6 9
 * 4 5 6  => 6 5 4  => 2 5 8
 * 7 8 9     9 8 7     1 4 7
*/
void anti_rotate(vector<vector<int> > &matrix) 
{
    for (auto vi : matrix) reverse(vi.begin(), vi.end());
    for (int i = 0; i < matrix.size(); ++i) 
    {
        for (int j = i + 1; j < matrix[i].size(); ++j)
            swap(matrix[i][j], matrix[j][i]);
    }
}
```




