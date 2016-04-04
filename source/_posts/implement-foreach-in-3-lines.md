title: 3 行 C++ 代码实现 foreach
categories:
  - C++
date: 2016-04-04 15:48:45
tags:
---

# 什么是 foreach

foreach 语句一种用于遍历容器的语句，是现代编程语言的标配。使用 foreach 可以方便地对容器进行遍历操作。下面是几种常见语言的 foreach：

- Python:
    ```python
    lst = [1, 2, 3]
    for i in lst:
        print(i)
    ```
- C#
    ```cs
    int[] lst = {1, 2, 3};
    foreach (int i in lst) {
        Console.WriteLine(i);
    }
    ```
- Java 8
    ```java
    int[] lst = {1, 2, 3};
    for (int i : lst) {
        System.out.println(i);
    }
    ```
- PHP
    ```php
    $lst = array(1, 2, 3);
    foreach ($lst as $i) {
        echo "$i\n";
    }
    ```
# C++ 中的 foreach

C++ 在 C++11 之前没有 foreach，遍历容器主要有以下两种方法：

1. 遍历索引, 这种方法继承自 C 语言。
    ```cpp
    int arr[] = {1, 2, 3};
    std::vector<int> v(arr, arr + 3);
    for (int i = 0; i < v.size(); ++i) {
        std::cout << v[i] << std::endl;
    }
    ```
2. 使用迭代器(iterator)。
    ```cpp
    int arr[] = {1, 2, 3};
    std::vector<int> v(arr, arr + 3);
    for (std::vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
        std::cout << *it << std::endl;
    }
    ```

遍历索引的缺点是缺乏通用性，只能用于支持 `[]` 操作符的容器。

迭代器具有通用性，STL 容器都支持迭代器，缺点则是写起来不方便，迭代器如同裹脚布般又臭又长的类型名容易导致 for 语句长度突破天际。

C++11 为 C++ 增加了 foreach 语句。现在你可以愉快地写出这样的代码：

```cpp
std::vector<int> v {1, 2, 3};   // 使用了 C++11 的列表初始化语法 ;)
for (auto i : v) {              // auto 自动类型推导，与foreach 搭配使用进一步简化语法
    std::cout << i << endl;
}
```

不幸的是，现在 C++11 尚未普及，有时候你只能使用不支持 C++11 的编译器。遇到这种情况，除了使用上面2种写法，你还有一种选择，那就是动用 C++ 中的黑魔法。

下面有请 C++ 大魔法师`Boost`为我们演示初级魔法`Boost.Foreach`：

```cpp
#include <boost/foreach.hpp>

...

int arr[] = {1, 2, 3};
std::vector<int> v(arr, arr + 3);
BOOST_FOREACH(int i, v) {
    std::cout << i << std::endl;
}
```

使用`BOOST_FOREACH`也有缺点，那就是编译需要`Boost`库，为使用带来了麻烦。

如果你不想/不能用`Boost`库，那么就只能自己动手造一个轮子了。

# 如何造 foreach

首先，让我们看下迭代器的写法：
```cpp
for (std::vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
    int i = *it; // 取出元素值
    ...
}
```
我们可以定义一个`foreach`宏，把`foreach (i，v)`这样的语句变成上面的形式：
```cpp
#define foreach(i, v) \
    for (??? it = v.begin(); it != v.end(); ++it) { \
        ??? i = *it;
```
`???`是相应迭代器和元素的类型，很不幸，在 C++11 之前没有办法可以自动推导类型。

为了解决这个问题，我们只能动用GCC编译器的拓展`typeof`，`typeof` 用法类似 `sizeof`，只不过它返回的不是类型大小，而是类型本身。

使用`typeof`后：

```cpp
#define foreach(i, v) \
    for (typeof(v.begin()) it = v.begin(); it != v.end(); ++it) { \
        typeof(*it) i = *it;
```

用起来是这样：

```cpp
int arr[] = {1, 2, 3};
std::vector<int> v(arr, arr + 3);
foreach (i, v)
    std::cout << i << std::endl;
}
```

