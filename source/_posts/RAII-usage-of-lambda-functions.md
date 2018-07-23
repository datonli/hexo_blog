---
title: lambda函数的RAII用法
date: 2018-07-23 22:31:29
tags: cpp
---

有很多时候操作是成对出现的，或者是有些操作是希望在函数return的时候调用的，以往的用法需要每个return都增加上对应的重复代码，当然也可以用goto（其实更坑）。

可以使用lambda+func_scope_guard实现RAII，用法很简单，只需要增加一个class就行。

```
#include <functional>
namespace common {

class FuncScopeGuard {
public:
    explicit FuncScopeGuard(std::function<void()> on_exit_scope)
        : on_exit_scope_(on_exit_scope), dismissed_(false) {}

    FuncScopeGuard(FuncScopeGuard const&) = delete;
    FuncScopeGuard& operator=(const FuncScopeGuard&) = delete;

    virtual ~FuncScopeGuard() {
        if(!dismissed_) {
            on_exit_scope_();
        }
    }

    void Dismiss() {
        dismissed_ = true;
    }

private:
    std::function<void()> on_exit_scope_;
    bool dismissed_;
};

}
using common::FuncScopeGuard;
```
