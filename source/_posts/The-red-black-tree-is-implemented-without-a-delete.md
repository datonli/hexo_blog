---
title: 红黑树无delete版实现
date: 2018-05-13 20:26:10
tags: algorithm
---
第一次实现了红黑树无delete版本^_^，带delete功能的下次再post一次。

```
#include <iostream>
#include <string>
#include <vector>
#include <stack>

using namespace std;

enum class COLOR {
    RED,
    BLACK,
    UNKOWN,
};

ostream& operator<<(ostream& o, const COLOR& c) {
    static const char* msg[] = {
        "RED",
        "BLACK",
        "UNKOWN",
    };
    static uint32_t msg_size = sizeof(msg)/sizeof(const char*);
    using color_type = std::underlying_type<COLOR>::type;
    uint32_t index = static_cast<color_type>(c) - static_cast<color_type>(COLOR::RED);
    o << msg[index];
    return o;
}

class RedBlackTree {
private:
    struct Node {
        int val;
        Node* left;
        Node* right;
        Node* p;
        COLOR color;
        Node() {}
        Node(int value) : val(value),
                        left(NULL),
                        right(NULL),
                        p(NULL),
                        color(COLOR::UNKOWN) {}
    };

public:
    RedBlackTree(const vector<int>& arr) : root(NULL), nil(new Node()) {
        root = nil;
        for (auto &a : arr) {
            RootInsertNode(a);
        }
    }
    void InsertNode(int value) {
        RootInsertNode(value);
    }
    void Remove(int value) {

    }
    void TreeWalk() {
        InorderTreeWalk(root);
        cout << endl;
    }
    bool IsExist(int value) {
        return FindNode(value) != nil;
    }
    int TreeHeight() {
        if (root == nil) {
            return 0;
        } else {
            return max(MaxHeight(root->left), MaxHeight(root->right)) + 1;
        }
    }
    void PrintLevelNodes() {
        cout << "begin PrintLevelNodes" << endl;
        level_nodes.clear();
        int height = TreeHeight();
        level_nodes.resize(height);
        stack<Node*> sta_tmp;
        stack<Node*> sta_main;
        sta_tmp.push(root);
        for (int cnt = 0 ; cnt < height; ++cnt) {
            while (!sta_tmp.empty()) {
                level_nodes[cnt].push_back(sta_tmp.top());
                if (sta_tmp.top()->left != nil) {
                    sta_main.push(sta_tmp.top()->left);
                }
                if (sta_tmp.top()->right != nil) {
                    sta_main.push(sta_tmp.top()->right);
                }
                sta_tmp.pop();
            }
            while (!sta_main.empty()) {
                // sta_main => sta_tmp
                sta_tmp.push(sta_main.top());
                sta_main.pop();
            }
        }

        for (int cnt = 0; cnt < height; ++cnt) {
            cout << "level " << cnt + 1 << " : ";
            for (auto& n : level_nodes[cnt]) {
                cout << "[" << n->val << ", " << n->color << ", " << (n->left != nil) << ", " << (n->right != nil) << "], ";
            }
            cout << endl;
        }
        cout << "end PrintLevelNodes" << endl;
        cout << endl;
    }
private:
    void RootInsertNode(int value) {
        Node* new_node = new Node(value);
        new_node->color = COLOR::RED;
        new_node->left = new_node->right = nil;
        if (root == nil) {
            new_node->p = nil;
            root = new_node;
            root->color = COLOR::BLACK;
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

        // same as binary_search_tree above
        RBInsertFixup(new_node);
        PrintLevelNodes();
    }

    void RBInsertFixup(Node* z) {
        if (z == nil) {
            cout << "wrong input in RBInsertFixup" << endl;
        }
        while (z != nil && z->p != nil && z->p->color == COLOR::RED) {
            if (z->p == z->p->p->left) {
                Node* y = z->p->p->right;
                if (y->color == COLOR::RED) {
                    z->p->color = COLOR::BLACK;
                    y->color = COLOR::BLACK;
                    z->p->p->color = COLOR::RED;
                    z = z->p->p;
                } else if (z == z->p->right) {
                    z = z->p;
                    LeftRotate(z);
                } else {
                    z->p->color = COLOR::BLACK;
                    z->p->p->color = COLOR::RED;
                    RightRotate(z->p->p);
                }
            } else {
                Node* y = z->p->p->left;
                if (y->color == COLOR::RED) {
                    z->p->color = COLOR::BLACK;
                    y->color = COLOR::BLACK;
                    z->p->p->color = COLOR::RED;
                    z = z->p->p;
                } else if (z == z->p->right) {
                    z = z->p;
                    LeftRotate(z);
                } else {
                    z->p->color = COLOR::BLACK;
                    z->p->p->color = COLOR::RED;
                    RightRotate(z->p->p);
                }
            }
        }
        root->color = COLOR::BLACK;
    }

    bool LeftRotate(Node* x) {
        if (x == nil || x->right == nil) {
            cout << "left rotate wrong node" << endl;
            return false;
        }    
        Node* y = x->right;
        x->right = y->left;
        if (y->left != nil) {
            y->left->p = x;
        }
        y->p = x->p;
        if (x->p == nil) {
            root = y;
        } else {
            x->p->left == x ? x->p->left = y : x->p->right = y;
        }
        y->left = x;
        x->p = y;
        return true;
    }
    bool RightRotate(Node* y) {
        if (y == nil || y->left == nil) {
            cout << "right rotate wrong node" << endl;
            return false;
        }
        Node* x = y->left;
        y->left = x->right;
        if (x->right != nil) {
            x->right->p = y;
        }
        x->p = y->p;
        if (y->p == nil) {
            root = x;
        } else {
            y->p->left == y ? y->p->left = x : y->p->right = x;
        }
        y->p = x;
        x->right = y;
        return true;
    }
    int MaxHeight(Node* node) {
        if (node == nil) {
            return 0;
        }
        return max(MaxHeight(node->left), MaxHeight(node->right)) + 1;
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
        cout << "[" << node->val << ", " << node->color << "], ";
        InorderTreeWalk(node->right);
    }

private:
    Node* root;
    Node* nil;
    vector<vector<Node*>> level_nodes;
};

int main() {
    vector<int> arr{11, 2, 14, 1, 7, 15, 5, 8, 4};
    RedBlackTree rb_tree(arr);
    rb_tree.InsertNode(6);
    rb_tree.TreeWalk();
    cout << "tree height = " << rb_tree.TreeHeight() << endl;
    rb_tree.PrintLevelNodes();
    return 0;
}
```