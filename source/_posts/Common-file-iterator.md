---
title: 常用文件迭代器
date: 2020-01-03 21:30:20
tags: cpp
---

在开发中，经常遇到需要读文件的场景，cpp的实现太多而可以直接拿来用的太少。

根据迭代器接口，写了个易用的按行读文件的实现，大家可以直接copy使用，当然如果开源要用的话需要附上我这篇博客，让我开心下，哈哈，谢谢大家了！

```
// file_iterator.h
#pragma once

#include <string>
#include <cstdio>

namespace Utils {

template <typename T>
class Iterator {
 public:
  virtual ~Iterator() {}
  virtual void Next() = 0;
  virtual bool Valid() = 0;
  virtual T Value() = 0;
};

class FileIterator : public Iterator<std::string> {
 public:
  FileIterator(const std::string& file_name);
  virtual ~FileIterator();
  void Next() override;
  bool Valid() override;
  std::string Value() override;

 private:
  std::FILE* fd;
  bool valid_;
  char rep_[1024];
};
}


// file_iterator.cpp
#include "file_iterator.h"
#include <cstring>

namespace Utils {
FileIterator::FileIterator(const std::string& file_name) {
  fd = std::fopen(file_name.c_str(), "r");
  if (fd == nullptr) {
    valid_ = false;
  } else {
    valid_ = true;
  }
  std::memset(rep_, 0, sizeof(rep_));
}

FileIterator::~FileIterator() {
  std::fclose(fd);
  valid_ = false;
}

void FileIterator::Next() {
  std::memset(rep_, 0, sizeof(rep_));
  if (nullptr == std::fgets(rep_, sizeof(rep_), fd)) {
    valid_ = false;
  }
}

bool FileIterator::Valid() { return valid_ == true; }

std::string FileIterator::Value() { return std::string(rep_); }
}


```
