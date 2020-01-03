---
title: 自定义容器初始化列表
date: 2020-01-03 21:41:48
tags: cpp
---

其实initializer_list就是一个array包含了同类数据的类型，专门用“{}”这个中括号符号来表示，在gcc词法语法分析时会将中括号的内容转化成array，然后用initializer_list这个迭代器来遍历。

```
#include <iostream>
#include <initializer_list>
#include <vector>

template <typename V>
class MyVector
{
public :
  MyVector(std::initializer_list<V> v_list) {
    internal_v_.insert(internal_v_.begin(), v_list.begin(), v_list.end());
  }
  void Print() {
    for (auto &v : internal_v_) {
      std::cout << v << ",";
    }
    std::cout <<std::endl;
  }
private:
    std::vector<V> internal_v_;
}
;

int main() {
  MyVector<int> myv{1,3,5,6,9};
  myv.Print();
return 0;
}

```

```
/// initializer_list
  template<class _E>
    class initializer_list
    {
    public:
      typedef _E 		value_type;
      typedef const _E& 	reference;
      typedef const _E& 	const_reference;
      typedef size_t 		size_type;
      typedef const _E* 	iterator;
      typedef const _E* 	const_iterator;

    private:
      iterator			_M_array;
      size_type			_M_len;

      // The compiler can call a private constructor.
      constexpr initializer_list(const_iterator __a, size_type __l)
      : _M_array(__a), _M_len(__l) { }

    public:
      constexpr initializer_list() noexcept
      : _M_array(0), _M_len(0) { }

      // Number of elements.
      constexpr size_type
      size() const noexcept { return _M_len; }

      // First element.
      constexpr const_iterator
      begin() const noexcept { return _M_array; }

      // One past the last element.
      constexpr const_iterator
      end() const noexcept { return begin() + size(); }
    };
```
