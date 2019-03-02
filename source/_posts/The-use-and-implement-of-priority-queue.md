---
title: priority_queue的用法和实现
date: 2018-11-19 10:39:23
tags: cpp
---
### priority_queue用法
1. 使用优先队列接口push和pop，优先队列内部通过heap接口rebalance，不需用户关心，非常方便；
2. 使用优先队列场景来说，往往都是自定义的元素，一般需要自定义比较器。

```
#include <iostream>
#include <queue>
#include <vector>
#include <utility>

using my_pair_t = std::pair<size_t,bool>;
using my_container_t = std::vector<my_pair_t>;

int main() {
    auto my_comp =
        [](const my_pair_t& e1, const my_pair_t& e2)
        { return e1.first > e2.first; };
    std::priority_queue<my_pair_t,
                        my_container_t,
                        decltype(my_comp)> queue(my_comp);
    queue.push(std::make_pair(5, true));
    queue.push(std::make_pair(3, false));
    queue.push(std::make_pair(7, true));
    std::cout << std::boolalpha;
    while(!queue.empty())
    {
        const auto& p = queue.top();
        std::cout << p.first << " " << p.second << "\n";
        queue.pop();
    }
}
```

### priority_queue实现
`priority_queue`利用stl的`heap+vector`实现的。提供的`top()`、`pop()`和`push()`接口足够使用。具体实现不难，直接看代码吧。

```
  template<typename _Tp, typename _Sequence = vector<_Tp>,
	   typename _Compare  = less<typename _Sequence::value_type> >
    class priority_queue
    {
    protected:
      //  See queue::c for notes on these names.
      _Sequence  c;
      _Compare   comp;

      priority_queue(const _Compare& __x = _Compare(),
		     const _Sequence& __s = _Sequence())
      : c(__s), comp(__x)
      { std::make_heap(c.begin(), c.end(), comp); }

      template<typename _InputIterator>
	  priority_queue(_InputIterator __first, _InputIterator __last,
		       const _Compare& __x,
		       const _Sequence& __s)
	  : c(__s), comp(__x)
	  {
	    __glibcxx_requires_valid_range(__first, __last);
	    c.insert(c.end(), __first, __last);
	    std::make_heap(c.begin(), c.end(), comp);
	  }

	  bool
      empty() const
      { return c.empty(); }

      size_type
      size() const
      { return c.size(); }

      const_reference
      top() const
      {
	    __glibcxx_requires_nonempty();
	    return c.front();
      }

      void
      push(const value_type& __x)
      {
	    c.push_back(__x);
	    std::push_heap(c.begin(), c.end(), comp);
      }

      template<typename... _Args>
	  void
	  emplace(_Args&&... __args)
	  {
	    c.emplace_back(std::forward<_Args>(__args)...);
	    std::push_heap(c.begin(), c.end(), comp);
	  }

	  void
      pop()
      {
	    __glibcxx_requires_nonempty();
	    std::pop_heap(c.begin(), c.end(), comp);
	    c.pop_back();
      }

    }
```
