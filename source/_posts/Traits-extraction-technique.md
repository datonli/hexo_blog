---
title: Traits萃取技术
date: 2018-11-15 19:32:17
tags: cpp
---

### 概述
traits是一种特性萃取技术,它在Generic Programming中被广泛运用,常常被用于使不同的类型可以用于相同的操作,或者针对不同类型提供不同的实现.traits在实现过程中往往需要用到以下三种C++的基本特性:
1. enum
2. typedef
3. template (partial) specialization

其中:
1. enum用于将在不同类型间变化的标示统一成一个,它在C++中常常被用于在类中替代define,你可以称enum为类中的define;
2. typedef则用于定义你的模板类支持特性的形式,你的模板类必须以某种形式支持某一特性,否则类型萃取器traits将无法正常工作.看到这里你可能会想,太苛刻了吧?其实不然,不支持某种特性本身也是一种支持的方式(见示例2,我们定义了两种标示,__xtrue_type和__xfalse_type,分别表示对某特性支持和不支持).
3. template (partial) specialization被用于提供针对特定类型的正确的或更合适的版本.

traits可被用于针对不同类型提供不同的实现。

### 例一
假定我们需要为某个类设计一个可以对所有类型(包括普通的int/long...,提供了clone方法的复杂类型CComplexObject,及由该类派生的类)进行操作的函数clone,下面,先用OO的方法来考虑一下解决方案.看到前面的条件,最先跳进你脑子里的肯定是Interface,pure virtual function等等.对于我们自己设计的类CComplexObject而言,这不是问题,但是,对于基本数据类型呢?还有那些没有提供clone方法的复杂类型呢?

```
template <typename T, bool isClonable>
class XContainer
{
     ...
     void clone(T* pObj)
     {
         if (isClonable)
         {
             pObj->clone();
         }
         else
         {
             //... non-Clonable algorithm ...
         }
     }
};
```

但是只要你测试一下,这段代码不能通过编译.为什么会这样呢?原因很简单:对于没有实现clone方法的非Clonable类或基本类型,pObj->clone这一句是非法的.
那么怎样解决上面的这个难题呢?上面不能通过编译的代码告诉我们,要使我们的代码通过编译,就不能使非Clonable类或基本类型的代码中出现pObj->clone,即我们需要针对不同类型提供不同的实现.为了实现这一点,我们可以在我们的模板类中用enum定义一个trait,以标示类是否为Clonable类,然后在原模板类内部引入一个traits提取类Traits,通过对该类进行specilizing,以根据不同的trait提供不同的实现.具体实现如下:

```
#include <iostream>
using namespace std;

class CComplexObject // a demo class
{
public:
     void clone() { cout << "in clone" << endl; }
};

// Solving the problem of choosing method to call by inner traits class
template <typename T, bool isClonable>
class XContainer
{
public:
     enum {Clonable = isClonable};

     void clone(T* pObj)
     {
         Traits<isClonable>().clone(pObj);
     }

     template <bool flag>
         class Traits
     {
     };

     template <>
         class Traits<true>
     {
     public:
         void clone(T* pObj)
         {
             cout << "before cloning Clonable type" << endl;
             pObj->clone();
             cout << "after cloning Clonable type" << endl;
         }
     };

     template <>
         class Traits<false>
     {
     public:
         void clone(T* pObj)
         {
             cout << "cloning non Clonable type" << endl;
         }
     };
};

void main()
{
     int* p1 = 0;
     CComplexObject* p2 = 0;

     XContainer<int, false> n1;
     XContainer<CComplexObject, true> n2;

     n1.clone(p1);
     n2.clone(p2);
}
```
编译运行一下,上面的程序输出如下的结果:
```
cloning non Clonable type
before doing something Clonable
in clone
after doing something Clonable
```
这说明,我们成功地根据传入的isClonable模板参数为模板实例选择了不同的操作,在保证接口相同的情况下,为不同类型提供了不同的实现.

### 例二
我们再对上面的例子进行一些限制,假设我们的clone操作只涉及基本类型和CComplexObject及其派生类,那么我们可以进一步给出下面的解法:
```
#include <iostream>
using namespace std;

struct __xtrue_type { }; // define two mark-type
struct __xfalse_type { };

class CComplexObject // a demo class
{
public:
     virtual void clone() { cout << "in clone" << endl; }
};

class CDerivedComplexObject : public CComplexObject // a demo derived class
{
public:
     virtual void clone() { cout << "in derived clone" << endl; }
};

// A general edtion of Traits
template <typename T>
struct Traits
{
     typedef __xfalse_type has_clone_method; // trait 1: has clone method or not? All types defaultly has no clone method.
};

// Specialized edtion for ComplexObject
template <>
struct Traits<CComplexObject>
{
     typedef __xtrue_type has_clone_method;
};

template <typename T>
class XContainer
{
     template <typename flag>
         class Impl
     {
     };
     template <>
         class Impl <__xtrue_type>
     {
     public:
         void clone(T* pObj)
         {
             pObj->clone();
         }
     };
     template <>
         class Impl <__xfalse_type>
     {
     public:
         void clone(T* pObj)
         {
         }
     };
public:
     void clone(T* pObj)
     {
         Impl<Traits<T>::has_clone_method>().clone(pObj);
     }
};

void main()
{
     int* p1 = 0;
     CComplexObject c2;
     CComplexObject* p2 = &c2;
     CDerivedComplexObject c3;
     CComplexObject* p3 = &c3; // you must point to a derived object by a base-class pointer,
                             //it's a little problem

     XContainer<int> n1;
     XContainer<CComplexObject> n2;
     XContainer<CComplexObject> n3;

     n1.clone(p1);
     n2.clone(p2);
     n3.clone(p3);
}

```
现在,所有基本类型以及CComplexObject类系都可以用于XContainer了.

另外延伸一下：上面指定的特化都是全特化，另外还有偏特化，偏特化跟全特化对比就像是导数和偏导数，偏特化本身还是一个模板，只是其中一个模板参数做了特化了。
