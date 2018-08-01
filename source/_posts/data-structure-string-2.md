---
title: 字符串(String)问题、哈希表、双指针
date: 2018-04-02
categories: Data Structure
tags: [code, string, hash table, two pointers]
---
数据结构与算法中字符串(String)问题总结归纳。
<!--more-->

### Substring with Concatenation of All Words
[Description](https://leetcode.com/problems/substring-with-concatenation-of-all-words/description/): You are given a string, s, and a list of words, words, that are all of the same length. Find all starting indices of substring(s) in s that is a concatenation of each word in words exactly once and without any intervening characters.
For example, given: s: "barfoothefoobarman", words: ["foo", "bar"]. You should return the indices: [0,9]. (order does not matter).
```cpp
/*
Using an unordered_map<string, int> counts to record the expected times of each word and another 
unordered_map<string, int> seen to record the times we have seen. Then checking for every possible 
position of i. Once we meet an unexpected word or the times of some word is larger than its expected 
times, we stop the check. If we finish the check successfully, push i to the result indexes.
*/
class Solution {
public:
    vector<int> findSubstring(string s, vector<string>& words) {
        unordered_map<string, int>counts;
        for(string word : words)
        {
            counts[word]++;
        }
        int length = s.length(), num = words.size(), len = words[0].length();
        vector<int> result;
        for(int i = 0; i < length - num * len + 1; i++)
        {
            int j = 0;
            unordered_map<string, int> seen;
            for(; j < num; j++)
            {
                string word = s.substr(i + j * len, len);
                if(counts.find(word) != counts.end())
                {
                    seen[word]++;
                    if(seen[word] > counts[word])
                        break;
                }
                else
                    break;
            }
            if(j == num)
                result.push_back(i);
        }
        return result;
    }
};
```

### Minimum Window Substring
[Description](https://leetcode.com/problems/minimum-window-substring/description/): Given a string S and a string T, find the minimum window in S which will contain all the characters in T in complexity O(n).
For example, S = "ADOBECODEBANC", T = "ABC", Minimum window is "BANC".

Note:
If there is no such window in S that covers all characters in T, return the empty string "".
If there are multiple such windows, you are guaranteed that there will always be only one unique minimum window in S.
```cpp
/**
1.用两个“指针”begin、end指向s中字符代表一个窗口
2.移动end指针搜索有效窗口(窗口中包含t中所有字符)
3.移动begin指针缩小有效窗口范围
为了让[begin, end)窗口成为一个有效窗口，需要一个映射表map(char, count)，存储t中每个字符出现的次数（t中字符可重复），
并定义一个counter统计t中的字符数，如果s中出现t中字符，即map[s[end]]>0，则counter--，直至counter为0，说明已获取一个
有效窗口，因此移动begin指针缩小窗口范围。一个比较关键的步骤就是end指针每次步进前，都会执行map[s[end]]--操作，这样
如果t中已不存在对应字符，就会使得该字符的统计数为负。在得到一个有效的窗口（counter=0）时，判断现有窗口和之前窗口的
大小，取窗口较小值，并记录起始位置head=begin，如果遇到map[s[begin]] == 0，说明遇到了t中一个字符且该字符为有效窗口的
左边界，begin指针再往前走，窗口中就不包含t中所有字符，因此counter计数值自增。和end指针一样，begin指针每次步进前都要
执行map[s[begin]]++操作，对于s="bba",t="ba"，t中字符'b'只出现一次，而s中出现两次，end指针的map[s[end]]--操作使得
map['b']<0，因此begin指针在步进时需要逐步恢复每个字符的统计值。
*/
class Solution {
public:
    string minWindow(string s, string t) {
        //vector<int> map(128, 0);
        unordered_map<char, int> map;
        for(int i = 0; i < t.size(); i++)
            map[t[i]]++;
        int head = 0, begin = 0, end = 0, len = INT_MAX, count = t.size();
        while(end < s.size())
        {
            if(map[s[end]] > 0)
                count--;
            map[s[end]]--;
            end++;
            while(count == 0)
            {
                if(end - begin < len)
                {
                    head = begin;
                    len = end - begin;
                }
                if(map[s[begin]] == 0)
                    count++;
                map[s[begin]]++;
                begin++;
            }
        }
        return len == INT_MAX ? "" : s.substr(head, len);
    }
};

// See https://leetcode.com/problems/minimum-window-substring/discuss/26808

/**
可读性较差的简化版本
*/
string minWindow(string s, string t) 
{
    vector<int> map(128,0);
    for(auto c: t) map[c]++;
    int counter=t.size(), begin=0, end=0, d=INT_MAX, head=0;
    while(end<s.size()){
        if(map[s[end++]]-->0) counter--; //in t
        while(counter==0){ //valid
            if(end-begin<d)  d=end-(head=begin);
            if(map[s[begin++]]++==0) counter++;  //make it invalid
        }  
    }
    return d==INT_MAX? "":s.substr(head, d);
}


/***************子串搜索的模板*******************/

/**
For most substring problem, we are given a string and need to find a substring of it which satisfy some restrictions. 
A general way is to use a hashmap assisted with two pointers. The template is given below.
**/
int findSubstring(string s)
{
    vector<int> map(128,0);
    int counter; // check whether the substring is valid
    int begin=0, end=0; //two pointers, one point to tail and one  head
    int d; //the length of substring

    for() { /* initialize the hash map here */ }

    while(end<s.size()){

        if(map[s[end++]]-- ?){  /* modify counter here */ }

        while(/* counter condition */){ 
             
             /* update d here if finding minimum*/

            //increase begin to make it invalid/valid again
            
            if(map[s[begin++]]++ ?){ /*modify counter here*/ }
        }  

        /* update d here if finding maximum*/
    }
    return d;
}

/**
One thing needs to be mentioned is that when asked to find maximum substring, we should update maximum after the inner 
while loop to guarantee that the substring is valid. On the other hand, when asked to find minimum substring, we should 
update minimum inside the inner while loop.
*/

/**
The code of solving Longest Substring with At Most Two Distinct Characters is below:
*/
int lengthOfLongestSubstringTwoDistinct(string s) 
{
    vector<int> map(128, 0);
    int counter=0, begin=0, end=0, d=0; 
    while(end<s.size()){
        if(map[s[end++]]++==0) counter++;
        while(counter>2) if(map[s[begin++]]--==1) counter--;
        d=max(d, end-begin);
    }
    return d;
}

/**
The code of solving Longest Substring Without Repeating Characters is below:
*/

int lengthOfLongestSubstring(string s) 
{
    vector<int> map(128,0);
    int counter=0, begin=0, end=0, d=0; 
    while(end<s.size()){
        if(map[s[end++]]++>0) counter++; 
        while(counter>0) if(map[s[begin++]]-->1) counter--;
        d=max(d, end-begin); //while valid, update d
    }
    return d;
}
```


### Longest Substring Without Repeating Characters
[Description](https://leetcode.com/problems/longest-substring-without-repeating-characters/description/): Given a string, find the length of the longest substring without repeating characters.
Examples:
Given "abcabcbb", the answer is "abc", which the length is 3.
Given "bbbbb", the answer is "b", with the length of 1.
Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, 
"pwke" is a subsequence and not a substring.

```cpp
int lengthOfLongestSubstring(string s) 
{
    unordered_map<char, int> map;
    int begin = 0, end = 0, len = 0;
    while(end < s.length())
    {
        map[s[end]]++;
        while(map[s[end]] > 1)
        {
            map[s[begin]]--;
            begin++;
        }
        end++;
        if(end - begin > len)
            len = end - begin;
    }
    return len;
}

int lengthOfLongestSubstring(string s) 
{
    vector<int> dict(256, -1);
    int maxLen = 0, start = -1;
    for (int i = 0; i != s.length(); i++) 
    {
        if (dict[s[i]] > start)
            start = dict[s[i]];
        dict[s[i]] = i;
        maxLen = max(maxLen, i - start);
    }
    return maxLen;
}
```

### Longest Palindromic Substring
[Description](https://leetcode.com/problems/longest-palindromic-substring/description/): Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.
Example 1: Input: "babad", Output: "bab", Note: "aba" is also a valid answer.
Example 2: Input: "cbbd", Output: "bb"

```cpp
/*reference
依次遍历字符串中的元素，并以该元素为中心点向两边搜索，判断两边元素是否对称，并延伸至最大长度，
而且在中心点处考虑了重复元素（连续出现）的情况
*/
string longestPalindrome(string s) 
{
    if (s.empty()) 
        return "";
    if (s.size() == 1) 
        return s;
    int min_start = 0, max_len = 1;
    for (int i = 0; i < s.size();) 
    {
        if (s.size() - i <= max_len / 2)    
            break;
        int j = i, k = i;
        while (k < s.size()-1 && s[k+1] == s[k])    // Skip duplicate characters.
            ++k; 
        i = k + 1;
        while (k < s.size()-1 && j > 0 && s[k + 1] == s[j - 1])     // Expand.
        { 
            ++k; 
            --j; 
        }
        int new_len = k - j + 1;
        if (new_len > max_len)
        { 
            min_start = j; 
            max_len = new_len; 
        }
    }
    return s.substr(min_start, max_len);
}

/*依次遍历字符串中每个字符，以该字符为起点，截取剩余长度子串，判断是否为回文序列，若不是，则子串长度递减，
直至不大于当前最长的回文序列；在外层循环中，若剩余长度不大于当前最长回文串的长度，则直接退出，时间复杂度O(N^2)
*/
string longestPalindrome(string s) 
{
    if(s.length() <= 1)
        return s;
    int lenS = s.length(), start = 0, maxLen = 1, len = 0;
    for(int i = 0; i < lenS; i++)
    {
        for(len = lenS - i; len > maxLen; len--)
        {
            int mid = i + len / 2;
            int j = i, k = i + len - 1;
            for(; j < mid && s[j] == s[k]; j++, k--);
            if(j == mid && len > maxLen)
            {
                start = i;
                maxLen = len;
                break;
            }
        }
        if(len < maxLen)
            break;
    }
    return s.substr(start, maxLen);
}
```

### Palindromic Substrings
[Description](https://leetcode.com/problems/palindromic-substrings/description/): Given a string, your task is to count how many palindromic substrings in this string. The substrings with different start indexes or end indexes are counted as different substrings even they consist of same characters.
Example 1: Input: "abc", Output: 3. Explanation: Three palindromic strings: "a", "b", "c".
Example 2: Input: "aaa", Output: 6. Explanation: Six palindromic strings: "a", "a", "a", "aa", "aa", "aaa".
Note: The input string length won't exceed 1000.
```cpp
class Solution {
public:
    int countSubstrings(string s) {
        int count = 0;
        for(int i = 0; i < s.length(); i++)
        {
            countPalindrome(s, i, i, count);
            countPalindrome(s, i, i + 1, count);
        }
        return count;
    }
    void countPalindrome(string& s, int left, int right, int& count)
    {
        while(left >= 0 && right < s.length() && s[left] == s[right])
        {
            count++;
            left--;
            right++;
        }
    }
};
```

### Group Anagrams
[Description](https://leetcode.com/problems/group-anagrams/description/): Given an array of strings, group anagrams together.
For example, given: ["eat", "tea", "tan", "ate", "nat", "bat"], 
Return:
[
  ["ate", "eat","tea"],
  ["nat","tan"],
  ["bat"]
]
Note: All inputs will be in lower-case.
```cpp
/**
将每个字符串进行排序，并将排序后的字符串作为hash键值，对原始字符串进行索引分类
*/
vector<vector<string>> groupAnagrams(vector<string>& strs) 
{
    vector<vector<string>> results;
    unordered_map<string, vector<string>> map;
    for(string str : strs)
    {
        string word = str;
        sort(word.begin(), word.end());
        map[word].push_back(str);
    }
    for(auto iter = map.begin(); iter != map.end(); iter++)
    {
        results.push_back(iter->second);
    }
    return results;
}
```
