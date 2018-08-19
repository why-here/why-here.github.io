### 1 二叉树遍历

#### 1.1 前序/中序/后序遍历

[LeetCode 144 Preorder](https://leetcode.com/problems/binary-tree-preorder-traversal/description/) / [LeetCode 94 Inorder](https://leetcode.com/problems/binary-tree-inorder-traversal/description/) / [LeetCode 145 Postorder](https://leetcode.com/problems/binary-tree-postorder-traversal/description/)

##### 1.1.1 递归思路

用 vector 的引用存结果，若非空，按照前序/中序/后序，将当前节点存入 vector 的操作分别放在第 1 / 2 / 3 步。

```c++
// 前序
void preOrder(TreeNode *root, vector<int> &res) {
    if(root == NULL)
        return;
    res.push_back(root->val);  // 第 1 步
    preOrder(root->left, res);
    preOrder(root->right, res);
}
// 中序
void inOrder(TreeNode *root, vector<int> &res) {
    if(root == NULL)
        return;
    inOrder(root->left, res);
    res.push_back(root->val);  // 第 2 步
    inOrder(root->right, res);
}
// 后序
void postOrder(TreeNode *root, vector<int> &res) {
    if(root == NULL)
        return;
    postOrder(root->left, res);
    postOrder(root->right, res);
    res.push_back(root->val);  // 第 3 步
}
```

##### 1.1.2 非递归思路

都是通过 vector 存储结果，并使用栈辅助遍历。可以想象递归的过程来理解非递归的遍历流程。

前序：将 root 入栈，栈非空，取出栈顶元素，存入 vector 尾部，并先将右子树入栈然后左子树入栈。

中序：将 root 入栈，取得左子树，不断将左子树入栈，然后取出栈顶元素，即为中序的元素，存入 vector 尾部，以同样方式处理当前节点的右子树。

后序：将 root 入栈，栈非空，取出栈顶元素，存入 vector 头部（模拟一个栈），然后先将左子树入栈，然后右子树入栈（因为是以“栈”的方式存储结果）。

```c++
// 前序
vector<int> preOrder(TreeNode* root) {
    vector<int> res;
    if(root == NULL)
        return res;
    stack<TreeNode*> s;
    s.push(root);
    while(!s.empty()) {
        TreeNode *n = s.top();
        s.pop();
        res.push_back(n->val);
        if(n->right) s.push(n->right); // 右子树先入栈，因为需要先遍历左子树
        if(n->left) s.push(n->left);
    }
    return res;
}
// 中序
vector<int> inOrder(TreeNode* root) {
    vector<int> res;
    if(root == NULL)
        return res;
    stack<TreeNode*> s;
    s.push(root);
    TreeNode * curr = root->left;
    while(!s.empty()) {
        while(curr) {
            s.push(curr);
            curr = curr->left;
        }
        curr = s.top();  // 取得“中序”元素，左子树也以“中序”的方式存入 vector 中
        res.push_back(curr->val);  
        s.pop();
        curr = curr->right;       // 以相同的方式处理右子树
    }
    return res;
}
// 后序
vector<int> postOrder(TreeNode* root) {
    vector<int> res;
    if(root == NULL)
        return res;
    stack<TreeNode*> s;
    s.push(root);        
    while(!s.empty()) {
        TreeNode *curr = s.top();
        s.pop();
        res.insert(res.begin(), curr->val); // 模拟栈
        if(curr->left) s.push(curr->left);  // 先将左子树入栈，与前序不同。
        if(curr->right) s.push(curr->right);
    }
    return res;
}
```

#### 1.2 层次遍历

[LeetCode 102 Top to Bottom](https://leetcode.com/problems/binary-tree-level-order-traversal/description/) / [LeetCode 107 Bottom to Top](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/description/) / [LeecCode 103 Zigzag](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/submissions/1)

##### 1.2.1 自顶向下 / 自底向上

递归思路：二维数组存储层次遍历结果。传递一个整数指明当前深度（层次）。将当前节点值存入对应层的 vector 尾部，递归遍历左/右子树。

非递归思路：使用队列辅助遍历。将 root 如队列，队列非空，记录当前队列长度，遍历整个队列元素，存入 vector 中，并将其左/右子树分别如队列。

自底向上的递归可以通过控制 level 以及插入新 vector 的位置实现。或者直接将自顶向下的结果 reverse 即可。

```c++
// 递归（自顶向下）
void levelOrder(TreeNode* root, int level, vector<vector<int>> &res) {
    if(root == NULL)
        return;
    if (level == res.size())
        res.push_back(vector<int>());
    res[level].push_back(root->val);
    levelOrder(root->left, level+1, res);
    levelOrder(root->right, level+1, res);
}
// 非递归（自顶向下）
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int cnt = q.size();
        vector<int> vals;
        for (int i = 0; i < cnt; ++i) {
            TreeNode *cur = q.front();
            vals.push_back(cur->val);
            q.pop();
            if (cur->left) q.push(cur->left);
            if (cur->right) q.push(cur->right);
        }
        res.push_back(vals);
    }
    return res;
}
```

##### 1.2.2 之字型遍历

思路：与普通层次遍历一致，额外以一个整型记录当前深度（层次），来决定是将当前层的数存入 vector 头部还是尾部，实现之字型遍历。

```c++
// 递归
void zigzag(TreeNode* root, int level, vector<vector<int>> &res) {
    if(root == NULL) return;
    if(level == res.size())
        res.push_back(vector<int>());
    if(level & 1)
        res[level].insert(res[level].begin(), root->val);
    else
        res[level].push_back(root->val);
    zigzag(root->left, level + 1, res);
    zigzag(root->right, level + 1, res);
}
```

### 1.3 侧视图

递归思路：修改先序遍历，传入 level 当前层次，当 vector 大小小于 level 时，存入当前数值，并按序递归右/左子树（右侧试图顺序，若是左侧试图则反过来）。

非递归思路：修改层次遍历，记录每一层的节点数，并判断是否是该层最右边节点（或最左边节点），然后将其右/左子树分别如队列（左侧试图相反）。

```c++
// 递归（右侧视图）level 初始值为 1
void rightSideView(TreeNode *root, int level, vector<int> &res)
{
    if(root == NULL) return;
    if(res.size() < level) res.push_back(root->val);
    rightSideView(root->right, level+1, res);
    rightSideView(root->left, level+1, res);
}
// 非递归（右侧视图）
vector<int> rightSideView(TreeNode* root) {
    vector<int> res;
    if(root == NULL)
        return res;
    queue<TreeNode*> q;
    q.push(root);
    while(!q.empty()) {
        int n = q.size();
        for(int i = 0; i < n; i++){
            TreeNode *node = q.front();
            q.pop();
            if(i == 0)
                res.push_back(node->val);
            if(node->right) q.push(node->right);
            if(node->left) q.push(node->left);
        }
    }
    return res;
}
```

