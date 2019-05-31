---
title: 容器set自定义比较函数
date: 2019-06-01 00:11:29
tags: cpp
---

在c++中使用STL，经常需要自定义容器的比较函数。因为c++11引入了匿名函数，而匿名函数也是可以用于作为比较函数的，不过使用匿名函数作为比较函数的时候需要把函数作为参数传给构造函数。

在使用过程中，有一个有趣的点是，`set<pair<int, int>>`类型是不需要自定义比较函数的，内置已经支持，也就是说，作为内置的类型`pair`是会被STL容器默认支持的，往往需要自定义比较函数的是struct/class。

下面是具体的例子：
```
#include <iostream>
#include <map>
#include <set>

using namespace std;

struct Point {
 public:
  int x;
  int y;
  Point(int a, int b) : x(a), y(b) {}
};

struct SetCmp {
  bool operator()(const Point& first, const Point& second) {
    if (first.x == second.x) {
      if (first.y == second.y) {
        return false;
      }
      return first.y < second.y;
    } else {
      return first.x < second.x;
    }
  }
};

auto SetCmp2 = [](const Point& first, const Point& second) -> bool {
  if (first.x == second.x) {
    if (first.y == second.y) {
      return false;
    }
    return first.y < second.y;
  } else {
    return first.x < second.x;
  }
};

int main() {
  set<Point, SetCmp> point_set;
  point_set.insert({Point(1, 2), Point(1, 3), Point(3, 1), Point(2, 3),
                    Point(2, 1), Point(2, 2), Point(1, 1)});
  cout << "set size = " << point_set.size() << endl;
  for (auto& p : point_set) {
    cout << p.x << ", " << p.y << endl;
  }
  auto low_it = point_set.lower_bound(Point(2, 0));
  auto up_it = point_set.upper_bound(Point(2, 2));
  cout << "range display ..." << endl;
  while (low_it != up_it) {
    cout << low_it->x << ", " << low_it->y << endl;
    low_it++;
  }

  set<Point, decltype(SetCmp2)> point_set2(SetCmp2);
  point_set2.insert({Point(1, 2), Point(1, 3), Point(3, 1), Point(2, 3),
                     Point(2, 1), Point(2, 2), Point(1, 1), Point(2, 1),
                     Point(2, 2)});
  cout << "set2 size = " << point_set2.size() << endl;
  for (auto& p : point_set2) {
    cout << p.x << ", " << p.y << endl;
  }

  map<int, set<Point, decltype(SetCmp2)>> point_set_map;
  set<Point, decltype(SetCmp2)> point_set3(SetCmp2);

  point_set3.insert(
      {Point(1, 2), Point(1, 3), Point(3, 1), Point(2, 3), Point(2, 2)});
  point_set_map.emplace(1, std::move(point_set3));

  // compile error, candidate constructor (the implicit move constructor) not
  // viable: requires 1 argument, but 0 were provided
  // map<int, set<Point, decltype(SetCmp2)>> point_set_map2;
  // set<Point, decltype(SetCmp2)> &point_set4 = point_set_map2[1];
  return 0;
}
```

注意，最后的例子里面使用`map`嵌套`set`，`set`使用匿名函数作为比较器，这时候需要将匿名函数传给`set`初始化参数，否则就会初始化失败，而使用`map[]`希望返回`set`对象时，使用的是默认构造函数，而因为在模板中定义了需要传入比较器类型，所以会出现`requires 1 argument, but 0 were provided`的错误信息。