看上去已经实现了foreach的效果，但。。。最后那个不匹配的花括号逼死强迫症啊！

虽然可以把不匹配的花括号定义成一个宏：
```cpp
#define endforeach }

foreach (i, v)
    std::cout << i << std::endl;
endforeach
```

但理想的写法应该是这样的：
```cpp
foreach (i, v) {
    std::cout << i << std::endl;
}
```

这样就需要宏里不能出现`{`。不用`{`，如何从迭代器里取出元素值呢？

放到`for`语句里？

```cpp
#define foreach(i, v) \
    for (typeof(v.begin()) it = v.begin()，typeof(*it) i = *it; \
        it != v.end(); \
        i = *++it) \
```

不行，`for`语句不能定义两种不同类型的变量。

不过使用`for`语句取出元素值是正确的思路方向，再使用一条`for`语句就可以了：

```cpp
#define foreach(i, v) \
    for (typeof(v.begin()) it = v.begin(); it != v.end(); ++it) \
        for (typeof(*it) i = *it; ???; ???)
```

接下来，我们得让里面的`for`只循环一次。可以用一个`loop`变量实现：

```cpp
#define foreach(i, v) \
    for (typeof(v.begin()) it = v.begin(); it != v.end(); ++it) \
        for (typeof(*it) i = *it; loop; loop = false)
```

那么问题来了，`loop`该定义在哪呢？再用一个`for`？

```cpp
#define foreach(i, v) \
    for (typeof(v.begin()) it = v.begin(); it != v.end(); ++it) \
        for (bool loop = true; loop;) \
            for (typeof(*it) i = *it; loop; loop = false)
```

Great！It works！

# 优化完善

## #1 使用 `if` 代替 `for`

中间的`for`循环可以使用`if`代替：

```cpp
#define foreach(i, v) \
    for (typeof(v.begin()) it = v.begin(); it != v.end(); ++it) \
        if (bool loop = true) \
            for (typeof(*it) i = *it; loop; loop = false)
```

## #2 调整循环顺序

里面的`for`循环只循环一遍，放在里面会打断外部`for`循环的顺序执行。

交换下两者的顺序，再作一点调整：

```cpp
#define foreach(i, v) \
    if (bool loop = true) \
        for (typeof(v.begin()) it = v.begin(); loop; loop = false) \
            for (typeof(*it) i = *it; it != v.end(); i = *++it)
```

## #3 加上括号,变量名混淆

```cpp
#define foreach(i, v) \
    if (bool __loop = true) \
        for (typeof((v).begin()) __it = (v).begin(); __loop; __loop = false) \
            for (typeof(*__it) i = *__it; __it != (v).end(); i = *++__it)
```

加上括号是为了避免类似`foreach(i, *p)`的语句出错，`.`的优先级比大多数运算符高。

变量名混淆是为了避免因为重名而导致问题，就像这样：
```cpp
bool loop = false;
foreach (i : lst) {
    if (i) {
        loop = true; // 这里修改的不是3行前的loop
    }
}
```

## #4 变成 3 行 ;P

```cpp
#define foreach(i, v) if (bool __loop = true) \
    for (typeof((v).begin()) __it = (v).begin(); __loop; __loop = false) \
        for (typeof(*__it) i = *__it; __it != (v).end(); i = *++__it)
```

# 总结

就这样，我们使用 3 行代码就实现了类似`BOOST_FOREACH`的功能，是不是很 <ruby>简单<rt>hei mo fa</rt></ruby> 呢。

<br/> <br/> <br/> <br/> <br/> <br/> <br/> <br/>
<br/> <br/> <br/> <br/> <br/> <br/> <br/> <br/>
<br/> <br/> <br/> <br/> <br/> <br/> <br/> <br/>

<span style='font-size:40px'>
(╯‵□′)╯︵┴─┴ 简单个毛啊，C++11 大法好，快用 C++11 啊！
</span>

