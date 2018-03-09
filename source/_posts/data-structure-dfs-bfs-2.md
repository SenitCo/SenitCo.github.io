---
title: 深度优先遍历与广度优先遍历（二）
date: 2018-02-20
categories: Data Structure
tags: [code, dfs, bfs]
---
数据结构与算法中深度优先遍历(DFS)与广度优先遍历(BFS)问题总结归纳。
<!--more-->

### Word Ladder
题目描述：[LeetCode](https://leetcode.com/problems/word-ladder/description/)
Given two words (beginWord and endWord), and a dictionary's word list, find the length of shortest 
transformation sequence from beginWord to endWord, such that:
Only one letter can be changed at a time.
Each transformed word must exist in the word list. Note that beginWord is not a transformed word.

For example, Given:
beginWord = "hit"
endWord = "cog"
wordList = ["hot","dot","dog","lot","log","cog"]
As one shortest transformation is "hit" -> "hot" -> "dot" -> "dog" -> "cog",
return its length 5.

Note:
Return 0 if there is no such transformation sequence.
All words have the same length.
All words contain only lowercase alphabetic characters.
You may assume no duplicates in the word list.
You may assume beginWord and endWord are non-empty and are not the same.
```cpp
//对于求取最小长度(深度)，一般采用广度优先遍历
int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
        int size = wordList.size();
        int depth = 2;
        vector<bool> visited(size, false);
        queue<string> toVisit;
        findNextWord(beginWord, wordList, toVisit, visited);
        while(!toVisit.empty())
        {
            int num = toVisit.size();   
            //此处的for循环是为了分层，统计层数(深度)，如果只是遍历所有元素，可不需要
            for(int i = 0; i < num; i++)
            {
                string word = toVisit.front();
                toVisit.pop();
                if(word == endWord)
                    return depth;
                findNextWord(word, wordList, toVisit, visited);
            }
            depth++;
        }
        return 0;
    }

    void findNextWord(string& word, vector<string>& wordList, queue<string>& toVisit, vector<bool>& visited)
    {      
        for(int i = 0; i < wordList.size(); i++)
        {
            if(!visited[i] && numOfDiffLetter(word, wordList[i]) == 1)
            {
                toVisit.push(wordList[i]);
                visited[i] = true;
            }
        }
    }
        
    int numOfDiffLetter(string& word1, string& word2)
    {
        int cnt = 0;
        for(int i = 0; i < word1.size(); i++)
        {
            if(word1[i] != word2[i])
                cnt++;
        }
        return cnt;
    }
```

### Word Ladder II
题目描述：[LeetCode](https://leetcode.com/problems/word-ladder-ii/description/)
Given two words (beginWord and endWord), and a dictionary's word list, find all shortest transformation sequence(s) 
from beginWord to endWord, such that:
Only one letter can be changed at a time
Each transformed word must exist in the word list. Note that beginWord is not a transformed word.
For example, Given:
beginWord = "hit"
endWord = "cog"
wordList = ["hot","dot","dog","lot","log","cog"]
Return
  [
    ["hit","hot","dot","dog","cog"],
    ["hit","hot","lot","log","cog"]
  ]
Note:
Return an empty list if there is no such transformation sequence.
All words have the same length.
All words contain only lowercase alphabetic characters.
You may assume no duplicates in the word list.
You may assume beginWord and endWord are non-empty and are not the same.

```cpp
/**
直接采用广度优先遍历，不需要回溯。tricky: 对路径进行BFS，而不是对单词
*/
class Solution1 {
public:
    vector<vector<string>> findLadders(string beginWord, string endWord, vector<string>& wordList) {
        vector<vector<string>> result;
        queue<vector<string>> pathList; //队列中存储的为单词变换的路径
        pathList.push({beginWord});
        unordered_set<string> wordSet(wordList.begin(), wordList.end());
        unordered_set<string> visited;
        if(wordSet.find(endWord) == wordSet.end())  //endWord必在单词列表中
            return result;
        //wordSet.insert(endWord);
        int depth = 1, minDepth = INT_MAX;
        
        while (!pathList.empty())
        {
            vector<string> path = pathList.front();
            pathList.pop();
            if (path.size() > depth)    //用于更新深度（层数）
            {
                if (path.size() > minDepth) //如果已取得最小深度，且遍历完该层，则返回
                    break;
                for(string s : visited) //将集合中已访问的元素剔除
                    wordSet.erase(s);
                depth = path.size();
            }
            string word = path.back();
            //遍历单词列表中的所有的元素并且和目标单词逐字符比较的方式会造成算法效率较低
            for (unordered_set<string>::iterator iter = wordSet.begin(); iter != wordSet.end();)
            {
                if (numOfDiffLetter(word, *iter) == 1)
                {
                    vector<string> newPath = path;
                    newPath.push_back(*iter);
                    if (*iter == endWord)
                    {
                        result.push_back(newPath);  //将满足条件的路径存储到最终结果中
                        minDepth = depth;
                        break;
                    }
                    else
                    {
                        pathList.push(newPath); //将可能的路径存储到队列中
                        visited.insert(*iter);  //记录已访问（在路径）中的单词
                    }          
                }
                iter++;
            }
        }
        return result;
    }
    
private: 
    int numOfDiffLetter(string& word1, const string& word2)
    {
        int cnt = 0;
        for(int i = 0; i < word1.size(); i++)
        {
            if(word1[i] != word2[i])
                cnt++;
        }
        return cnt;
    }
};

/**
广度优先遍历 + 回溯：BFS构造可能的路径，Backtrack搜索满足条件的路径
*/
class Solution2 {
public:
    vector<vector<string>> findLadders(string beginWord, string endWord, vector<string>& wordList) {
        vector<vector<string>> result;
        unordered_set<string> wordSet(wordList.begin(), wordList.end());
        unordered_set<string> toVisit({beginWord}), temp;
        if (wordSet.find(endWord) == wordSet.end())
            return result;
        wordSet.erase(endWord);
        unordered_map<string, vector<string>> children; //存储每个单词的子结点
        bool found = false;
        int len = beginWord.length();

        while (!toVisit.empty() && !found)
        {
            for (string word : toVisit) //在单词集合（列表）中剔除父结点（上一层的单词结点）
                wordSet.erase(word);
            for (const string word : toVisit)   //搜索满足条件的子结点
            {
                string cur = word;
                for (int j = 0; j < len; j++)
                {
                    char ch = cur[j];
                    for (int k = 'a'; k <= 'z'; k++)
                    {
                        cur[j] = k;
                        if (cur == endWord)
                        {
                            found = true;
                            children[word].push_back(endWord);
                            break;
                        }
                        else if (wordSet.count(cur) && !found)
                        {
                            children[word].push_back(cur);
                            temp.insert(cur);  
                        }
                    }
                    cur[j] = ch;
                }
            }
            swap(toVisit, temp);
            temp.clear();         
        }
        
        if (found)
        {
            vector<string> path({ beginWord });
            backtrack(beginWord, endWord, children, path, result);
        }
        return result;
    }

private:
    //在所有可能的路径中递归回溯，搜索满足条件的路径
    void backtrack(string& word, string& endWord, unordered_map<string, vector<string>>& children, 
        vector<string>& path, vector<vector<string>>& result)
    {
        if (word == endWord)
        {
            result.push_back(path);
            return;
        }
        if (children.find(word) == children.end())
            return;
        for (string child : children[word])
        {
            path.push_back(child);
            backtrack(child, endWord, children, path, result);
            path.pop_back();
        }
    }
};

/**
双向广度优先遍历 + 回溯，算法效率较高
See http://zxi.mytechroad.com/blog/searching/leetcode-126-word-ladder-ii/
*/
class Solution3 {
public:
    vector<vector<string>> findLadders(string beginWord, string endWord, vector<string>& wordList) {
        vector<vector<string>> ans;
        unordered_set<string> dict(wordList.begin(), wordList.end());
        if (!dict.count(endWord)) return ans;
        
        int l = beginWord.length();
        
        unordered_set<string> q1{beginWord};
        unordered_set<string> q2{endWord};
        unordered_map<string, vector<string>> children;
 
        bool found = false;
        bool backward = false;
        
        while (!q1.empty() && !q2.empty() && !found) {            
            // Always expend the smaller queue first
            if (q1.size() > q2.size()) {
                std::swap(q1, q2);
                backward = !backward;
            }
            
            for (const string& w : q1)
                dict.erase(w);
            for (const string& w : q2)
                dict.erase(w);
                        
            unordered_set<string> q;
            
            for (const string& word : q1) {
                string curr = word;
                for (int i = 0; i < l; i++) {
                    char ch = curr[i];
                    for (int j = 'a'; j <= 'z'; j++) {
                        curr[i] = j;
                        
                        const string* parent = &word;
                        const string* child = &curr;
                        
                        if (backward)
                            std::swap(parent, child);
                        
                        if (q2.count(curr)) {
                            found = true;
                            children[*parent].push_back(*child);
                        } else if (dict.count(curr) && !found) {
                            q.insert(curr);
                            children[*parent].push_back(*child);
                        }
                    }
                    curr[i] = ch;
                }
            }
            
            std::swap(q, q1);
        }
        
        if (found) {
            vector<string> path{beginWord};
            getPaths(beginWord, endWord, children, path, ans);
        }
        
        return ans;
    }
private:
    void getPaths(const string& word, 
                  const string& endWord,                   
                  const unordered_map<string, vector<string>>& children, 
                  vector<string>& path, 
                  vector<vector<string>>& ans) {        
        if (word == endWord) {
            ans.push_back(path);
            return;
        }
        
        const auto it = children.find(word);
        if (it == children.cend()) return;
        
        for (const string& child : it->second) {
            path.push_back(child);
            getPaths(child, endWord, children, path, ans);
            path.pop_back();
        }
    }
};
```

