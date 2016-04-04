title: 【C++11】列表初始化
tags:
  - C++11
categories:
  - C++
date: 2016-03-20 16:03:00
---

在C++11之前，初始化一个变量有3种方式：

## 1.默认初始化(default initialization)
```cpp
T obj;
```
对于内置类型与指针，

## 2.直接初始化(direct initialization)
```cpp
T obj(args...);
```
## 3.拷贝初始化(copy initialization)
```cpp
T obj = other;
```

可以看出以上3种初始化方式语法不够统一，为了解决这个问题，C++11提出了列表初始化语法。

## 4.列表初始化(list initialization)
```cpp
T obj {args...};
```
