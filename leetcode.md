# 力扣刷题笔记

## 二叉树

[113. 路径总和 II ](https://leetcode.cn/problems/path-sum-ii/)

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

