---
title: 深度优先遍历与广度优先遍历（一）
date: 2018-02-18
categories: Data Structure
tags: [code, dfs, bfs]
---
数据结构与算法中深度优先遍历(DFS)与广度优先遍历(BFS)问题总结归纳。
<!--more-->

### Clone Graph
题目描述：[LeetCode](https://leetcode.com/problems/clone-graph/description/)
Clone an undirected graph. Each node in the graph contains a label and a list of its neighbors.
OJ's undirected graph serialization:
Nodes are labeled uniquely.
We use # as a separator for each node, and , as a separator for node label and each neighbor of the node.


```cpp
/**
As an example, consider the serialized graph {0,1,2#1,2#2,2}.
The graph has a total of three nodes, and therefore contains three parts as separated by #.
First node is labeled as 0. Connect node 0 to both nodes 1 and 2.
Second node is labeled as 1. Connect node 1 to node 2.
Third node is labeled as 2. Connect node 2 to node 2 (itself), thus forming a self-cycle.
Visually, the graph looks like the following:
       1
      / \
     /   \
    0 --- 2
         / \
         \_/
*/

//Definition for undirected graph.
struct UndirectedGraphNode 
{
    int label;
    vector<UndirectedGraphNode *> neighbors;
    UndirectedGraphNode(int x) : label(x) {};
};

/**广度优先遍历
创建一个hash表，用于将原图中的所有节点和新拷贝的节点一一对应，然后采用广度优先遍历的方法依次访问原图节点，
拷贝创建对应的新节点，并处理邻接关系
*/
class Solution {
public:
    UndirectedGraphNode *cloneGraph(UndirectedGraphNode *node) {
        if(!node)   return NULL;
        unordered_map<UndirectedGraphNode*, UndirectedGraphNode*> mp;
        UndirectedGraphNode* copy = new UndirectedGraphNode(node->label);
        mp[node] = copy;
        queue<UndirectedGraphNode*> toVisit;
        toVisit.push(node);
        while(!toVisit.empty())
        {
            UndirectedGraphNode* cur = toVisit.front();
            toVisit.pop();
            for(UndirectedGraphNode* neigh : cur->neighbors)
            {
                if(mp.find(neigh) == mp.end())  //如果原图节点的对应结点不存在则创建
                {
                    UndirectedGraphNode* copy_neigh = new UndirectedGraphNode(neigh->label);
                    mp[neigh] = copy_neigh;
                    toVisit.push(neigh);
                }
                mp[cur]->neighbors.push_back(mp[neigh]);    //存储新节点的相邻结点
            }
        }
        return copy;
    }
};

//深度优先遍历（递归法）
class Solution {
private:
    unordered_map<UndirectedGraphNode*, UndirectedGraphNode*> mp;
public:
    UndirectedGraphNode *cloneGraph(UndirectedGraphNode *node) {
        if(!node)   return NULL;
        if(mp.find(node) == mp.end())
        {
            mp[node] = new UndirectedGraphNode(node->label);     
            for(UndirectedGraphNode* neigh : node->neighbors)
            {
                mp[node]->neighbors.push_back(cloneGraph(neigh));
            }
        }
        return mp[node];
    }
};
```

### Surrounded Regions
题目描述：[LeetCode](https://leetcode.com/problems/surrounded-regions/description/)
Given a 2D board containing 'X' and 'O' (the letter O), capture all regions surrounded by 'X'.
A region is captured by flipping all 'O's into 'X's in that surrounded region.
For example,
X X X X
X O O X
X X O X
X O X X
After running your function, the board should be:
X X X X
X X X X
X X X X
X O X X
```cpp
//广度优先遍历(BFS)
class Solution {
public:
    void solve(vector<vector<char>>& board) {
        if(board.empty())    return;
        int height = board.size(), width = board[0].size();
        for(int i = 0; i < height; i++) //从左右边界开始搜索
        {
            if(board[i][0] == 'O')
                bfs(board, i, 0);
            if(board[i][width - 1] == 'O')
                bfs(board, i, width - 1);
        }
        
        for(int j = 0; j < width; j++)  //从上下边界开始搜索
        {
            if(board[0][j] == 'O')
                bfs(board, 0, j);
            if(board[height - 1][j] == 'O')
                bfs(board, height - 1, j);
        }
        
        for(int i = 0; i < height; i++)
        {
            for(int j = 0; j < width; j++)
            {
                if(board[i][j] == 'O')
                    board[i][j] = 'X';
                else if(board[i][j] == 'B')
                    board[i][j] = 'O';
            }
        }
        
    }
private:
    void bfs(vector<vector<char>>& board, int i, int j)
    {
        int height = board.size(), width = board[0].size();
        queue<pair<int, int>> toVisit;
        board[i][j] = 'B';
        toVisit.push(make_pair(i, j));
        while(!toVisit.empty())
        {
            pair<int, int> cur = toVisit.front();
            toVisit.pop();
            if(cur.first > 0 && board[cur.first - 1][cur.second] == 'O')
            {
                toVisit.push(make_pair(cur.first - 1, cur.second));
                board[cur.first - 1][cur.second] = 'B';
            }
            if(cur.first < height - 1 && board[cur.first + 1][cur.second] == 'O')
            {
                toVisit.push(make_pair(cur.first + 1, cur.second));
                board[cur.first + 1][cur.second] = 'B';
            }
            if(cur.second > 0 && board[cur.first][cur.second - 1] == 'O')
            {
                toVisit.push(make_pair(cur.first, cur.second - 1));
                board[cur.first][cur.second - 1] = 'B';
            }
            if(cur.second < width - 1 && board[cur.first][cur.second + 1] == 'O')
            {
                toVisit.push(make_pair(cur.first, cur.second + 1));
                board[cur.first][cur.second + 1] = 'B';
            }
        } 
    }
};

//深度优先遍历(DFS)
class Solution {
public:
    void solve(vector<vector<char>>& board) {
        if(board.empty())    return;
        int height = board.size(), width = board[0].size();
        for(int i = 0; i < height; i++)
        {
            if(board[i][0] == 'O')
                dfs(board, i, 0);
            if(board[i][width - 1] == 'O')
                dfs(board, i, width - 1);
        }
        
        for(int j = 0; j < width; j++)
        {
            if(board[0][j] == 'O')
                dfs(board, 0, j);
            if(board[height - 1][j] == 'O')
                dfs(board, height - 1, j);
        }
        
        for(int i = 0; i < height; i++)
        {
            for(int j = 0; j < width; j++)
            {
                if(board[i][j] == 'O')
                    board[i][j] = 'X';
                else if(board[i][j] == 'B')
                    board[i][j] = 'O';
            }
        }
        
    }
private:
    void dfs(vector<vector<char>>& board, int i, int j)
    {
        int height = board.size(), width = board[0].size();
        if(i >= 0 && i < height && j >= 0 && j < width && board[i][j] == 'O')
        {
            board[i][j] = 'B';
            dfs(board, i - 1, j);
            dfs(board, i + 1, j);
            dfs(board, i, j - 1);
            dfs(board, i, j + 1);
        }
    }
};

```

