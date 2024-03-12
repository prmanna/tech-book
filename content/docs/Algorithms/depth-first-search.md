---
title: "Depth First Search"
bookCollapseSection: true
weight: 5
---

# Depth First Search
---
## Intro

The *pre-order traversal* of a tree is DFS.
```c++
    Node<int>* dfs(Node<int>* root, int target) {
        if (root == nullptr) return nullptr;
        if (root->val == target) return root;
        // return non-null return value from the recursive calls
        Node<int>* left = dfs(root->left, target);
        if (left != nullptr) return left;
        // at this point, we know left is null, and right could be null or non-null
        // we return right child's recursive call result directly because
        // - if it's non-null we should return it
        // - if it's null, then both left and right are null, we want to return null
        return dfs(root->right, target);
    }

```

## Max depth of a binary tree
Max depth of a binary tree is the longest root-to-leaf path. Given a binary tree, find its max depth. Here, we define the length of the path to be the number of edges on that path, not the number of nodes.

```c++
    int dfs(Node<int>* root) {
        // Null node adds no depth
        if (root == nullptr) return 0;
        // num nodes in longest path of current subtree = max num nodes of its two subtrees + 1 current node
        return std::max(dfs(root->left), dfs(root->right)) + 1;
    }

    int tree_max_depth(Node<int>* root) {
        return root? dfs(root) - 1 : 0;
    }
```

## Visible Tree Node | Number of Visible Nodes
In a binary tree, a node is labeled as "visible" if, on the path from the root to that node, there isn't any node with a value higher than this node's value.

The root is always "visible" since there are no other nodes between the root and itself. Given a binary tree, count the number of "visible" nodes.

```c++
    int dfs(Node<int>* root, int max_sofar) {
        if (!root) return 0;
        int total = 0;
        if (root->val >= max_sofar) total++;
        total += dfs(root->left, std::max(max_sofar, root->val));
        total += dfs(root->right, std::max(max_sofar, root->val));
        return total;
    }

    int visible_tree_node(Node<int>* root) {
        // start max_sofar with smallest number possible so any value root has is greater than it
        return dfs(root, std::numeric_limits<int>::min());
    }
```
