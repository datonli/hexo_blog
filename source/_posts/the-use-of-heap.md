---
title: heap的使用
date: 2018-10-19 21:03:30
tags: cpp
---
最大堆、最小堆有非常好的时间复杂度和空间效率。STL提供了这个函数，可以非常方便地构建最大堆（heap，默认就是max_heap，即根节点最大）。学习下heap的使用方法。

#### 常规使用

```
std::make_heap(v.begin(), v.end());   // build a heap for vector
std::pop_heap(v.begin(), v.end());    // moves the largest to the end，即从heap中删掉最大的节点，挪到vector的最后
std::push_heap(v.begin(), v.end());   // 在vector最后存在一个未排序的节点前提下，调用push_heap将对最后元素往前面已经组成heap的结构中添加
```

#### 特殊使用
因为默认的只是max heap，并且可以支持的comparator函数比较少，往往在实际使用中需要自定义。

```
struct Greater {
  bool operator()(int a, int b) {
    return a > b;
  }
};

const Greater greater_;
make_heap(v.begin(), v.end(), greater_);
```
注意重载`operator()`，使其类可以作为类似函数的使用。

