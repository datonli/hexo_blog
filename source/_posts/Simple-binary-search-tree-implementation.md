---
title: 简单二叉搜索树实现
date: 2018-05-01 02:45:38
tags: algorithm
---
实现个二叉搜索树。唯一需要注意的是：在Insert的时候注意传参进去传的是形参，需要获取返回值Node*来更新指针。

比较麻烦的是删除操作的时候需要区分几种不同节点分布的情况，具体看算法导论P167。

```
#include "common.h"

class BinarySearchTree {
private:
    struct Node {
        int val;
        Node* left;
        Node* right;
        Node(int value) : val(value), left(NULL), right(NULL) {}
    };
public:
    BinarySearchTree(const vector<int>& arr) : root(NULL) {
        for (auto &a : arr) {
            root = InnerInsertNode(root, a);
        }
    }

    void InsertNode(int value) {
        root = InnerInsertNode(root, value);
    }

    bool FindNode(int value) {
        Node* node = root;
        while (node != NULL) {
            if (value > node->val) {
                node = node->right;
            } else if (value < node->val) {
                node = node->left;
            } else {
                return true;
            }
        }
        return false;
    }

    bool RemoveNode(int value) {
        bool is_existed = false;
        Node* node = root;
        Node* node_parent = NULL;
        bool is_left = false;
        while (node != NULL) {
            if (value > node->val) {
                node_parent = node;
                node = node->right;
                is_left = false;
            } else if (value < node->val) {
                node_parent = node;
                node = node->left;
                is_left = true;
            } else {
                is_existed = true;
                break;
            }
        }
        if (!is_existed) {
            return false;
        }
        if (node_parent == NULL) {
            // remove root
            if (node->right == NULL) {
                root = node->left;
            } else {
                Node* successor_parent = node;
                Node* successor = node->right;
                if (successor->left == NULL) {
                    successor->left = node->left;
                } else {
                    while(successor->left != NULL) {
                        successor_parent = successor;
                        successor = successor_parent->left;
                    }
                    successor_parent->left = successor->right;
                    successor->left = node->left;
                    successor->right = node->right;
                }
                root = successor;
            }
        } else {
            if (IsLeaf(node)) {
                is_left == true ? node_parent->left = NULL : node_parent->right = NULL;
            } else if (!IsLeaf(node) && (node->left != NULL && node->right == NULL)) {
                Node* new_node = node->left;
                is_left == true ? node_parent->left = new_node : node_parent->right = new_node;
            } else if (!IsLeaf(node) && (node->left == NULL && node->right != NULL)) {
                Node* new_node = node->right;
                is_left == true ? node_parent->left = new_node : node_parent->right = new_node;
            } else {
                // two children
                // successor parent
                Node* successor_parent = node;
                Node* successor = node->right;
                if (successor->left == NULL) {
                    is_left == true ? node_parent->left = successor : node_parent->right = successor;
                    successor->left = node->left;
                } else {
                    while(successor->left != NULL) {
                        successor_parent = successor;
                        successor = successor_parent->left;
                    }
                    successor_parent->left = successor->right;
                    is_left == true ? node_parent->left = successor : node_parent->right = successor;
                    successor->left = node->left;
                    successor->right = node->right;
                }
            }
        }
        delete node;
        node = NULL;
        return true;
    }

    void TreeWalk() {
        InorderTreeWalk(root);
        cout << endl;
    }

    int MaxValue() {
        Node* node = root;
        int max_value = 0;
        while (node != NULL) {
            max_value = node->val;
            node = node->right;
        }
        return max_value;
    }

    int MinValue() {
        Node* node = root;
        int min_value = 0;
        while (node != NULL) {
            min_value = node->val;
            node = node->right;
        }
        return min_value;
    }

private:
    bool IsLeaf(Node* node) {
        return (node->left == NULL) && (node->right == NULL);
    }
    Node* InnerInsertNode(Node* node, int value) {
        if (node == NULL) {
            node = new Node(value);
        } else {
            if (value < node->val) {
                node->left = InnerInsertNode(node->left, value);
            } else {
                node->right = InnerInsertNode(node->right, value);
            }
        }
        return node;
    }

    void InorderTreeWalk(Node* node) {
        if (node == NULL) return;
        InorderTreeWalk(node->left);
        cout << node->val << ", ";
        InorderTreeWalk(node->right);
    }
    Node* root;
};

int main() {
    vector<int> arr{8, 1, 16, 14, 7, 9, 4, 10, 3, 2};
    BinarySearchTree binary_tree(arr);
    binary_tree.TreeWalk();
    binary_tree.InsertNode(43);
    binary_tree.InsertNode(22);
    binary_tree.TreeWalk();
    binary_tree.RemoveNode(16);
    binary_tree.TreeWalk();
    binary_tree.RemoveNode(8);
    binary_tree.TreeWalk();
    return 0;    
}
```