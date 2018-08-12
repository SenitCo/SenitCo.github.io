---
title: 二叉树问题集锦（一）
date: 2018-02-03
categories: Data Structure
tags: [code, binary tree, preorder, inorder, postorder]
---

数据结构中经典二叉树问题的总结归纳。
<!--more-->
二叉树结点定义：
```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};
```

### 二叉树先序遍历
问题描述：[LeetCode](https://leetcode.com/problems/binary-tree-preorder-traversal/description/)
分析：二叉树的遍历可通过递归法和迭代法求解。
```cpp
//递归法
vector<int> preorderTraversal(TreeNode* root) 
{
    vector<int> result;
    traverse(root, result);
    return result;
}

void traverse(TreeNode* node, vector<int>& result)
{
    if(!node)   return;
    result.push_back(node->val);
    traverse(node->left, result);
    traverse(node->right, result);
}

//迭代法（一）
vector<int> preorderTraversal(TreeNode* root) 
{
    stack<TreeNode*> toVisit;
    vector<int> result;
    while(root != NULL || !toVisit.empty())
    {
        while(root)
        {
            result.push_back(root->val);
            toVisit.push(root); //此处是进栈的时候访问结点元素值
            root = root->left;
        }
        root = toVisit.top();
        toVisit.pop();
        root = root->right;
    }
    return result;
}

//迭代法（二）
vector<int> preorderTraversal(TreeNode* root) 
{
    vector<int> result;
    if(!root)   return result;
    stack<TreeNode*> toVisit;
    toVisit.push(root);
    while(!toVisit.empty())
    {
        root = toVisit.top();
        toVisit.pop();
        result.push_back(root->val);    
        if(root->right)  //注意此处要先将右子结点压入栈中，因为是出栈的时候访问结点元素值
            toVisit.push(root->right); 
        if(root->left)
            toVisit.push(root->left);
    }
    return result;
}
```

### 二叉树中序遍历
问题描述：[LeetCode](https://leetcode.com/problems/binary-tree-inorder-traversal/description/)
```cpp
//递归法
vector<int> inorderTraversal(TreeNode* root) 
{
    vector<int> result;
    traverse(root, result);
    return result;
}

void traverse(TreeNode* node, vector<int>& result)
{
    if(node == NULL)    return;
    traverse(node->left, result);
    result.push_back(node->val);
    traverse(node->right, result);
}

//迭代法
vector<int> inorderTraversal(TreeNode* root) 
{
    vector<int> nodes;
    std::stack<TreeNode*> toVisit;
    while(1) {
        while(root) 
        { 
            toVisit.push(root); 
            root = root->left; 
        }
        if(toVisit.empty()) break;
        root = toVisit.top(); 
        toVisit.pop();
        nodes.push_back(root->val); //和先序遍历不同，此处是出栈访问结点元素值
        root = root->right;
    }
    return nodes;
}

//和上述迭代法思路一致，只是将while循环替换成if条件语句
vector<int> inorderTraversal(TreeNode* root) 
{
    vector<int> nodes;
    stack<TreeNode*> toVisit;
    TreeNode* curNode = root;
    while (curNode || !toVisit.empty()) 
    {
        if (curNode) 
        {
            toVisit.push(curNode);
            curNode = curNode -> left;
        }
        else 
        {
            curNode = toVisit.top();
            toVisit.pop();
            nodes.push_back(curNode -> val);    
            curNode = curNode -> right;
        }
    }
    return nodes;
}
```

### 二叉树后序遍历
问题描述：[LeetCode](https://leetcode.com/problems/binary-tree-postorder-traversal/description/)
```cpp
//递归法
vector<int> postorderTraversal(TreeNode* root) 
{
    vector<int> result;
    traverse(root, result);
    return result;
}

void traverse(TreeNode* node, vector<int>& result)
{
    if(!node)    return;
    traverse(node->left, result);
    traverse(node->right, result);
    result.push_back(node->val);
}

/**
迭代法：后序遍历是左-右-根，将其转换为根-右-左（实现方法和先序遍历根-左-右类似，只是左右互换），
最后将数组逆转即可
*/
vector<int> postorderTraversal(TreeNode* root) 
{
    vector<int> result;
    stack<TreeNode*> toVisit;
    while(root != NULL || !toVisit.empty())
    {
        while(root)
        {
            result.push_back(root->val);
            toVisit.push(root);
            root = root->right; //相对先序遍历，左右子结点顺序互换
        }
        root = toVisit.top();
        toVisit.pop();
        root = root->left;
    }
    reverse(result.begin(), result.end());
    return result;
}

//对应先序遍历的迭代法（二）
vector<int> postorderTraversal(TreeNode* root) 
{
    vector<int> result;
    if(!root)   return result;
    stack<TreeNode*> toVisit;
    toVisit.push(root);
    while(!toVisit.empty())
    {
        root = toVisit.top();
        toVisit.pop();
        result.push_back(root->val);
        if(root->left)          //注意此处要先将左子结点压入栈中
            toVisit.push(root->left);
        if(root->right)
            toVisit.push(root->right);
    }
    reverse(result.begin(), result.end());
    return result;
}

vector<int> postorderTraversal(TreeNode* root) 
{
    vector<int> result;
    if(!root)   return result;
    stack<TreeNode*> toVisit;
    TreeNode* last = NULL;
    while(root || !toVisit.empty())
    {
        if(root)
        {
            toVisit.push(root);
            root = root->left;
        }
        else
        {
            TreeNode *top = toVisit.top();
            if(top->right && last != top->right)
                root = top->right;
            else
            {
                toVisit.pop();
                result.push_back(top->val);
                last = top;
            }
        }      
    }
    return result;
}

//遍历的时候将元素插入最前面
vector<int> postorderTraversal(TreeNode* root) 
{
    vector<int> result;
    if(!root)   return result;
    stack<TreeNode*> toVisit;
    while(root || !toVisit.empty())
    {
        while(root)
        {
            toVisit.push(root);
            result.insert(result.begin(), root->val);
            root = root->right;
        }
        root = toVisit.top();
        toVisit.pop();
        root = root->left;
    }
    return result;
}
```

### 还原二叉搜索树
问题描述：给定一颗二叉搜索树，其中两个结点互换，将其还原成一颗完整的二叉搜索树。[LeetCode](https://leetcode.com/problems/recover-binary-search-tree/description/)
分析：对二叉搜索树进行中序遍历，即可得到一个有序的序列。对于异常点，存在前一个元素大于后一个元素，但两个异常点略有差异（第一个异常点，其值大于后一个元素；第二个异常点，其值小于前一个元素）
```cpp
//递归法中序遍历

TreeNode *first = NULL, *second = NULL; //如果不定义为全局变量（或类的成员变量），而是作为形参，则应该传指针变量的引用
TreeNode *prev = new TreeNode(INT_MIN);

void recoverTree(TreeNode* root) 
{
    inorderTraverse(root);
    int temp = first->val;
    first->val = second->val;
    second->val = temp;
}
    
void inorderTraverse(TreeNode* root)
{
    if(!root)   return;
    inorderTraverse(root->left);
    if(first == NULL && prev->val >= root->val)
        first = prev;
    if(first != NULL && prev->val >= root->val)
        second = root;
    prev = root;
    inorderTraverse(root->right);
}

//迭代法中序遍历
void recoverTree(TreeNode* root) 
{
    TreeNode *first = NULL, *second = NULL;
    TreeNode *prev = new TreeNode(INT_MIN);
    stack<TreeNode*> toVisit;
    while(root != NULL || !toVisit.empty())
    {
        while(root)
        {
            toVisit.push(root);
            root = root->left;
        }
        root = toVisit.top();
        toVisit.pop();
        if(first == NULL && prev->val >= root->val)
            first = prev;
        if(first != NULL && prev->val >= root->val)
            second = root;
        prev = root;
        root = root->right;
    }
    
    int temp = first->val;
    first->val = second->val;
    second->val = temp;
}
```

### 计算二叉树根结点到叶子结点的数字和
问题描述：[LeetCode](https://leetcode.com/problems/sum-root-to-leaf-numbers/description/)
Given a binary tree containing digits from 0-9 only, each root-to-leaf path could represent a number.Find the total sum of all root-to-leaf numbers. 
```cpp
/**
For example,
    1
   / \
  2   3
The root-to-leaf path 1->2 represents the number 12.
The root-to-leaf path 1->3 represents the number 13.
Return the sum = 12 + 13 = 25.
*/

//递归法
int sumNumbers(TreeNode* root) 
{
    return sum(root, 0);
}

int sum(TreeNode* node, int s)
{
    if(!node)   return 0;
    if(!node->left && !node->right)
        return s * 10 + node->val;
    return sum(node->left, s * 10 + node->val) + sum(node->right, s * 10 + node->val);
}

//迭代法
int sumNumbers(TreeNode* root) 
{
    if(!root)   return 0;
    int sum = 0;
    stack<TreeNode*> toVisit;
    toVisit.push(root);
    while(!toVisit.empty())
    {
        TreeNode* cur = toVisit.top();
        toVisit.pop();
        if(cur->right)
        {
            cur->right->val = cur->val * 10 + cur->right->val;
            toVisit.push(cur->right);
        }
        if(cur->left)
        {
            cur->left->val = cur->val * 10 + cur->left->val;
            toVisit.push(cur->left);
        }
        if(!cur->left && !cur->right)
        {
            sum += cur->val;
        }
    }
    return sum;
}
```

### 线索二叉树的构建与遍历
线索二叉树：将二叉链表中的空指针分别指向对应的前驱或后继结点
```cpp
typedef enum {Link, Thread} PointerTag;    //Link表示指向孩子结点，Thread表示指向前驱后后继线索

typedef struct BiThrNode
{
    TElemType data;
    struct BiThrNode *lchild, *rchild;
    PointerTag LTag, RTag;
}BiThrNode, *BiThrTree;

BiThrTree pre;

/**
中序遍历建立中序线索化树链表，并增加一个头结点，令其左子域指向二叉树的根结点，
右子域指向中序遍历访问的最后一个结点。反之，令中序遍历中第一个结点的左子域
和最后一个结点的右子域均指向头结点，这样既可以从前往后遍历，也可以从最后一个
结点顺前驱进行遍历
*/
Status InorderThreading(BiThrTree& Thrt, BiThrTree T)
{
    if(!Thrt = (BiThrTree)malloc(sizeof(BiThrNode)))
        exit(OVERFLOW);
    Thrt->LTag = Link;
    Thrt->RTag = Thread;
    Thrt->rchild = Thrt;    //右指针回指
    if(!T)    
        Thrt->lchild = Thrt; //若二叉树空，则左指针回指
    else
    {
        Thrt->lchild = T;
        pre = Thrt;
        InThreading(T);
        pre->rchild = Thrt;    //最后一个结点线索化
        pre->RTag = Thread；
        Thrt->rchild = pre;
    }
    return OK;
}

void InThreading(BiThrTree p)
{
    if(p)
    {
        InThreading(p->lchild);
        if(!p->lchild)
        {
            p->LTag = Thread;    //前驱线索
            p->lchild = pre;
        }
        if(!pre->rchild)
        {
            pre->RTag = Thread;    //后继线索
            pre->rchild = p;
        }
        pre = p;    //保持pre指向p的前驱
        InThreading(p->rchild);
    }
}

/**中序遍历线索二叉树**/
Status InorderTraverse_Thr(BiThrTree T)
{
    BiThrTree p;
    p = T->lchild;
    while(p != T)    //空树或遍历结束时，p == T
    {
        while(p->LTag == Link)
            p = p->lchild;
        printf("%c", p->data);
        while(p->RTag == Thread && p->rchild != T)
        {
            p = p->rchild;
            printf("%c", p->data);
        }
        p = p->rchild;
    }
    return OK;
}

```

