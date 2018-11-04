---
title: heap的使用
date: 2018-10-19 21:03:30
tags: cpp
---
最大堆、最小堆有非常好的时间复杂度和空间效率。STL提供了这个函数，可以非常方便地构建最大堆（heap，默认就是max_heap，即根节点最大）。学习下heap的使用方法。

#### 常规使用

```
std::make_heap(v.begin(), v.end());   // build a heap for vector


// 组合用法
std::pop_heap(v.begin(), v.end());    // moves the largest to the end，即从heap中删掉最大的节点，挪到vector的最后
v.pop_back();


// 组合用法
v.push_back(a);
std::push_heap(v.begin(), v.end());   // 在vector最后存在一个未排序的节点前提下，调用push_heap将对最后元素往前面已经组成heap的结构中添加
```


#### 特殊使用
因为默认的只是max heap，并且可以支持的comparator函数比较少，往往在实际使用中需要自定义。

```
// 最小堆的实现
struct Greater {
  bool operator()(int a, int b) {
    return a > b;
  }
};

const Greater greater_;
make_heap(v.begin(), v.end(), greater_);
```
注意重载`operator()`，使其类可以作为类似函数的使用。

#### 经典面试题：找到最大的前k个数
从给定数组中找到最大的k个数（或者第k个数），经典解答就是用堆排序，但是要用的却是最小堆，因为需要先构建最小堆然后不停更新最小堆的根节点（最小值），用数组中遍历到的大于根节点的数值来替换，这样才能找到前k个最大的数。
```
#include <algorithm>
#include <iostream>
#include <vector>

using namespace std;

struct Greater {
  bool operator()(int a, int b) { return a > b; }
};

class Solution {
 public:
  vector<int> GetLargestNumbers_Solution(vector<int> input, int k) {
    if (k <= 0 || k > input.size() || input.size() == 0) return vector<int>();
    Greater greater_;
    vector<int> vs(input.begin(), input.begin() + k);
    make_heap(vs.begin(), vs.end(), greater_);
    for (int i = k; i < input.size(); ++i) {
      if (input[i] > vs[0]) {
        pop_heap(vs.begin(), vs.end(), greater_);
        vs.pop_back();

        vs.push_back(input[i]);
        push_heap(vs.begin(), vs.end(), greater_);
      }
    }
    return std::move(vs);
  }
};

int main() {
  Solution sol;
  vector<int> input{3, 5, 1, 6, 7, 2, 10};
  vector<int> vs = sol.GetLargestNumbers_Solution(input, 4);
  for (auto vi : vs) {
    cout << vi << ",";
  }
  cout << endl;
}

// 输出：5,7,6,10
```
