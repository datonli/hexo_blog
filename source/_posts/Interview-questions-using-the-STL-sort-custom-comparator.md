---
title: 使用STL sort自定义comparator的面试题
date: 2018-09-16 18:10:56
tags: cpp
---

偶遇一道比较有意思的面试题，题目要求是：10w个10M大小的文件，取每个文件前32字节作比较对文件名排序，按照从小到大的顺序重新命名文件名。

想到的第一个想法是32字节分别按照4个8字节来存储，从高位到低位来对比进行比较大小，按照分桶排序的思维，更好的办法是把数据存储在vector中，利用STL的sort自定义比较器进行比较，核心代码如下。

```
#include <stddef.h>
#include <stdint.h>
#include <algorithm>
#include <array>
#include <iostream>
#include <sstream>
#include <utility>
#include <vector>

using namespace std;

int cmp(const pair<array<size_t, 4>, string>& first,
        const pair<array<size_t, 4>, string>& second) {
  for (int i = 0; i < 4; ++i) {
    if (first.first[i] == second.first[i]) {
      continue;
    } else {
      return first.first[i] > second.first[i];
    }
  }
  return 0;
}

string GetArrStr(const array<size_t, 4>& arr) {
  ostringstream oss;
  for (auto& a : arr) {
    oss << a << ",";
  }
  return oss.str();
}

int main() {
  vector<pair<array<size_t, 4>, string>> u64_file_list{{{4, 9, 11, 3}, "1"},
                                                       {{3, 5, 6, 8}, "2"},
                                                       {{3, 4, 2, 19}, "3"},
                                                       {{4, 8, 11, 19}, "4"},
                                                       {{1, 4, 2, 9}, "5"}};
  sort(u64_file_list.begin(), u64_file_list.end(), cmp);
  for (auto& u64_file : u64_file_list) {
    cout << "u64[" << GetArrStr(u64_file.first) << "] : file["
         << u64_file.second << "]" << endl;
  }
  return 0;
}
```
