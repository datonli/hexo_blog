---
title: 基类和派生类虚析构对释放资源影响
date: 2018-04-19 23:34:15
tags: cpp
---

小结：
1. 如果操作的指针总是派生类指针，无论析构虚不虚，都不影响资源释放；
2. 如果操作的指针是基类，虚析构才能释放派生类的资源，非虚析构会造成派生类不调用派生类析构函数。

具体测试代码和结果输出如下。

#### 0. 代码

```
#include <iostream>

using namespace std;

class Resource {
    public:
        Resource(string res) : res_(res) {
            cout << "Resource " << res_ << " construct" << endl;
        }
        ~Resource() {
            cout << "Resource " << res_ << " deconstruct" << endl;
        }
    private:
        string res_;
};

class Base {
    public:
        Base() {
            r_pro = new Resource(string("protected"));
            r_pri = new Resource(string("private"));
            cout << "Base::Base" << endl;
        }
        virtual ~Base() {
            delete r_pro;
            delete r_pri;
            cout << "~Base::Base" << endl;
        }

    protected:
        Resource* r_pro;
    private:
        Resource* r_pri;
};

class Derive : public Base {
    public:
        Derive() {
            cout << "Derive::Derive" << endl;
        }
        ~Derive() {
            cout << "~Derive::Derive" << endl;
        }
};

int main() {
    Base* b = new Derive();
    delete b;
    return 0;
}
```

#### 1. 基类虚析构时，执行：
```
    Base* b = new Derive();
    delete b;
```
输出：
```
    Base::Base
    Resource protected construct
    Resource private construct
    
    Derive::Derive
    
    ~Derive::Derive
    
    ~Base::Base
    Resource protected deconstruct
    Resource private deconstruct
```

#### 2. 基类非虚析构时，执行：
```
    Base* b = new Derive();
    delete b;
```
输出：
```
    Base::Base
    Resource protected construct
    Resource private construct
    
    Derive::Derive
    
    ~Base::Base
    Resource protected deconstruct
    Resource private deconstruct
```

#### 3. 基类虚析构时，执行：
```
    Derive* d = new Derive();
    delete d;
```
输出：
```
    Base::Base
    Resource protected construct
    Resource private construct
    
    Derive::Derive
    
    ~Derive::Derive
    
    ~Base::Base
    Resource protected deconstruct
    Resource private deconstruct
```

#### 4. 基类非虚析构时，执行：
```
    Derive* d = new Derive();
    delete d;
```
输出：
```
    Base::Base
    Resource protected construct
    Resource private construct
    
    Derive::Derive
    
    ~Derive::Derive
    
    ~Base::Base
    Resource protected deconstruct
    Resource private deconstruct
```