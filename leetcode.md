# 力扣刷题笔记

## 二叉树

### [113. 路径总和 II ](https://leetcode.cn/problems/path-sum-ii/)

给你二叉树的根节点 `root` 和一个整数目标和 `targetSum` ，找出所有 **从根节点到叶子节点** 路径总和等于给定目标和的路径。

**解**

- 从根结点出发到达叶子节点(没有左右子树就是叶子节点)
- 递归前进状态：将当前节点都需要添加到本次的路径上temp
- 递归回退状态：退出当前的路径，就需要从本次路径temp上移出当前节点
- 时间复杂度O(n), 空间复杂度O(n)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:

    vector<vector<int>> ret;  // 保存路径
    vector<int> temp;  // 保存路径上的节点

    /* 前序遍历从根节点找到所有到达叶子节点的路径，并找到符合条件的路径 */
    void preorder(TreeNode* root, int targetSum)
    {
        if (!root)
            return;

        temp.push_back(root->val);  // 将当前节点放到本次遍历的路径上

        /* 找到符合结果的路径 */
        if (root->val == targetSum && !root->left && !root->right)      
            ret.push_back(temp);
        
        preorder(root->left, targetSum - root->val);
        preorder(root->right, targetSum - root->val);
        temp.pop_back();  // 回溯时需要将temp中的最后添加的元素移出
        

    }

    vector<vector<int>> pathSum(TreeNode* root, int targetSum) 
    {
        preorder(root, targetSum);
        return ret;
    }
};
```

### [99. 恢复二叉搜索树](https://leetcode.cn/problems/recover-binary-search-tree/)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:

    /*
     * 确定遍历方式：中序遍历
     * 正常放入节点
     * 一边记录上一次弹出的节点（last），每次弹出栈顶后需要比较一下 
     * 找到两个节点就完成任务（one, two）
     */

    void recoverTree(TreeNode* root) 
    {
        stack<TreeNode*> stk;
        TreeNode* one = nullptr;
        TreeNode* two = nullptr;
        TreeNode* last = nullptr;
        while (stk.size() || root)
        {
            while (root)
            {
                /* 正常放入节点 */
                stk.push(root);
                root = root->left;
            }
            root = stk.top();
            stk.pop();
            /* 弹出节点需要记录弹出的最后一个节点 */
            if (last != nullptr && root->val < last->val)
            {
                one = root;
                if (two == nullptr) 
                    two = last;
                else   
                    break;
            }
            last = root;
            root = root->right;
        }

        swap(one->val, two->val);
    }
};
```



### [101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/submissions/)

```cpp

class Solution {
public:
    /*
     * 二叉树对称条件：每棵树的左子树与右子树是镜像的
     */

    /* 这个环节是 && 说明有一部分出错，就能说明非对称 */
    bool check(TreeNode* l, TreeNode* r)
    {
        /* 如果左右子树同时为空，说明遍历到叶子节点上也没有出现非对称的情况 */
        if (!l && !r) return true; 
        /* 左右子树有一个为空，返回false */
        if (!l || !r) return false;
        return (l->val == r->val) && check(l->right, r->left) && check(l->left, r->right);
    }
    
    bool isSymmetric(TreeNode* root)
    {
        return check(root, root);
    }
};
```



[102. 二叉树的层序遍历 ](https://leetcode.cn/problems/binary-tree-level-order-traversal/submissions/)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root)
    {
        /* 常规判空 */
        if (!root) return {};
        vector<vector<int>> ret;  // 结果vector
        vector<int> floor;        // 保存二叉树的每一层，向ret中添加后清空
        queue<TreeNode*> que;     // 广度遍历保存节点的单向队列
        int floorCount;           // 记录二叉树每一层节点的个数 
        TreeNode* tFront;         // 队列的头,预分配空间
        que.push(root);
        while (que.size())
        {
            floorCount = que.size();  
            /* 当前层有几个节点就遍历几次，每次添加当前节点的左右子树 */
            while (floorCount--)
            {
                tFront = que.front();
                que.pop();
                floor.push_back(tFront->val);
                /* 如果遍历到叶子节点不能向队列中添加空节点 */
                if (tFront->left) que.push(tFront->left);
                if (tFront->right) que.push(tFront->right);
            }
            /* 添加每一层的结果 */
            ret.push_back(floor);
            floor.clear();
        }
        return ret;
    }
};
```

