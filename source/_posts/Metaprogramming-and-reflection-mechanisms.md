---
title: 元编程与反射机制
date: 2018-08-04 23:59:15
tags: cpp
---
### 元编程
元编程是一种比较陌生的编程范式，有两个好处：
1. 编译期进行计算，编译期之后直接得到常量结果；
2. 通过模板生成通用代码；

#### 元编程实现编译期计算
可以直接用模板+模板特化+enum定义常量类型，实现编译期进行计算
```
template <int N>
struct Factorial {                             
    enum { value = N * Factorial<N - 1>::value; }
};

template <>
struct Factorial<0> {
    enum { value = 1; }
};

// Factorial<4>::value == 24
// Factorial<0>::value == 1
void foo() {
    int x = Factorial<4>::value; // == 24
    int y = Factorial<0>::value; // == 1
    std::cout << "x = " << x << ", y = " << y << std::endl;
}  
```
可以把enum替换成static const的类型，作为static类型是为了可以直接方便模板类型的调用
```
template <int N>
struct Factorial {                             
    static const int value = N * Factorial<N - 1>::value;
};

template <>
struct Factorial<0> {
    static const int value = 1;
};

// Factorial<4>::value == 24
// Factorial<0>::value == 1
void foo() {
    int x = Factorial<4>::value; // == 24
    int y = Factorial<0>::value; // == 1
    std::cout << "x = " << x << ", y = " << y << std::endl;
}  
```

#### 元编程模板生成代码
模板生成适用<int, long>和<double, long>输入类型的类模板
```
// template_kv.h
struct T1 {
    using Req = int;
    using Res = long;
};

struct T2 {
    using Req = double;
    using Res = long;
};

template<class T>
class foo {
public:
    foo() {
        req = 3;
        res = 5;
    }                                                                                                                      
    void print() {
        std::cout << "req = " << req << ", res = " << res << std::endl;
    }                                                                                                                      
private:
    typename T::Req req;
    typename T::Res res;
}; 

// main.cpp
int main() {
    foo<T2> ff;
    ff.print();
    return 0;
}   
```

### 反射机制
反射机制，在Java中的反射是通过JVM的机制，动态加载编译好的`.class`文件，反序列化到内存中的对象，有Jave虚拟机保证得到类的元信息。

而`C++`是强类型语言，要求在编译器期间把几乎（除了虚函数实现的多态）所有都静态确定好。所以反射机制在`C++`层面是不提供的，并且是不好实现的。

近日看`protobuf v3.6.1`的源代码中使用反射实现了proto的动态加载，多有裨益。在pb中，使用`Descriptor`保存proto文件的`message`中属性，这得到的就是用户定义的`message`的元信息，pb提供的对元信息操作的接口，将元信息传入`reflection`类中通过`reflection`的函数操作，具体的pb关键uml图如下。

![protobuf uml](o_protobuf_classdiagram.png)

反射机制最关键的地方就是通过元信息对数据进行操作。

为什么把元编程和反射机制放在一起讨论？其实是有关系的，要实现元编程的代码生成，有两种方法：
1. 不用模板，直接用宏编程，通过把源代码直接翻译为目的代码；
2. 使用模板；

反射机制在这里面担任的是什么角色呢？其实对于pb来说，把proto翻译为`.h&.cc`，并不需要反射，直接用宏编程，可以直接翻译出来代码。而反射机制提供了动态加载的思路，具体来说就是：在程序编译成二进制运行之后，还可以通过动态加载一个pb的message，在程序中调用pb接口+反射，通过反射对元信息调用，直接得到内存中的对象实例。

另外，如果要动态加载，也有一个别的办法——unload和load动态链接库`.so`文件，只要保证调用接口一致，就可以实现动态加载和热更新，具体的操作函数是：
```
#include <dlfcn.h>

void *dlopen(const char *filename, int flag);
char *dlerror(void);
void *dlsym(void *handle, const char *symbol);
int dlclose(void *handle);
```
