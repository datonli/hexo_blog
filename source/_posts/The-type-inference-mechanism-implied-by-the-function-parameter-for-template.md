---
title: template函数参数暗含的类型推导机制
date: 2018-04-17 22:28:38
tags: cpp
---
在使用template的时候，经常会因为不了解`c++`的类型推导（Deducing Types）导致编译不过。在此做一下记录。参考`《Effective Modern C++》Chapter 1`。分为3中情况：

1. ParamType是Reference或者Pointer，但不是Universal Reference时

Reference时：
```
template<typename T>
void f(T& param);           // param is a reference
int x = 27;                 // x is an int
const int cx = x;           // cx is a const int
const int& rx = x;          // rx is a reference to x as a const int

f(x);                       // T is int, param's type is int&

f(cx);                      // T is const int,
                            // param's type is const int&

f(rx);                      // T is const int,
                            // param's type is const int&
```

const Reference时：
```
template<typename T>
void f(const T& param);     // param is now a ref-to-const
int x = 27;
const int cx = x;
const int& rx = x;

f(x);                       // T is int, param's type is const int&

f(cx);                      // T is int, param's type is const int&

f(rx);                      // T is int, param's type is const int&
```

Pointer时：
```
template<typename T>
void f(T* param);           // param is now a pointer

int x = 27;                 // as before
const int *px = &x;         // px is a ptr to x as a const int

f(&x);                      // T is int, param's type is int*

f(px);                      // T is const int,
                            // param's type is const int*
```
2. ParamType是Universal Reference时

**Universal Reference意思是使用&&作为param前缀，这时传入的可以使右值，也可以是左值，如果在这个函数内需要调用其他函数并且传参为右值的话，需要用std::perfect_forward，否则会传左值**
```
template<typename T>
void f(T&& param);          // param is now a universal reference

int x = 27;
const int cx = x;
const int& rx = x;

f(x);                       // x is lvalue, so T is int&, 
                            // param's type is also int&

f(cx);                      // cx is lvalue, so T is const int&,
                            // param's type is also const int&

f(rx);                      // rx is lvalue, so T is const int&,
                            // param's type is also const int&

f(27);                      // 27 is rvalue, so T is int,
                            // param's type is therefore int&&
```

3. ParamType既不是Pointer又不是Reference时
```
template<typename T>
void f(T param);            // param is now passed by value

int x = 27;
const int cx = x;
const int& rx = x;

f(x);                       // T's and param's types are both int

f(cx);                      // T's and param's types are again both int

f(rx);                      // T's and param's types are still both int
```
假如expr是const pointer to a const object时，expr pass by-value：
```
template<typename T>
void f(T param);            // param is now passed by value

const char* const ptr =     // ptr is const pointer to const object
    "Fun with pointers";

f(ptr);                     // pass arg of type const char * const

```

除了上面的情况，还有传array和function作为参数的，后面用到在做总结吧。