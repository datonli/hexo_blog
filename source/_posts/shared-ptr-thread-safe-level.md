---
title: shared_ptr的安全级别
date: 2018-05-10 20:06:41
tags: cpp
---

这是一个很容易忘记的问题，使用shared_ptr并不意味着就一定是安全的。因为shared_ptr的结构（g++的实现见：`include/bits/shared_ptr_base.h`）为：
```
    class __shared_ptr
    : public __shared_ptr_access<_Tp, _Lp>
    {
    public:
        ...
    private:
      element_type*	   _M_ptr;         // Contained pointer.
      __shared_count<_Lp>  _M_refcount;    // Reference counter.
    };
```

引用计数是线程安全的，但是__shared_ptr本身却不是线程安全的。对于shared_ptr本身的读写不能原子化。

介绍文档：`https://www.boost.org/doc/libs/1_48_0/libs/smart_ptr/shared_ptr.htm#example`

根据boost中的介绍，shared_ptr的线程安全级别和内建类型、标准库容器、std::string一样，即：
1. 一个shared_ptr对象实体可被多个线程同时读取（例1）；
2. 两个shared_ptr对象实体可以被两个线程同时写入（例2），“析构”算写操作；
3. 如果要从多个线程读写同一个shared_ptr对象实体，需要加锁（例3~5）。

具体例子罗列如下：

```
shared_ptr<int> p(new int(42));

//--- Example 1 ---

// thread A
shared_ptr<int> p2(p); // reads p

// thread B
shared_ptr<int> p3(p); // OK, multiple reads are safe

//--- Example 2 ---

// thread A
p.reset(new int(1912)); // writes p

// thread B
p2.reset(); // OK, writes p2

//--- Example 3 ---

// thread A
p = p3; // reads p3, writes p

// thread B
p3.reset(); // writes p3; undefined, simultaneous read/write

//--- Example 4 ---

// thread A
p3 = p2; // reads p2, writes p3

// thread B
// p2 goes out of scope: undefined, the destructor is considered a "write access"

//--- Example 5 ---

// thread A
p3.reset(new int(1));

// thread B
p3.reset(new int(2)); // undefined, multiple write
```

具体加锁的方式是：
```

std::shared_ptr<T> global_ptr;
std::mutex mu;

void ThreadOpSharedPtr() {
    std::shared_ptr<T> local_ptr;
    {
        MutexLockGuard lock(mu);
        local_ptr = global_ptr;
    }
    // use local_ptr do whatever you want, it's thread-safe now
}
```
