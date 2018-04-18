---
title: 更优雅实现enum类型映射到string的方法——重载<<运算符
date: 2018-04-18 22:03:57
tags: cpp
---
在代码中经常需要写状态机的代码，通常状态state都是用enum，而为了方便打印出一些状态信息，都需要建立state <--> msg map，使用的方法都是类似于用map结构保存，或者是建立一个全局数组，通过一个函数接口来GetStateMsg。

可以考虑重载ostream的<<运算符，非常优雅地达到这样的目的。

**注意：c++11后enum更安全的定义方法是enum class name [: type] {}，这里type就是underlying_type，通常是int，但是由编译器决定，使用underlying_type来做static_cast会是更安全的用法**

```
#include <iostream>

using namespace std;

enum class StateType {
    StatePrepare,
    StateProcess,
    StateEnd,
    StateUnknown
};


ostream& operator<<(ostream& o, const StateType& st) {
    static const char* msg[] = {
        "StatePrepare",
        "StateProcess",
        "StateEnd",
        "StateUnknown"
    };
    static uint32_t msg_size = sizeof(msg)/sizeof(const char*);
    using state_type = std::underlying_type<StateType>::type;
    uint32_t index = static_cast<state_type>(st) - static_cast<state_type>(StateType::StatePrepare);
    index = index < msg_size ? index : (msg_size - 1);
    o << msg[index];
    return o;
}

int main() {
    StateType st = StateType::StateProcess;
    cout << st << endl;
    return 0;
}
```
