---
title: 纯虚函数模板的使用
date: 2019-06-02 13:56:53
tags: cpp
---

纯虚函数/虚函数的模板，结合了模板和虚函数的灵活性，在某些场景下可能会需要。

模板其实是在编译期展开，所以在编译之后其实得到的还是一个带虚函数的类，既不违背模板的编译期确定，也不违背虚函数的运行期确定。

```
#include <functional>
#include <iostream>
#include <vector>

template <typename T>
class Processor {
 public:
  Processor() {}
  virtual ~Processor() {}

  virtual T Process(const std::vector<T>&) = 0;
};

template <typename Func>
class ProcessorImpl : public Processor<uint64_t> {
 public:
  ProcessorImpl(Func func) : func_(func) {}
  ~ProcessorImpl() {}

  uint64_t Process(const std::vector<uint64_t>& process_list) {
    return func_(process_list);
  }

 private:
  Func func_;
};

int main() {
  auto AddFunc = [](const std::vector<uint64_t>& process_list) -> uint64_t {
    uint64_t result = 0;
    for (uint64_t num : process_list) {
      result += num;
    }
    return result;
  };
  ProcessorImpl<decltype(AddFunc)> processor_impl(AddFunc);
  std::cout << "sum(3,4,5) = "
            << processor_impl.Process(std::vector<uint64_t>{3, 4, 5})
            << std::endl;

  auto MultiplyFunc =
      [](const std::vector<uint64_t>& process_list) -> uint64_t {
    uint64_t result = 1;
    for (uint64_t num : process_list) {
      result *= num;
    }
    return result;
  };
  ProcessorImpl<decltype(MultiplyFunc)> processor_impl2(MultiplyFunc);
  std::cout << "multiply(3,4,5) = "
            << processor_impl.Process(std::vector<uint64_t>{3, 4, 5})
            << std::endl;

  return 0;
}
```

结果：
```
sum(3,4,5) = 12
multiply(3,4,5) = 12
```