title: 解析 C++ Hello World
tags: []
categories:
  - C++
date: 2016-03-10 16:01:00
---
## Hello world

Hello world 几乎是每个人学习一门语言时写的第一个程序。

C++ 的 Hello world 是这样的：

```cpp
#include <iostream>
using namespace std;
int main() {
    cout << "Hello world!" << endl;
    return 0;
}
```

这段代码虽然只有5行(不算上最后一个大括号的话)，但却隐藏着许多C++知识。接下来，让我们一行行的解析这段代码。


## 第一行

第一行的意思是包含`<iostream>`这个头文件。 它本身是一段预处理指令。

相关标准描述位于《ISO IEC 14882》（C++11标准文档 ）的 `16.2 Source file inclusion [cpp.include]`。

`<...>`是指在系统头文件路径中查找。可用以下命令显示系统头文件路径。

```sh
g++ -E -x c++ - -v < /dev/null
clang++ -E -x c++ - -v < /dev/null
```


## 第二行

第二行是引入`std`命名空间。

相关标准描述位于《ISO IEC 14882》（C++11标准文档 ）的`7.3.4 Using directive [namespace.udir]`。

在实际工程中，直接`using namespace std;`应被禁止，因为`std`命名空间这会污染当前命名空间。


## 第三行

第三行定义了`main`函数，`main`函数是程序的入口。

相关标准描述位于《ISO IEC 14882》的`3.6.1 Main function [basic.start.main]`。

标准规定，`main`函数不可预定义，不可重载，允许的函数原型至少包括：

```cpp
int main();
int main(int argc, char* argv[]);
```


## 第四行

这一行是包含知识点最多的一行。

### C++ I/O模型

`cout` 在`<iostream>`中只有声明：
```cpp
extern ostream cout;
```

其类型为`ostream`，`ostream`是一个alias： `typedef basic_ostream<char> ostream`。

`basic_ostream`的继承结构如下：
![](/images/analyse-cpp-hello-world/basic_ostream.png)

### 运算符重载

`cout <<`是C++的运算符重载语法。

第四行用到了`<ostream>`中定义的两个`<<`运算符重载函数：

```cpp
template<class traits>
basic_ostream<char,traits>& operator<<(basic_ostream<char,traits>& out,
     const char* s);

basic_ostream<charT,traits>& basic_ostream<charT,traits>::operator<<
    (basic_ostream<charT,traits>&(*pf)(basic_ostream<charT,traits>&));
```

这两个函数位于`std`命名空间下。

这一行等价于：

```cpp
operator<<(operator<<(cout, "Hello world"), endl);
```

### ADL(Argument-dependent name lookup)

如果我们去掉第二行`using namespace std;`，那么这一行就得写成：

```cpp
std::cout << "Hello world" << std::endl;
```

等价于：

```cpp
operator<<(operator<<(std::cout, "Hello world"), std::endl);
```

那么问题来了，我们知道这里用到的两个运算符重载函数位于`std`命名空间，我们没有引入`std` 命名空间，编译器是如何找到它们的呢？

答案是ADL (Argument-dependent name lookup)。

相关标准描述位于《ISO IEC 14882》的 `3.4.2 Argument-dependent name lookup [basic.lookup.argdep]`。

ADL, 又称Koenig查找。当一个函数调用中被调用函数名前面没有类、命名空间等作用域限制时，则称之为无限定域的函数调用。
当编译器对其进行无限定域的函数调用名字查找时，如果一般的名字查找失败，会在函数参数的命名空间中继续查找。

### I/O manipulator

`std::endl`是一个 I/O manipulator。

相关标准描述位于《ISO IEC 14882》的 `27.7.3.8 Standard basic_ostream manipulators [ostream.manip]`

其函数原型为：

```cpp
template <class charT, class traits>
std::basic_ostream<charT, traits>& endl(std::basic_ostream<charT, traits>& os);
```

`cout << endl` 会调用 `basic\_ostream<charT, traits>` 的运算符重载成员函数：

```cpp
typedef basic_ostream<charT, traits> ostream_type;
ostream_type& operator<<(ostream_type& (*pf)(ostream_type&))
{
    return pf(*this);
}
```

`std::endl`的定义是这样子的：

```cpp
template<typename _CharT, typename _Traits>
inline basic_ostream<_CharT, _Traits>& endl(basic_ostream<_CharT, _Traits>& __os)
{ return flush(__os.put(__os.widen('\n'))); }
```

经过两层函数调用，`cout << endl` 变成了 `flush(cout.put(cout.widen('\n')))`。

## 第五行

第五行的意思是将 0 这个值作为`main`函数的返回值返回。

相关标准描述位于《ISO IEC 14882》的`3.6.1 Main function [basic.start.main]`的第5条。

这条语句有两个作用：
1. 结束`main`函数。
2. 使用返回值调用`std::exit`。
根据标准，当这条语句位于`main`函数最末尾时可以省略。
