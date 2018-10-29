---
title: STL最重要的基础——空间配置器（allocator）
date: 2018-10-29 19:46:32
tags: cpp
---
### 缘由和作用
一般使用STL根本不需要关系allocator，但是要读懂STL首先要了解STL，因为allocator在STL实现中无处不在。

这东西有什么用呢？

首先要清楚一般的对象生成和销毁流程：
```
Obj* obj = new Obj;     // 创建内存空间，通过Obj()初始化构造实例
delete obj;             // 通过destroy销毁实例，释放内存空间
```

而，allocator做的事情就是把上面涉及到的4步拆分开来：
1. 通过allocate创建内存空间；
2. 通过construct调用对象构造函数在指定内存空间构造实例；
3. 通过destroy销毁实例；
4. 通过deallocate释放内存空间。

这个的作用就是通过更精细化管理内存，提升效率。

### 基本接口说明
```
      typedef size_t     size_type;
      typedef ptrdiff_t  difference_type;
      typedef _Tp*       pointer;
      typedef const _Tp* const_pointer;
      typedef _Tp&       reference;
      typedef const _Tp& const_reference;
      typedef _Tp        value_type;

      pointer
      address(reference __x)

      template<typename _Up, typename... _Args>
        void
        construct(_Up* __p, _Args&&... __args)
	  { ::new((void *)__p) _Up(std::forward<_Args>(__args)...); }

      template<typename _Up>
        void
        destroy(_Up* __p) { __p->~_Up(); }

      pointer
      allocate(size_type __n, const void* = 0);

      void
      deallocate(pointer __p, size_type __n);
```

其实最简单的就是：
1. allocate调用malloc分配空间；
2. deallocate调用free 释放空间；
3. construct调用类的初始化函数，针对指定空间，通过place new的方式；
4. destroy通过调用类的析构函数；

### 简单的allocator实现
一般使用的STL包含的代码在`https://github.com/gcc-mirror/gcc/tree/master/libstdc%2B%2B-v3/include/bits`，在这里面包括各种常用的数据结构。

简单的allocator实现可以看：`libstdc++-v3/include/ext/malloc_allocator.h`(`https://github.com/gcc-mirror/gcc/tree/master/libstdc%2B%2B-v3/include/bits`)。
```
      pointer
      address(reference __x) const _GLIBCXX_NOEXCEPT
      { return std::__addressof(__x); }

      pointer
      allocate(size_type __n, const void* = 0)
      {
        __ret = static_cast<_Tp*>(std::malloc(__n * sizeof(_Tp)));
	    return __ret;
      }

      void
      deallocate(pointer __p, size_type)
      { std::free(static_cast<void*>(__p)); }

      template<typename _Up, typename... _Args>
        void
        construct(_Up* __p, _Args&&... __args)
	  { ::new((void *)__p) _Up(std::forward<_Args>(__args)...); }

      template<typename _Up>
        void
        destroy(_Up* __p) { __p->~_Up(); }

```

### STL中vector调用的allocator实现
前面说了最简单的allocator实现，但是实际使用的allocator要复杂些，主要是使用了内存池的思想，提升分配小value的对象效率。

