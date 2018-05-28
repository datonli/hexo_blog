---
title: c++11线程安全单例写法
date: 2018-05-28 21:43:18
tags: cpp
---

c++11规定`局部静态变量初始化线程安全`。所以单例可以有更好的写法，坚决抛弃double check中间还加锁的做法。

注意：要局部静态变量初始化，不能`new`，因此，单例返回的一定是这个对象的引用。

```
class CSingleton {
public:
    ~CSingleton() {}
    static CSingleton& getInstance() {
        static CSingleton m_instance;
        return m_instance;
    }
};
```
