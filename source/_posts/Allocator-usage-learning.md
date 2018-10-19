---
title: Allocator用法学习
date: 2018-09-17 13:19:26
tags: cpp
---
### `allocator`的作用说明
一般使用`new`和`delete`限定了内存分配和对象构造一起执行，缺少灵活性。使用`allocator`可以把二者分离。
1. 使用`allocator`对象的`allocate`函数可以分配未初始化的内存；
2. 使用`construct`进行对象初始化，注意初始化的参数第一个需要是未初始化内存的开始指针；一定注意未初始化的内存不能访问；
3. 使用`destroy`对对象进行析构；
4. 使用`deallocate`释放分配的内存；

此外，配合`uninitialized_copy`和`uninitialized_fill_n`函数，可以对未初始化内存创建对象。


### `allocator`的具体用法
```
#include <iostream>
#include <memory>
#include <string>
#include <vector>

int main() {
  std::allocator<int> a1;   // default allocator for ints
  int* a = a1.allocate(1);  // space for one int
  a1.construct(a, 7);       // construct the int
  std::cout << a[0] << '\n';
  a1.deallocate(a, 1);  // deallocate space for one int

  // default allocator for strings
  std::allocator<std::string> a2;

  // same, but obtained by rebinding from the type of a1
  decltype(a1)::rebind<std::string>::other a2_1;

  // same, but obtained by rebinding from the type of a1 via allocator_traits
  std::allocator_traits<decltype(a1)>::rebind_alloc<std::string> a2_2;

  int str_num = 10;
  std::string* s = a2.allocate(str_num);  // space for 2 strings

  std::string* s0 = s;
  a2.construct(s0, "foo");
  std::string* s1 = s + 1;
  a2.construct(s1, "bar");
  std::cout << *s0 << ' ' << *s1 << '\n';

  std::string* s2 = s1 + 1;
  std::vector<std::string> vec{"hello", "world"};

  // Copy vec to s2, avoid string assign after construct
  auto s3 = uninitialized_copy(vec.begin(), vec.end(), s2);
  // fill 6 string with "xxx"
  uninitialized_fill_n(s3, str_num - 4, "xxx");

  for (int i = 0; i < str_num; ++i) {
    std::cout << s[i] << ",";
  }
  std::cout << std::endl;

  std::string* start = s;
  for (int i = 0; i < str_num; ++i) {
    a2.destroy(start++);
  }
  a2.deallocate(s, str_num);
}

```
