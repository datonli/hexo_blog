---
title: 奇异递归模板模式（CRTP）——enable_shared_from_this实现原理
date: 2018-11-17 15:56:44
tags: cpp
---
### CRTP
奇异递归模板模式(curiously recurring template pattern,CRTP)，是cpp中一种静态多态技术。大致如：
```
// The Curiously Recurring Template Pattern (CRTP)
template <class T>
class Base {
public:
    void interface()
    {
        // ...
        static_cast<T*>(this)->implementation();
        // ...
    }

    static void static_func()
    {
        // ...
        T::static_sub_func();
        // ...
    }
private:
    friend class T;
};

class Derived : Base<Derived> {
public:
    void implementation();
    static void static_sub_func();
};
```
CRTP简单来说就是：`以派生类作为基类模板参数`，这样在基类中就能通过`static_cast<T*>(this)`这种方式获取派生类对象做函数操作，并且往往还可以在基类中加入派生类类型友元，方便派生类访问基类对象。

### CRTP好处
绕这么大圈有啥好处呢？
1. 最大的好处就是在基类的非虚函数中可以调用派生类对象函数，注意，是非虚函数，这意味着不需要使用虚函数+动态多态这套东西可以实现多态的功能，如上面例子中`static_cast<T*>(this)->implementation()`转换之后其实就是派生类正常调用，随着模板展开这里其实是静态的，没有动态的特性，但是却对不同派生类有不同的实现（其实就是多态），也叫做静态多态。
2. 另一个好处就是在父类中保存有子类的实例信息，经典实现就是`enable_shared_from_this`。

### `enable_shared_from_this`源码解析
`enable_shared_from_this`使用说明：
1. 目标类继承基类`template<typename _Tp> class enable_shared_from_this`；
2. 调用`shared_ptr<_Tp> shared_from_this()`获取目标类的shared_ptr。

下面是具体实现：
```
template<typename _Tp>
class enable_shared_from_this {
public:
  shared_ptr<_Tp>
  shared_from_this()
  { return shared_ptr<_Tp>(this->_M_weak_this); }

private:
  mutable weak_ptr<_Tp>  _M_weak_this;
};
```
目标类使用范例：
```
class TargetObj : public enable_shared_from_this<TargetObj> {
public:
    TargetObj() {}
    void DoSth(OtherObj* obj) { obj->invoke(shared_from_this()); }
}

class OtherObj {
public:
    void invoke(shared_ptr<TargetObj> obj) { ... }
}
```

当`new TargetObj`的同时，会调用基类`enable_shared_from_this`构造`weak_ptr`，这样在调用到使用`shared_from_this()`时，就从基类的`weak_ptr` promote为`shared_ptr`返回。
