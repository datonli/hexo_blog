---
title: 线程安全队列
date: 2018-07-13 21:14:17
tags: cpp
---

记录一个非常好的线程安全队列写法。非常方便、简洁的实现。
```
// thread_safe_queue.h
template <typename T>
class ThreadSafeQueue {
public:
         void Insert(T value);
         void Popup(T &value);
         bool Empety();
 
private:
       mutable std::mutex mut_;
       std::queue<T> que_;
       std::condition_variable cond_;
};
```

对应接口实现。
```
// thread_safe_queue.cpp
template <typename T>
void ThreadSafeQueue::Insert(T value) {
        std::lock_guard<std::mutex> lk(mut_);
        que_.push_back(value);
        cond_.notify_one();
}
 
 
template <typename T>
void ThreadSafeQueue::Popup(T &value) {
	std::unique_lock<std::mutex> lk(mut_);
	cond_.wait(lk, [this]{return !que_.empety();});
	value = que_.front();
	que_.pop();
}
 
 
template <typename T>
bool ThreadSafeQueue::Empty() const {
        std::lock_guard<std::mutex> lk(mut_);
        return que_.empty();
}
```