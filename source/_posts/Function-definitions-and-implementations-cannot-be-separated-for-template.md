---
title: template函数定义和实现不能分离
date: 2018-04-17 22:33:13
tags:
---
今天遇到一个编译器的小坑，使用template在编译的时候不能把定义和实现分别放在.h和.cpp中，不然会报`Undefined symbols for architecture x86_64` 或 `undefined reference to`错，出错在链接的时候（`collect2: error: ld returned 1 exit status`）。

出错原因是：使用模板的时候，程序编译期间因为能够找到模板函数的声明(.h)，所以编译器不会报错；但是在链接库（.so or .a）的时候，因为模板在编译器中是根据调用者传入不同的参数类型编译出不同的代码块，在编译期间只对模板函数声明进行进行编译，而实现却没有进行编译，链接期间无法找到模板传入不同参数实现的代码库，而导致链接报错。

测试代码就是把下面的代码拆到.h和.cpp中。

```
// template_test.cpp
#include <iostream>
using namespace std;
//递归终止函数
void print()
{
   cout << "empty" << endl;
}
//展开函数
template <class T, class ... Args>
void print(T head, Args ... rest)
{
   cout << "parameter " << head << endl;
   print(rest...);
}


int main(void)
{
   print(1,2,3,4);
   return 0;
}
```