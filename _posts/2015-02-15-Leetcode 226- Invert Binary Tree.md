---
layout: post_layout
title: Leetcode反转二叉树
time: 2015年2月15日
location: 长春
pulished: true
excerpt_separator: "##"
---


## 反转二叉树
Invert a binary tree.

![图片1](http://7xlv11.com1.z0.glb.clouddn.com/20151021154733.png)


to:
![图片](http://7xlv11.com1.z0.glb.clouddn.com/20151021154752.png)

### 递归方法
这道题可以这样细化：反转二叉树的左右子树，然后将左右子树交换。所以用递归的思想是很自然的：
```c++
//author:huxiangming
//email:husama@qq.com
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if (root ==NULL)
            return NULL;
        TreeNode * temp =root->left;
        root->left =invertBinaryTree(root->right);
        root->right =invertBinaryTree(temp);
        return root;
    }
};
```

### 非递归方法：
用栈和队列都能模拟，思路差不多，就只放出用栈实现的。
```c++
//author:huxiangming
//email:husama@qq.com
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
         if (root ==NULL)
            return NULL;
        stack<TreeNode *> S;
        S.push(root);
        TreeNode *p;
        while (!S.empty()) {
            p=S.top();
            S.pop();
            TreeNode *pNode =p->left;
            p->left =p->right;
            p->right =pNode;
            if (p->left !=NULL)
                S.push(p->left);
            if (p->right !=NULL)
                S.push(p->right);
        }
        return root;
    }
};
```

很明显递归的算法要简洁易懂许多，但是效率不高，容易栈溢出，所以迭代和递归都要掌握。
