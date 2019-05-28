---
title: 仿函数和匿名函数的使用
date: 2019-05-26 15:25:47
tags: cpp
---

仿函数其实就是一个类，这个类实现的是一个`operator()`函数，所以是一个仿函数的类，仿造的一个函数类。

用c的方式写出来的函数，通过函数指针传递和调用也是一样的可以实现同样功能。

在c++里面，除了仿函数，同样也可以用匿名函数来实现同样功能。

仿函数和匿名函数都可以从外面传入一些额外的数值，如果用函数指针来做同样事情需要利用上全局变量来做。

```
#include <algorithm>
#include <iostream>
#include <vector>

template <class T>
class Display {
 public:
  Display(T init_val) : val_(init_val) {}
  Display() : val_(0) {}
  ~Display() {}
  void operator()(const T &x) { std::cout << x + val_ << " "; }

 private:
  T val_;
};

int main() {
  std::vector<int> v{1, 4, 2, 6, 8};
  std::sort(v.begin(), v.end(), std::greater<int>());
  std::cout << "Display() : ";
  for_each(v.begin(), v.end(), Display<int>());
  std::cout << std::endl;

  std::cout << "Lambda : ";
  for_each(v.begin(), v.end(), [](const int &x) { std::cout << x << " "; });
  std::cout << std::endl;

  Display<int> display;
  std::cout << "for loop :";
  for (const int &x : v) {
    display(x);
  }
  std::cout << std::endl;

  Display<int> display2(2);
  std::cout << "for init val = 2 loop :";
  for (const int &x : v) {
    display2(x);
  }
  std::cout << std::endl;
  return 0;
}
```

仿函数还有些高级用法：
```
template <class Container, class DisplayBase>
static void DisplayTravel(Container *container, DisplayBase *display) {
  std::cout << "DisplayTravel : ";
  for (const auto &x : *container) {
    (*display)(x);
  }
  std::cout << std::endl;
}

int main() {
  Display<int> display3;
  DisplayTravel<std::vector<int>, Display<int>>(&v, &display3);
}
```
通过函数模板传入仿函数的基类，不同派生类可以有不同的实现，其实穿进去的DisplayBase就是一个正常的类，有所有类的功能和封装，非常方便。
