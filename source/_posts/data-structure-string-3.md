---
title: 字符串(String)问题集锦
date: 2018-04-08
categories: Data Structure
tags: [code, string]
---
数据结构与算法中字符串(String)问题总结归纳。
<!--more-->

### Multiply Strings
Given two non-negative integers num1 and num2 represented as strings, return the product of num1 and num2.
Note:
The length of both num1 and num2 is < 110.
Both num1 and num2 contains only digits 0-9.
Both num1 and num2 does not contain any leading zero.
You must not use any built-in BigInteger library or convert the inputs to integer directly.
```cpp
/**
see https://github.com/SenitCo/Algorithm/blob/master/images/43_multiply.jpg
*/
class Solution {
public:
    string multiply(string num1, string num2) {
        int len1 = num1.length(), len2 = num2.length();
        vector<int> pos = vector<int>(len1 + len2, 0);
        int p1 = 0, p2 = 0, mul = 0, sum = 0;
        for(int i = len1 - 1; i >= 0; i--)
        {
            for(int j = len2 - 1; j >= 0; j--)
            {
                mul = (num1[i] - '0') * (num2[j] - '0');
                p1 = i + j;
                p2 = i + j + 1;
                sum = mul + pos[p2];    //+pos[p2]是因为经过j--,上一轮的进位由pos[p1]变为pos[p2]
                pos[p1] += sum / 10;    //pos[p1]表示这一轮的进位
                pos[p2] = sum % 10;
            }
        }
        int i = 0;
        string result;
        while(i < pos.size() && pos[i] == 0)    i++;
        for(; i < pos.size(); i++)
        {
            result.append(1, pos[i] + '0');
        }
        if(result.empty())
            return "0";
        return result;
    }
};
```

### Simplify Path
[Description](https://leetcode.com/problems/simplify-path/description/): Given an absolute path for a file (Unix-style), simplify it.
For example, path = "/home/", => "/home", path = "/a/./b/../../c/", => "/c"

Corner Cases:
Did you consider the case where path = "/../"?
In this case, you should return "/".
Another corner case is the path might contain multiple slashes '/' together, such as "/home//foo/".
In this case, you should ignore redundant slashes and return "/home/foo".

```cpp
class Solution {
public:
    string simplifyPath(string path) {
        string result, temp;
        vector<string> vs;
        stringstream ss(path);
        while(getline(ss, temp, '/'))   //getlines可以分割(split)字符串
        {
            if(temp == "" || temp == ".")
                continue;
            if(temp == "..")
            {
                if(!vs.empty())
                    vs.pop_back();
            }
            else
                vs.push_back(temp);
        }
        for(auto s : vs)
            result += '/' + s;
        return result.empty() ? "/" : result;
    }
};
```

### Text Justification
[Description](https://leetcode.com/problems/text-justification/description/): Given an array of words and a length L, format the text such that each line has exactly L characters and is fully (left and right) justified. You should pack your words in a greedy approach; that is, pack as many words as you can in each line. Pad extra spaces ' ' when necessary so that each line has exactly L characters. Extra spaces between words should be distributed as evenly as possible. If the number of spaces on a line do not divide evenly between words, the empty slots on the left will be assigned more spaces than the slots on the right. For the last line of text, it should be left justified and no extra space is inserted between words.

For example, words: ["This", "is", "an", "example", "of", "text", "justification."], L: 16.
Return the formatted lines as:
[
   "This    is    an",
   "example  of text",
   "justification.  "
]
Note: Each word is guaranteed not to exceed L in length.
Corner Cases:
A line other than the last line might contain only one word. What should you do in this case?
In this case, that line should be left-justified.

```cpp
vector<string> fullJustify(vector<string> &words, int L) 
{
    vector<string> res;
    int i = 0, k = 0, l = 0;
    for(i = 0; i < words.size(); i += k) 
    {
        for(k = l = 0; i + k < words.size() && l + words[i + k].size() <= L - k; k++) 
        {
            l += words[i+k].size();
        }
        string tmp = words[i];
        for(int j = 0; j < k - 1; j++) 
        {
            if(i + k >= words.size()) 
                tmp += " ";
            else 
                tmp += string((L - l) / (k - 1) + (j < (L - l) % (k - 1)), ' ');
            tmp += words[i + j + 1];
        }
        tmp += string(L - tmp.size(), ' ');
        res.push_back(tmp);
    }
    return res;
}
```

### Valid Number
[Description](https://leetcode.com/problems/valid-number/description/): Validate if a given string is numeric.
Some examples:
"0" => true
" 0.1 " => true
"abc" => false
"1 a" => false
"2e10" => true
Note: It is intended for the problem statement to be ambiguous. You should gather all requirements up front before implementing one.

```cpp
class Solution {
public:
    bool isNumber(string s) {
        int i = 0;
        for(; s[i] == ' '; i++) {}  // skip the whilespaces

        if(s[i] == '+' || s[i] == '-') i++;     // check the significand, skip the sign if exist

        int n_num = 0, n_point = 0;
        for(; (s[i] >= '0' && s[i] <='9') || s[i]=='.'; i++)
            s[i] == '.' ? n_point++ : n_num++;       
        if(n_point > 1 || n_num < 1) // no more than one point, at least one digit
            return false;

        // check the exponent if exist
        if(s[i] == 'e') {
            i++;
            if(s[i] == '+' || s[i] == '-') i++;     // skip the sign

            n_num = 0;
            for(; s[i] >= '0' && s[i] <='9'; i++, n_num++) {}
            if(n_num < 1)
                return false;
        }

        for(; s[i] == ' '; i++) {}  // skip the trailing whitespaces

        return i == s.length();  // must reach the ending 0 of the string
    }
};
```

### Add Binary
[Description](https://leetcode.com/problems/add-binary/description/): Given two binary strings, return their sum (also a binary string). For example, a = "11", b = "1", Return "100".
```cpp
class Solution {
public:
    string addBinary(string a, string b) {
        int step = 0, sum = 0;
        int i = a.length() - 1, j = b.length() - 1;
        string c;
        for(; i >= 0 && j >= 0; i--, j--)
        {
            sum = (a[i] - '0') + (b[j] - '0') + step;
            c.push_back(sum % 2 + '0');
            step = sum / 2;              
        }
        for(; i >= 0; i--)
        {
            sum = (a[i] - '0') + step;
            c.push_back(sum % 2 + '0');
            step = sum / 2;
        }
        for(; j >= 0; j--)
        {
            sum = (b[j] - '0') + step;
            c.push_back(sum % 2 + '0');
            step = sum / 2;
        }
        if(step > 0)
            c.push_back('1');
        reverse(c.begin(), c.end());
        return c;
            
    }
};

class Solution {
public:
    string addBinary(string a, string b) {
        string s;   
        int c = 0, i = a.size() - 1, j = b.size() - 1;
        while(i >= 0 || j >= 0 || c == 1)
        {
            c += i >= 0 ? a[i--] - '0' : 0;
            c += j >= 0 ? b[j--] - '0' : 0;
            s.push_back(c % 2 + '0');
            c /= 2;
        }
        reverse(s.begin(), s.end());
        return s;          
    }
};
```
