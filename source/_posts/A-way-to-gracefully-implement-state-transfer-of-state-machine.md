---
title: 优雅地实现状态机状态转移的方法
date: 2018-04-18 22:02:42
tags: cpp
---
状态机转移，使用map+function+bind的方式可以实现得很优雅。

```
// state_machine.h
#include <map>
#include <functional>

enum class StateType {
    StatePrepare,
    StateProcess,
    StateEnd,
    StateUnknown
};

class StateMachProc {
public:
    StateMachProc() : state_type_(StateType::StatePrepare) {}
    virtual ~StateMachProc() {}
    bool SetNextState(const StateType& st);

private:
    void StateProcessHandler(const StateType& st);
    void StateEndHandler(const StateType& st);

    StateType state_type_;
    typedef std::function<void (StateMachProc*, const StateType&)> StateTypeHandler;
    static std::map<StateType, StateMachProc::StateTypeHandler> state_type_handler_map_;
};
```

```
// state_machine.cpp
#include "state_machine.h"
#include <iostream>

std::map<StateType, StateMachProc::StateTypeHandler>
StateMachProc::state_type_handler_map_ = {
    {StateType::StateProcess,
     std::bind(&StateMachProc::StateProcessHandler,
             std::placeholders::_1, std::placeholders::_2)},
    {StateType::StateEnd,
     std::bind(&StateMachProc::StateEndHandler,
             std::placeholders::_1, std::placeholders::_2)},
};

void StateMachProc::StateProcessHandler(const StateType& st) {
    if (st == StateType::StatePrepare) {
        std::cout << "good from prepare to process" << std::endl;
        state_type_ = StateType::StateProcess;
    }
}

void StateMachProc::StateEndHandler(const StateType& st) {
    if (st == StateType::StateProcess) {
        std::cout << "good from process to end" << std::endl;
        state_type_ = StateType::StateEnd;
    }
}

bool StateMachProc::SetNextState(const StateType& st) {
    auto func = state_type_handler_map_[st];
    func(this, state_type_);
    return true;
}
```
这里面Handler处理用的是同步，还可以改成异步处理，使用条件变量通知；
另外可以考虑增加TransationLock，防止多线程调用使得状态混乱。
```
// main.cpp
// g++ main.cpp state_machine.cpp -I./ -std=c++11 -o main
#include "state_machine.h"

int main() {
    StateMachProc smp;
    smp.SetNextState(StateType::StateProcess);
    smp.SetNextState(StateType::StateEnd);
    return 0;
}
```