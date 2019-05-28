---
title: vector的reverse函数陷阱
date: 2019-05-26 19:39:04
tags: cpp
---

vector的reserve函数，只是预留了空间，直接调用vector[0]进行赋值，是不会报错的。

但是reserve的空间用vector.at(1)使用时，就会出core，因为这部分空间是未分配的。

不过如果空间已经分配了，比如说vector初始化时指定了4个空间，用at时不会出core的。

```
#include <vector>
#include <iostream>

int main() {
  std::vector<int> aaa;
  aaa.reserve(4);
  std::cout << "size = " << aaa.size() << std::endl;
	aaa[0] = 1;
  //aaa.at(1) = 2; // 这个调用会出core
  std::cout << "aaa[0] = " << aaa[0] << ", aaa[1] = " << aaa[1] << std::endl;
  std::cout << "size = " << aaa.size() << std::endl;

  std::vector<int> bbb(4);
  std::cout << "size = " << bbb.size() << std::endl;
  bbb[0] = 2;
  bbb.at(1) = 4;
  std::cout << "bbb[0] = " << bbb[0] << ", bbb[1] = " << bbb[1] << std::endl;
  std::cout << "size = " << bbb.size() << std::endl;

  return 0;
}

result:
size = 0
aaa[0] = 1, aaa[1] = -2147483648
size = 0
size = 4
bbb[0] = 2, bbb[1] = 4
size = 4
```
