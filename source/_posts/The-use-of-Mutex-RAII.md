---
title: Mutex RAII的用法
date: 2018-07-23 22:27:20
tags: cpp
---
Mutex经常会被使用到，但是直接使用lock和unlock容易出错，现代c++追求`RAII(Resource Acquisition Is Initialization)`，可以大大减少出错的概率。用法非常简单，STL中就带有。

```
#include <mutex>

std::mutex mutex_;
std::lock_guard<std::mutex> lock(mutex_);
```
