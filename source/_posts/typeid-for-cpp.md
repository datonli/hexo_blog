---
title: cpp中的typeid多态使用
date: 2018-07-15 21:14:51
tags: cpp
---
在c++中使用多态，会遇到接口传入基类，而在接口内部对基类不同派生类调用不同函数，做不同操作的需求。typeid提供了对基类引用获取其实际派生类的方法，在多态使用时非常有帮助。
```
#include <iostream>
#include <typeinfo>
struct Base {}; // non-polymorphic
struct Derived : Base {};
struct Base2 { virtual void foo() {} }; // polymorphic
struct Derived2 : Base2 {};

int main() {
    // Non-polymorphic lvalue is a static type
    Derived d1;
    Base& b1 = d1;
    std::cout << "reference to non-polymorphic base: " << typeid(b1).name() << '\n';
    Derived2 d2;
    Base2& b2 = d2;
    std::cout << "reference to polymorphic base: " << typeid(b2).name() << '\n';
    if (typeid(b2).name() == typeid(Derived2).name()) {
        std::cout << "same type" << std::endl;
    }
    return 0;
}    
```
输出结果是：
```
reference to non-polymorphic base: 4Base
reference to polymorphic base: 8Derived2
same type
```
注意，typeid只对引用有效！！