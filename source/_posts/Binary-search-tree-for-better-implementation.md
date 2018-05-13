---
title: 二叉搜索树更好的实现
date: 2018-05-13 14:40:32
tags: algorithm
---
在前一个“二叉搜索树”博客基础上，增加nil节点作为哨兵节点，优化find和前继节点、后继节点的接口。
```
#include <iostream>
#include <string>
#include <vector>

using namespace std;

class BinarySearchTree {
private:
    struct Node {
        int val;
        Node* left;
        Node* right;
        Node* p;
        Node() {}
        Node(int value) : val(value),
                          left(NULL),
                          right(NULL),
                          p(NULL) {}
    };
public:
    BinarySearchTree(const vector<int>& arr) : root(NULL), nil(new Node()) {
        root = nil;
        for (auto &a : arr) {
            RootInsertNode(a);
        }
    }
    void InsertNode(int value) {
        RootInsertNode(value);
    }
    bool IsExist(int value) {
        return FindNode(value) != nil;
    }
    void Remove(int value) {
        Node* node = FindNode(value);
        if (node == nil) {
            cout << "node dont exist!" << endl;
            return;
        }
        Node* pnode = node->p;
        if (pnode == nil) { // root
            Node* success_node = SuccessNode(node);
            if (nil == success_node) {
                root = node->left;
                node->left->p = nil;
            } else {
                if (success_node->p != node) { // success node's parent not the delete node
                                               // otherwise, success node' right subtree don't need move
                    success_node->p->left = success_node->right;
                    success_node->right->p = success_node->p;
                    success_node->right = node->right;
                }
                success_node->p = pnode;
                success_node->left = node->left;
                root = success_node;
            }
        } else {
            if (node->left == nil && node->right == nil) {
                pnode->left == node ? pnode->left = nil : pnode->right = nil;
            } else if (node->left == nil && node->right != nil) {
                pnode->left == node ? pnode->left = node->right : pnode->right = node->right;
                node->right->p = pnode;
            } else if (node->left != nil && node->right == nil) {
                pnode->left == node ? pnode->left = node->left : pnode->right = node->left;
                node->left->p = pnode;
            } else {
                Node* success_node = SuccessNode(node);
                if (success_node->p != node) { // success node's parent not the delete node
                                               // otherwise, success node' right subtree don't need move
                    success_node->p->left = success_node->right;
                    success_node->right->p = success_node->p;
                    success_node->right = node->right;
                }
                success_node->p = pnode;
                success_node->left = node->left;
                pnode->left == node ? pnode->left = success_node : pnode->right = success_node;
            }
        }
        delete node;
    }
    void TreeWalk() {
        InorderTreeWalk(root);
        cout << endl;
    }
private:
    Node* SuccessNode(Node* node) {
        if (node == nil || node->right == nil) {
            return nil;
        }
        node = node->right;
        while(node->left != nil) {
            node = node->left;
        }
        return node;
    }
    Node* PreNode(Node* node) {
        if (node == nil || node->left == nil) {
            return nil;
        }
        node = node->left;
        while(node->right != nil) {
            node = node->right;
        }
        return node;
    }
    Node* FindNode(int value) {
        Node* node = root;
        while(node != nil) {
            if (value == node->val) {
                return node;
            } else if (value > node->val) {
                node = node->right;
            } else {
                node = node->left;
            }
        }   
        return nil;
    }
    void InorderTreeWalk(Node* node) {
        if (node == nil) return;
        InorderTreeWalk(node->left);
        cout << node->val << ", ";
        InorderTreeWalk(node->right);
    }
    void RootInsertNode(int value) {
        Node* new_node = new Node(value);
        new_node->left = new_node->right = nil;
        if (root == nil) {
            new_node->p = nil;
            root = new_node;
            return;
        }
        Node* node = root;
        Node* pre = root->p;
        bool is_left = false;
        while(node != nil) {
            if (value > node->val) {
                is_left = false;
                pre = node;
                node = node->right;
            } else {
                is_left = true;
                pre = node;
                node = node->left;
            }
        }
        new_node->p = pre;
        is_left ? pre->left = new_node : pre->right = new_node;
    }
private:
    Node* root;
    Node* nil;
};

int main() {
    vector<int> arr{8, 1, 16, 14, 7, 9, 4, 10, 3, 2};
    BinarySearchTree binary_tree(arr);
    binary_tree.InsertNode(15);
    binary_tree.TreeWalk();
    binary_tree.Remove(7);
    binary_tree.TreeWalk();
    binary_tree.Remove(16);
    binary_tree.TreeWalk();
    binary_tree.Remove(8);
    binary_tree.TreeWalk();
    return 0;
}
```