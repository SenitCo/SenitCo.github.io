---
title: 字符串(String)问题集锦
date: 2018-04-08
categories: Data Structure
tags: [code, string]
---
数据结构与算法中字符串问题总结归纳。
<!--more-->

### Longest Common Prefix
[Description](https://leetcode.com/problems/longest-common-prefix/description/): Write a function to find the longest common prefix string amongst an array of strings.
```cpp
string longestCommonPrefix(vector<string>& strs) 
{
    if(strs.empty())
        return "";
    int len = INT_MAX;
    for(int i = 0; i < strs.size(); i++)
    {
        if(len > strs[i].length())
            len = strs[i].length();
    }
    for(int i = 0; i < len; i++)
    {
        char value = strs[0][i];
        for(int j = 1; j < strs.size(); j++)
        {
            if(strs[j][i] != value)
                return strs[0].substr(0, i);
        }
            
    }
    return strs[0].substr(0, len);
}
```

### Implement strStr()
[Description](https://leetcode.com/problems/implement-strstr/description/): Return the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.
Example 1: Input: haystack = "hello", needle = "ll", Output: 2
Example 2: Input: haystack = "aaaaa", needle = "bba", Output: -1
```cpp
/*****调用STL函数******/
class Solution {
public:
    int strStr(string haystack, string needle) {
        return haystack.find(needle);
    }
};

/*****朴素匹配算法*****/
class Solution {
public:
    int strStr(string haystack, string needle) {
        int len1 = haystack.length(), len2 = needle.length();
        for(int i = 0; i < len1 - len2 + 1; i++)
        {
            int j = 0;
            for(; j < len2; j++)
            {
                if(haystack[i + j] != needle[j])
                    break;
            }
            if(j == len2)
                return i;
        }
        return -1;
    }
};

/***************KMP算法*******************/
class Solution {
public:
    int strStr(string haystack, string needle) {
        int len1 = haystack.length(), len2 = needle.length();
        if(len2 == 0)
            return 0;
        vector<int> next = getNext(needle);
        int i = 0, j = 0;
        while(i < len1 && j < len2)
        {
            if(j == -1 || haystack[i] == needle[j])
            {
                i++;
                j++;
            }
            else
                j = next[j];
        }
        if(j == len2)
            return i - j;
        return -1;
    }
private:
    vector<int> getNext(string& needle)
    {
        int len = needle.length();
        vector<int> next(len + 1);  //注意：next数组长度要大于子串长度
        next[0] = -1;
        for(int i = 0, j = -1; i < len;)
        {
            if(j == -1 || needle[i] == needle[j])
            {
                i++;    //i=len-1时，此处的自增可能导致下面next[i]溢出，因此next数组长度要大于子串长度
                j++;
                next[i] = j;
            }
            else 
                j = next[j];
        }
        return next;
    }
};

/***************KMP算法（改进版本）*******************/
class Solution {
public:
    int strStr(string haystack, string needle) {
        int len1 = haystack.length(), len2 = needle.length();
        if(len2 == 0)
            return 0;
        vector<int> next = getNext(needle);
        int i = 0, j = 0;
        while(i < len1 && j < len2)
        {
            if(j == -1 || haystack[i] == needle[j])
            {
                i++;
                j++;
            }
            else
                j = next[j];
        }
        if(j == len2)
            return i - j;
        return -1;
    }
private:
    vector<int> getNext(string& needle)
    {
        int len = needle.length();
        vector<int> next(len + 1);
        next[0] = -1;
        for(int i = 0, j = -1; i < len;)
        {
            if(j == -1 || needle[i] == needle[j])
            {
                i++;
                j++;
                if(needle[i] != needle[j])
                    next[i] = j;
                else
                    next[i] = next[j];
            }
            else 
                j = next[j];
        }
        return next;
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

### Multiply Strings
[Description](https://leetcode.com/problems/multiply-strings/description/): Given two non-negative integers num1 and num2 represented as strings, return the product of num1 and num2.
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

### Add Two Numbers
[Description](https://leetcode.com/problems/add-two-numbers/description/): You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list. You may assume the two numbers do not contain any leading zero, except the number 0 itself
```cpp
ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) 
{
    ListNode *pL1 = l1, *pL2 = l2;
    ListNode *result = new ListNode(0), *pResult = result;
    int value = 0;
    while(pL1 || pL2)
    {
        value /= 10;
        if(pL1)
        {
            value += pL1->val;
            pL1 = pL1->next;
        }
        if(pL2)
        {
            value += pL2->val;
            pL2 = pL2->next;
        }   
       
        pResult->next = new ListNode(value % 10);
        pResult = pResult->next;
    }
    if(value / 10 == 1 )
        pResult->next = new ListNode(1);
    return result->next;
}

ListNode *addTwoNumbers(ListNode *l1, ListNode *l2) {
    ListNode preHead(0), *p = &preHead;
    int extra = 0;
    while (l1 || l2 || extra) {
        int sum = (l1 ? l1->val : 0) + (l2 ? l2->val : 0) + extra;
        extra = sum / 10;
        p->next = new ListNode(sum % 10);
        p = p->next;
        l1 = l1 ? l1->next : l1;
        l2 = l2 ? l2->next : l2;
    }
    return preHead.next;
}
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

### Count and Say
The count-and-say sequence is the sequence of integers with the first five terms as following:
1.     1
2.     11
3.     21
4.     1211
5.     111221
1 is read off as "one 1" or 11.
11 is read off as "two 1s" or 21.
21 is read off as "one 2, then one 1" or 1211.
Given an integer n, generate the nth term of the count-and-say sequence.
Note: Each term of the sequence of integers will be represented as a string.
Example 1: Input: 1, Output: "1"
Example 2: Input: 4, Output: "1211"
```cpp
/**
第i+1个字符串是第i个字符串的读法，第一字符串为 “1”
比如第四个字符串是1211，它的读法是 1个1、1个2,2个1，因此第五个字符串是111221。
第五个字符串的读法是：3个1、2个2、1个1，因此第六个字符串是312211 
*/
class Solution {
public:
    string countAndSay(int n) {
        if(n < 1)   return "";
        string result = "1";
        for(int i = 1; i < n; i++)
        {
            string temp;
            int count = 1;
            char prev = result[0];
            result.push_back('#');      //处理边界条件，避免退出下面的循环后再进行一次+=和push处理
            for(int j = 1; j < result.size(); j++)
            {
                if(prev == result[j])
                    count++;
                else
                {
                    temp += to_string(count);
                    temp.push_back(prev);
                    prev = result[j];
                    count = 1;
                }
            }
            result = temp;
        }
        return result;
    }
};

string countAndSay(int n) 
{
    if (n == 0) return "";
    string res = "1";
    while (--n) {
        string cur = "";
        for (int i = 0; i < res.size(); i++) {
            int count = 1;
             while ((i + 1 < res.size()) && (res[i] == res[i + 1])){
                count++;    
                i++;
            }
            cur += to_string(count) + res[i];
        }
        res = cur;
    }
    return res;
}
```

### Restore IP Addresses
[Description](https://leetcode.com/problems/restore-ip-addresses/description/): Given a string containing only digits, restore it by returning all possible valid IP address combinations. For example: Given "25525511135", return ["255.255.11.135", "255.255.111.35"]. (Order does not matter)

```cpp
/**
迭代组合
*/
class Solution {
public:
    vector<string> restoreIpAddresses(string s) {
        int len = s.length();
        vector<string> result;
        for(int i = 1; i < 4 && i < len - 2; i++)
        {
            for(int j = i + 1; j < i + 4 && j < len - 1; j++)
            {
                for(int k = j + 1; k < j + 4 && k < len; k++)
                {
                    if(len - k > 3) continue;   //如不满足条件，提前终止
                    string s1 = s.substr(0, i);
                    string s2 = s.substr(i, j - i);
                    string s3 = s.substr(j, k - j);
                    string s4 = s.substr(k, len - k);
                    if(isValid(s1) && isValid(s2) && isValid(s3) && isValid(s4))
                        result.push_back(s1 + '.' + s2 + '.' + s3 + '.' + s4);
                }
            }
        }
        return result;
    }
private:
    bool isValid(string s)
    {
        if(std::stoi(s) > 255 || (s[0] == '0' && s.length() > 1))
            return false;
        return true;
    }
};

/**
递归法
*/
class Solution {
public:
    vector<string> restoreIpAddresses(string s) {
        int len = s.length();
        vector<string> result;
        string restored;
        restoreIP(s, result, 0, restored, 0);
        return result;
    }
private:
    void restoreIP(string s, vector<string>& result, int index, string restored, int cnt)
    {
        if (cnt > 4) return;
        if (cnt == 4 && index == s.length())
            result.push_back(restored);
        for (int i = 1; i < 4; i++)
        {
            if (index + i > s.length())  break;
            string str = s.substr(index, i);
            if ((str[0] == '0' && str.length() > 1) || (i == 3 && stoi(str) > 255))
                continue;
            restoreIP(s, result, index + i, restored + str + (cnt == 3 ? "" : "."), cnt + 1);
        }
    }
};
```

### Integer to Roman
[Description](https://leetcode.com/problems/integer-to-roman/description/): Given an integer, convert it to a roman numeral. Input is guaranteed to be within the range from 1 to 3999.
```cpp
/*
I = 1;
V = 5;
X = 10;
L = 50;
C = 100;
D = 500;
M = 1000;
其中每两个阶段之间有一个减法的表示，比如900=CM， C写在M前面表示M-C。
*/
string intToRoman(int num) 
{
    string result;    
    string symbol[] = { "M","CM","D","CD","C","XC","L","XL","X","IX","V","IV","I" };    
    int value[] = {1000,900,500,400, 100, 90,  50, 40,  10, 9,   5,  4,   1};   
    for(int i = 0; num != 0; ++i)  
    {  
        while(num >= value[i])  
        {  
            num -= value[i];  
            result += symbol[i];  
        }  
    }  
    return result;  
}  
```

### Roman to Integer
[Description](https://leetcode.com/problems/roman-to-integer/description/): Given a roman numeral, convert it to an integer. Input is guaranteed to be within the range from 1 to 3999.
```cpp
int romanToInt(string s) 
{
    int ret = toNumber(s[0]);  
    for (int i = 1; i < s.length(); i++) 
    {  
        if (toNumber(s[i - 1]) < toNumber(s[i])) 
        {  
            ret += toNumber(s[i]) - 2 * toNumber(s[i - 1]);  
        }
        else 
        {  
            ret += toNumber(s[i]);  
        }  
    }  
    return ret;  
}

int toNumber(char ch) 
{  
    switch (ch) 
    {  
        case 'I': return 1;  
        case 'V': return 5;  
        case 'X': return 10;  
        case 'L': return 50;  
        case 'C': return 100;  
        case 'D': return 500;  
        case 'M': return 1000;  
    }  
    return 0;  
}  
```

### String to Integer (atoi)
[Description](https://leetcode.com/problems/string-to-integer-atoi/description/): Implement atoi to convert a string to an integer.
```cpp
/*处理四种情况：开头连续的空格；正负符号；不合理的输入；溢出
不合理的输入"+-123"、"- 123"、"b123"等均输出0，"123a"这种情况输出前面的合理数字123
*/
class Solution {
public:
    int myAtoi(string str) {
        double result = 0;
        int i = 0, flag = 0;

        while(str[i] == ' ')
            i++;
        if(str[i] == '-')
        {
            flag = 1;
            i++;
        }
        else if(str[i] == '+')
        {
            i++;
        }
       
        for(; i < str.size(); i++)
        {
            if(str[i] > '9' || str[i] < '0')
                break;
            result = result * 10 + str[i]   - '0';
        }
        if(flag)
            result = -result;
        if(result > INT_MAX)
            result = INT_MAX;
        if(result < INT_MIN)
            result = INT_MIN;
        return (int)result;
    }
};

int myAtoi(string str) 
{
    int sign = 1, base = 0, i = 0;
    while (str[i] == ' ') { i++; }
    if (str[i] == '-' || str[i] == '+') 
    {
        sign = 1 - 2 * (str[i++] == '-'); 
    }
    while (str[i] >= '0' && str[i] <= '9')
    {
        if (base >  INT_MAX / 10 || (base == INT_MAX / 10 && str[i] - '0' > 7)) 
        {
            if (sign == 1) 
                return INT_MAX;
            else 
                return INT_MIN;
        }
        base  = 10 * base + (str[i++] - '0');
    }
    return base * sign;
}
```

### ZigZag Conversion
[Description](https://leetcode.com/problems/zigzag-conversion/description/)
```cpp
/**
The string "PAYPALISHIRING" is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility).
P   A   H   N
A P L S I I G
Y   I   R
And then read line by line: "PAHNAPLSIIGYIR"
*/

/**
将字符串按之字形排列，然后逐行读取字符以“之”字形出现的周期为 (T=2*n-2)，n为总行数，对于每一行，先读取往下走的字符，然后读取斜向上走的字符，
且两个字符之间的距离为 (T-2*i)，i为行数。
*/
class Solution {
public:
    string convert(string s, int numRows) {
        if(numRows <= 1)
            return s;
        string result;
        if(s.empty())
            return result;
        int period = numRows + numRows - 2;
        int length = s.length();
        
        for(int i = 0; i < numRows; i++ )
        {
            for(int j = i; j < length; j += period)
            {                    
                result.append(1, s[j]);
                if(i != 0 && i != numRows - 1 && j + period - 2 * i < length)
                {
                    result.append(1, s[j + period - 2 * i]);
                }
            }
        }
        return result;
    }
};
```

