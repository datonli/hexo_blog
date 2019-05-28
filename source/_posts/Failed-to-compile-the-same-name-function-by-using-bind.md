---
title: 使用bind函数绑定同名不同参函数编译失败
date: 2019-05-29 01:12:59
tags: cpp
---

bind函数/模板函数 绑定同名不同参函数，按照一般的ClassName::FunctionName进行bind会出现编译失败。

失败错误信息为：
```
no matching function for call to bind(<unresolved overloaded function type>
compile error ambiguous // 二义性，编译语义不明
```

源代码如下：
```
// bind_fault.cpp
#include <functional>
#include <iostream>
#include <thread>

class A {
 public:
  void doo() { std::cout << "doo" << std::endl; }
  void doo(const int x) { std::cout << "x: " << x << std::endl; }
};

template<typename Func, class Obj>
void RunFunc(Func f, Obj *obj)
{
  (obj->*f)();
}

int main() {
  A obj;
  std::thread t =
      // std::thread(std::bind(&A::doo, &obj)); // compile error <unresolved overloaded function type>
      std::thread(std::bind(static_cast<void (A::*)()>(&A::doo), &obj));
  t.join();

  //RunFunc(&A::doo, &obj); // compile error ambiguous
  RunFunc(static_cast<void (A::*)()>(&A::doo), &obj);

  return 0;
}
```

发生错误的原因是：
1. bind函数返回的std::function<> 是一个函数适配器，会抹去底层的数据类型；
2. 抹去底层数据类型的function只留下了函数名，无法区分同名不同参数的不同函数，导致编译二义；
3. 不仅仅是用function时会发生这种情况，在使用模板时，把使用函数指针作为模板
参数时也会出现类似问题，因为模板也会抹去具体的函数参数类型，导致二义。

解决办法就是：使用static_cast<>对函数指针进行指定（强制），可以避免编译失败。