先来看看最常用的vector（`stl_vector.h`）：
```
  template<typename _Tp, typename _Alloc = std::allocator<_Tp> >
    class vector : protected _Vector_base<_Tp, _Alloc>
```
vector类定义时，定义了`typename _Alloc`，并且设定了默认的allocator（`std::allocator<_Tp>`），这个默认allocator路径是`allocator.h`：
```
  template<typename _Tp>
    class allocator: public __allocator_base<_Tp>
```
默认allocator继承自`__allocator_base`，路径是`libstdc++-v3/config/allocator/pool_allocator_base.h`：
```
namespace std
{
  template<typename _Tp>
    using __allocator_base = __gnu_cxx::__pool_alloc<_Tp>;
}
```
`__pool_alloc`的实现在`libstdc++-v3/include/ext/pool_allocator.h`：
```
  template<typename _Tp>
    class __pool_alloc : private __pool_alloc_base
```
其中，`__pool_alloc_base`定义了一个内存池：
```
  class __pool_alloc_base
    {
    protected:

      enum { _S_align = 8 };
      enum { _S_max_bytes = 128 };
      enum { _S_free_list_size = (size_t)_S_max_bytes / (size_t)_S_align };

      union _Obj
      {
	union _Obj* _M_free_list_link;
	char        _M_client_data[1];    // The client sees this.
      };

      static _Obj* volatile         _S_free_list[_S_free_list_size];

      // Chunk allocation state.
      static char*                  _S_start_free;
      static char*                  _S_end_free;
      static size_t                 _S_heap_size;

      size_t
      _M_round_up(size_t __bytes)
      { return ((__bytes + (size_t)_S_align - 1) & ~((size_t)_S_align - 1)); }

      _GLIBCXX_CONST _Obj* volatile*
      _M_get_free_list(size_t __bytes) throw ();

      __mutex&
      _M_get_mutex() throw ();

      // Returns an object of size __n, and optionally adds to size __n
      // free list.
      void*
      _M_refill(size_t __n);

      // Allocates a chunk for nobjs of size size.  nobjs may be reduced
      // if it is inconvenient to allocate the requested number.
      char*
      _M_allocate_chunk(size_t __n, int& __nobjs);
    };
```
其中可以看到，`_S_free_list`由`128/8=16`个指向`_Obj*`的数组组成，不同的byte size就从不同的数组中获取`_Obj*`，大于128的byte size不从pool中取。
```
  template<typename _Tp>
    _Tp*
    __pool_alloc<_Tp>::allocate(size_type __n, const void*)
    {
      pointer __ret = 0;
      if (__builtin_expect(__n != 0, true))
	{
	  if (__n > this->max_size())
	    std::__throw_bad_alloc();

	  const size_t __bytes = __n * sizeof(_Tp);

#if __cpp_aligned_new
	  if (alignof(_Tp) > __STDCPP_DEFAULT_NEW_ALIGNMENT__)
	    {
	      std::align_val_t __al = std::align_val_t(alignof(_Tp));
	      return static_cast<_Tp*>(::operator new(__bytes, __al));
	    }
#endif

	  // If there is a race through here, assume answer from getenv
	  // will resolve in same direction.  Inspired by techniques
	  // to efficiently support threading found in basic_string.h.
	  if (_S_force_new == 0)
	    {
	      if (std::getenv("GLIBCXX_FORCE_NEW"))
		__atomic_add_dispatch(&_S_force_new, 1);
	      else
		__atomic_add_dispatch(&_S_force_new, -1);
	    }

	  if (__bytes > size_t(_S_max_bytes) || _S_force_new > 0)   // __bytes大于128B的，直接new
	    __ret = static_cast<_Tp*>(::operator new(__bytes));
	  else
	    {                                                       // __bytes小于128B的，从_M_get_free_list中get
	      _Obj* volatile* __free_list = _M_get_free_list(__bytes);

	      __scoped_lock sentry(_M_get_mutex());
	      _Obj* __restrict__ __result = *__free_list;
	      if (__builtin_expect(__result == 0, 0))               // 如果free_list不够空间了，调用_M_refill
		__ret = static_cast<_Tp*>(_M_refill(_M_round_up(__bytes)));
	      else
		{
		  *__free_list = __result->_M_free_list_link;
		  __ret = reinterpret_cast<_Tp*>(__result);             // 从_M_get_free_list的free_list中获取一个Obj节点，这个链表后移
		}
	      if (__ret == 0)
		std::__throw_bad_alloc();
	    }
	}
      return __ret;
    }
```
上面的代码大致把allocator说明白了，还需要深入看的是`_M_refill`（代码路径：`libstdc++-v3/src/c++98/pool_allocator.cc`）：
```
  void*
  __pool_alloc_base::_M_refill(size_t __n)
  {
    int __nobjs = 20;
    char* __chunk = _M_allocate_chunk(__n, __nobjs);            // 直接分配20个__n大小的内存空间
    _Obj* volatile* __free_list;
    _Obj* __result;
    _Obj* __current_obj;
    _Obj* __next_obj;

    if (__nobjs == 1)
      return __chunk;                                           // 如果只分配出来一块，直接返回给上层使用
    __free_list = _M_get_free_list(__n);

    // Build free list in chunk.
    __result = (_Obj*)(void*)__chunk;                           // 分配出多块，就需要留出来一块，把其他的拼到free list
    *__free_list = __next_obj = (_Obj*)(void*)(__chunk + __n);  // 把新allocate的内存块拼接上__n对应的__free_list上
    for (int __i = 1; ; __i++)                                  // 注意，从1开始，因为0那个已经留给上层直接用了
      {
	__current_obj = __next_obj;
	__next_obj = (_Obj*)(void*)((char*)__next_obj + __n);
	if (__nobjs - 1 == __i)
	  {
	    __current_obj->_M_free_list_link = 0;
	    break;
	  }
	else
	  __current_obj->_M_free_list_link = __next_obj;
      }
    return __result;
  }
```

再看看怎么归还这些空间：
```
  template<typename _Tp>
    void
    __pool_alloc<_Tp>::deallocate(pointer __p, size_type __n)
    {
      if (__builtin_expect(__n != 0 && __p != 0, true))
	{
#if __cpp_aligned_new
	  if (alignof(_Tp) > __STDCPP_DEFAULT_NEW_ALIGNMENT__)
	    {
	      ::operator delete(__p, std::align_val_t(alignof(_Tp)));
	      return;
	    }
#endif
	  const size_t __bytes = __n * sizeof(_Tp);
	  if (__bytes > static_cast<size_t>(_S_max_bytes) || _S_force_new > 0)  // 如果是大于128B的，直接delete掉
	    ::operator delete(__p);
	  else                                                                  // 小于128B的，归还到对应__free_list中
	    {
	      _Obj* volatile* __free_list = _M_get_free_list(__bytes);
	      _Obj* __q = reinterpret_cast<_Obj*>(__p);

	      __scoped_lock sentry(_M_get_mutex());
	      __q ->_M_free_list_link = *__free_list;
	      *__free_list = __q;
	    }
	}
    }
```

构造和析构跟一般的allocator实现是一样的：
```
      template<typename _Up, typename... _Args>
        void
        construct(_Up* __p, _Args&&... __args)
	{ ::new((void *)__p) _Up(std::forward<_Args>(__args)...); }

      template<typename _Up>
        void
        destroy(_Up* __p) { __p->~_Up(); }
```
