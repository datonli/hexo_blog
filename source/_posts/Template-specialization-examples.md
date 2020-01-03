---
title: 模板特化例子
date: 2020-01-03 21:45:29
tags: cpp
---


模板特化，分为函数模板特化（只能全特化）和类模板特化（可以偏特化）。

模板参数的true&false，只能用is_xx<>::value来判断输入模板参数的类型，不能判断值！

```
#include <iostream>

template <typename T>
struct DoWork {};

template <>
struct DoWork<int> {};

template <>
struct DoWork<double> {};

template <typename U>
struct DoWork<U*> {};

template<typename T>
void Hello(T h){ std::cout << "zero?" << h << std::endl;} 

template <>
void Hello(int h) { std::cout << "int " << h << std::endl;}

template<>
void Hello(double *h) { std::cout << "double * " << *h << std::endl;}

template <typename U>
void Hello(U* u) { std::cout << "U * " << *u<<std::endl;}

template <typename T, typename U>
void World(T t, U u) { std::cout << "empty" << std::endl;}

template <typename T>
void World(T t, int u) { std::cout << "int " << u << std::endl;}

template <typename T>
void World(T t, double u) { std::cout << "double " << u << std::endl;}

template <typename T, bool flag>
struct helper;

template <typename T>
struct helper<T, true> {
  void run() {
    std::cout << "true???" << std::endl;
  }
};

template <typename T>
struct helper<T, false> {
  void run() {
    std::cout << "false??" << std::endl;
  }
};

template <typename T>
void FF(T t) {
  helper<T, std::is_integral<T>::value>().run();
}

int main() {
  DoWork<int> i;
  DoWork<float*> j;

  Hello(3);
  double hh{5.0};
  Hello(&hh);

  float u{6.9};
  Hello(&u);

  int w{8};
  World(1, hh);
  World(1, w);


  FF(0);
  FF(1);
  return 0;
}

```
