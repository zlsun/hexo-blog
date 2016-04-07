title: 使用列表初始化语法初始化链表
tags:
  - C++11
categories:
  - C++
date: 2016-04-07 20:18:23
---

# 问题

已知链表节点定义如下：

```cpp
class ListNode {
public:
    int val;
    ListNode* next;
public:
    ListNode(int v, ListNode* n = NULL): val(v), next(n) {}
};
```

如何用 C++ 初始化出指定链表，如：1->2->3？

# 一般做法

1. 直接法，这种方法遇到长的链表会导致一行长度过长：
    ```cpp
    ListNode* list = new ListNode(1, new ListNode(2, new ListNode(3)));
    ```
2. 分步法，缩短列宽，缺点是初始化长度为n的链表需要n行代码：
    ```cpp
    ListNode* list = new ListNode(1);
    ListNode* node2 = list->next = new ListNode(2);
    node2->next = new ListNode(3);
    ```
2. 循环法，最通用的做法：
    ```cpp
    int values[] = {1, 2, 3};
    ListNode* list = new ListNode(values[0]);
    ListNode* cur = list;
    for (int i = 1; i < 3; ++i) {
        cur->next = new ListNode(values[i]);
        cur = cur->next;
    }
    ```

可以看出，这3种方法都比较啰嗦，如果使用 C++11 的列表初始化，可以简化很多。

# <ruby>列表<rt>list </rt></ruby><ruby>初始化<rt>initialization</rt></ruby>

[列表初始化](http://en.cppreference.com/w/cpp/language/list_initialization)是C++11在旧语法的基础上提出的新语法。其提出是为了解决C++混乱的初始化语法：

```cpp
int i = 1;              // C风格的初始化
int i(1);               // 构造初始化，与上面等价
int arr[] = {1, 2, 3};  // 花括号初始化，用于初始化数组或者结构体
struct { int i; char c; } a = {1, 'c'};
```

有了列表初始化，上面的初始化可以统一写成：

```cpp
int i {1};
int arr[] {1, 2, 3};
struct { int i; char c; } a {1, 'a'};
```

STL 中的容器也支持这种初始化方法：

```cpp
std::vector<int> v {1, 2, 3};
std::map<char, int> m {{'a', 1}, {'b', 2}};
```

这些容器是通过添加一个以[`std::initializer_list`](http://en.cppreference.com/w/cpp/utility/initializer_list)为参数的构造函数实现的。

以`std::vector`为例，其大致实现代码如下：

```cpp
template <typename T>
class vector {
    ......
    vector(initializer_list<T> lst) {
        // lst是一个容器，可以通过迭代器遍历出其中的元素，用于初始化vector
        ...
    }
    ......
};
```

# 初始化链表

现在，让我们再来看下开头的问题。如何利用列表初始化语法初始化链表呢？

可以直接这样做：

```cpp
ListNode* list {1, new ListNode {2, new ListNode {3}}};
```

但是这样，跟上面的方法 1 几乎是一样的，初始化语句中有许多冗余的`new ListNode`。

如何去掉`new ListNode`呢？

可以使用一个辅助类`ListBuilder`把`new ListNode`“包”起来：

```cpp
struct ListBuilder {
    int v;
    ListNode* p;
    ListBuilder(ListNode* p = nullptr): p(p) {}
    ListBuilder(int d, ListBuilder b = ListBuilder())
        : p(new ListNode(d, b.p)) {}
    operator ListNode* () const {
        return p;
    }
};
```

用法是这样：
```cpp
ListNode* list = ListBuilder {1, {2, {3}}};
```

当编译器遇到 `ListBuilder {1, {2, {3}}}` 时，认识到这是一个列表初始化，
会在 ListBuilder的构造函数中寻找匹配函数签名`(int, ...)`的函数。

`ListBuilder(int d, ListBuilder b = ListBuilder())` 被匹配到。

`{1, {2, {3}}}`中的`1`被用于初始化参数`d`,`{2, {3}}`被用于初始化参数`b`。

因为`b`也是`ListBuilder`，上述过程会递归地进行下去，直到遇到`{3}`，递归终止。

最后，`ListBuilder {1, {2, {3}}}`初始化完成，得到一个`ListBuilder`实例，将它赋值给`ListNode* list`。

为了实现从`ListBuilder`到`ListNode`的转换，需要类型操作符重载：

```cpp
    operator ListNode* () const {
        return p;
    }
```

# 练习

已知二叉树节点定义如下：

```cpp
class TreeNode {
public:
    int val;
    TreeNode *left;
    TreeNode *right;
public:
    TreeNode(int x, TreeNode* l = nullptr, TreeNode* r = nullptr)
        : val(x), left(l), right(r) {}
};
```

如何用 C++ 初始化出二叉树？

