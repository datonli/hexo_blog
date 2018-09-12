---
title: Cast转换总结
date: 2018-09-12 19:16:58
tags: cpp
---

写代码过程会遇到需要使用cast的场景，常见的有`static_cast`，`dynamic_cast`，`const_cast`等，使用时有些容易忽略或理解不清的地方导致误用。

#### `static_cast`使用要点
`static_cast`操作符可用于将一个指向基类的指针转换为指向子类的指针。但是这样的转换不总是安全的。`static_cast`不够安全，就是指在运行阶段不进行类型检查。
具体用法：
1. 用于类层次结构中基类（父类）和派生类（子类）之间指针或引用的转换；进行上行转换（把派生类的指针或引用转换成基类表示）是安全的；进行下行转换（把基类指针或引用转换成派生类表示）时，由于没有动态类型检查，所以是不安全的。
2. 用于基本数据类型之间的转换，如把int转换成char，把int转换成enum。这种转换的安全性也要开发人员来保证。
3. 把空指针转换成目标类型的空指针。
4. 把任何类型的表达式转换成void类型。
*static_cast不能转换掉expression的const、volatile、或者__unaligned属性。*

```
#include <vector>
#include <iostream>

struct B {
    int m = 0;
    void hello() const {
        std::cout << "Hello world, this is B!\n";
    }
};
struct D : B {
    void hello() const {
        std::cout << "Hello world, this is D!\n";
    }
};

enum class E { ONE = 1, TWO, THREE };
enum EU { ONE = 1, TWO, THREE };

int main()
{
    // 1: initializing conversion
    int n = static_cast<int>(3.14);
    std::cout << "n = " << n << '\n';
    std::vector<int> v = static_cast<std::vector<int>>(10);
    std::cout << "v.size() = " << v.size() << '\n';

    // 2: static downcast
    D d;
    B& br = d; // upcast via implicit conversion
    br.hello();
    D& another_d = static_cast<D&>(br); // downcast
    another_d.hello();

    // 3: lvalue to xvalue
    std::vector<int> v2 = static_cast<std::vector<int>&&>(v);
    std::cout << "after move, v.size() = " << v.size() << '\n';

    // 4: discarded-value expression
    static_cast<void>(v2.size());

    // 5. inverse of implicit conversion
    void* nv = &n;
    int* ni = static_cast<int*>(nv);
    std::cout << "*ni = " << *ni << '\n';

    // 6. array-to-pointer followed by upcast
    D a[10];
    B* dp = static_cast<B*>(a);

    // 7. scoped enum to int or float
    E e = E::ONE;
    int one = static_cast<int>(e);
    std::cout << one << '\n';

    // 8. int to enum, enum to another enum
    E e2 = static_cast<E>(one);
    EU eu = static_cast<EU>(e2);

    // 9. pointer to member upcast
    int D::*pm = &D::m;
    std::cout << br.*static_cast<int B::*>(pm) << '\n';

    // 10. void* to any type
    void* voidp = &e;
    std::vector<int>* p = static_cast<std::vector<int>*>(voidp);
}
```

#### `dynamic_cast`使用要点
`dynamic_cast`的转换也需要目标类型和源对象有一定的关系：继承关系。 更准确的说，`dynamic_cast`是用来检查两者是否有继承关系。因此该运算符实际上只接受基于类对象的指针和引用的类转换。

##### 对于指针的场景
用`dynamic_cast`转指针时，如果不符合继承关系的向下转换时，会返回`nullptr`，非常便于判断。
```
#include <iostream>

struct V {
    virtual void f() {};  // must be polymorphic to use runtime-checked dynamic_cast
};
struct A : virtual V {};
struct B : virtual V {
  B(V* v, A* a) {
    // casts during construction (see the call in the constructor of D below)
    dynamic_cast<B*>(v); // well-defined: v of type V*, V base of B, results in B*
    dynamic_cast<B*>(a); // undefined behavior: a has type A*, A not a base of B
  }
};
struct D : A, B {
    D() : B((A*)this, this) { }
};

struct Base {
    virtual ~Base() {}
};

struct Derived: Base {
    virtual void name() {}
};

int main()
{
    D d; // the most derived object
    A& a = d; // upcast, dynamic_cast may be used, but unnecessary
    D& new_d = dynamic_cast<D&>(a); // downcast
    B& new_b = dynamic_cast<B&>(a); // sidecast


    Base* b1 = new Base;
    if(Derived* d = dynamic_cast<Derived*>(b1))
    {
        std::cout << "downcast from b1 to d successful\n";
        d->name(); // safe to call
    }

    Base* b2 = new Derived;
    if(Derived* d = dynamic_cast<Derived*>(b2))
    {
        std::cout << "downcast from b2 to d successful\n";
        d->name(); // safe to call
    }

    delete b1;
    delete b2;
}
```


##### 对于引用场景
使用`dynamic_cast`转引用时，如果不符合继承关系的向下转换，会抛出`std::bad_cast`异常。
```
#include <iostream>
#include <typeinfo>

struct Foo { virtual ~Foo() {} };
struct Bar { virtual ~Bar() {} };

int main()
{
    Bar b;
    try {
        Foo& f = dynamic_cast<Foo&>(b);
    } catch(const std::bad_cast& e)
    {
        std::cout << e.what() << '\n';
    }
}
```

#### `const_cast`使用要点
对于const类型变量，只能使用`const_cast`进行转换。
```
#include <iostream>

struct type {
    int i;

    type(): i(3) {}

    void f(int v) const {
        // this->i = v;                 // compile error: this is a pointer to const
        const_cast<type*>(this)->i = v; // OK as long as the type object isn't const
    }
};

int main()
{
    int i = 3;                 // i is not declared const
    const int& rci = i;
    const_cast<int&>(rci) = 4; // OK: modifies i
    std::cout << "i = " << i << '\n';

    type t; // if this was const type t, then t.f(4) would be undefined behavior
    t.f(4);
    std::cout << "type::i = " << t.i << '\n';

    const int j = 3; // j is declared const
    int* pj = const_cast<int*>(&j);
    // *pj = 4;      // undefined behavior

    void (type::* pmf)(int) const = &type::f; // pointer to member function
    // const_cast<void(type::*)(int)>(pmf);   // compile error: const_cast does
                                              // not work on function pointers
}
```

