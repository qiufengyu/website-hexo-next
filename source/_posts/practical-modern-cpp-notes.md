---
title: Practical Modern C++ Notes
date: 2019-06-27 16:12:00
categories: [Programming Languages]
tags: [C++, Practical C++, Modern C++]
description: C++，尤其是 C++11 以来，引入了一些新的特性，除此以外，本文还记录了设计模式、推荐实践方案、常见技巧等，包括一些 Code Snippet。
---

### 善用 `const`

* 对于一个 class 的成员函数来说，如果不涉及对 private member 的改变，应该加上 `const` 修饰符。应该在设计之初对是否使用 `const` 进行充分考虑，因为后期再考虑加上 `const` 会有难以预估的连锁反应。
* 在函数传参时，使用 `const &`  可以避免对一些复杂对象的拷贝过程，提高效率。单纯的引用，可能在函数执行过程中对传入的对象进行修改，通常这是一种不希望发生的行为，尽量应该加上 `const` 修饰。
* `const` 修饰符的位置：[Should I put "const" before or after the type?](http://www.stroustrup.com/bs_faq2.html#constplacement)
  * 对于一般的 T 类型的变量声明而言，`const T` 和 `T const` 没有区别。前者比较易读，但似乎后者现在较为推荐
  * 以对象 A 为例，写成 `A const &` 和 `const A &` 没有区别，但是 `A & const` 中的 `const` 是一种冗余的写法
  * `const` 修饰指针有一些不同，采用的是 **从右到左** 结合的方式：
```c++
int * const p1 = q;	// constant pointer to int variable，指针不可变
int const * p2 = q;	// pointer to constant int，int 不可变
const int * p3 = q;	// pointer to constant int，int 不可变
```
### 初始化成员列表

* 可以使用初始化成员列表对类中的成员进行初始化
* 且对于子类而言，可以在初始化列表中，直接使用父类构造函数 

### 宏 `#pragma once`

* 首先，推荐在 `.h` 头文件中进行常量、函数、类等的声明，且头文件中尽量不要用 `using namespace std;` 等名空间作用域语句，可以使用 `using std::vector;`，在 `.cpp` 文件中进行具体的实现
* 为了避免头文件被多次 include，`#pragma once` 可以加在`.h` 头文件的开头，基本等价于
```c++
#ifndef __HEADER_H__
#define __HEADER_H__
... ... // 声明、定义语句
#endif
```

### 简单模板

* Function templates，函数模板
```c++
// declare
template <class T>
T GetMax (T a, T b) {
 return (a>b?a:b);
}

// use
int x,y;
GetMax <int> (x,y);    // <int> usually can be automatically found out
```

* Class templates，类模板
```c++
template <class T>
class mypair {
    T values [2];
  public:
    mypair (T first, T second)
    {
      values[0]=first; values[1]=second;
    }
};

```

* [Template specialization](https://zh.cppreference.com/mwiki/index.php?title=cpp/language/template_specialization)，模板特化
```c++
// class template:
template <class T>
class mycontainer {
    T element;
  public:
    mycontainer (T arg) {element=arg;}
    T increase () {return ++element;}
};

// class template specialization:
template <>
class mycontainer <char> {
    char element;
  public:
    mycontainer (char arg) {element=arg;}
    char uppercase ()
    {
      if ((element>='a')&&(element<='z'))
      element+='A'-'a';
      return element;
    }
};
```
 这里下半部分是模板特化的实现方式，对于 char 类型有一种专门的处理逻辑。

### 析构函数、拷贝构造函数与赋值操作符重载
* 在包含（传统）指针成员的类中：
  * 析构函数中通常需要 `delete` 由 `new` 创建的内容
  * 拷贝（复制）构造函数 `ClassName ( const ClassName & )`，则由 `new` 根据传入的现有对象创建
  * 赋值操作符重载 `ClassName & ClassName :: operator= ( const ClassName & ) `，首先需要 `delete` 旧对象的原有内容，再重新 `new` 申请新的内容
  * 对于拷贝构造函数、赋值操作符重载函数而言，加上 `=default` 可以强制编译器生成对应的函数，而不是使用隐式的函数，加上 `=delete`，可以避免隐式拷贝构造或复制赋值
  * 移动构造函数 `ClassName ( ClassName && )`，从同类型的右值初始化对象时，调用移动构造函数
  * 移动赋值操作符重载 `ClassName & ClassName :: operator= ( ClassName && )`，这两种主要涉及使用 `std::move() `的情况。
  * 推荐使用 `shared_ptr` 等智能指针进行内存管理！


### 基类和派生类：
* 类对象作为参数传参，尽量使用引用或指针，可以避免拷贝开销，并考虑 `const` 原则
* 基类的实例对象不能赋值给派生类，但派生类的实例可以赋值给基类，会进行 slicing，丢失派生类的一部分信息
* 要使用多态（polymorphism），要满足：
  * 对象一定是引用或者是指针类型：基类的指针可以指向派生类的对象，但是该基类指针无法访问派生类独有的成员，派生类的指针无法指向基类对象
  * 基类的成员函数声明为 `virtual`，通常来说，基类的析构函数是需要加上 `virtual` 的，且根据继承规则，基类的析构函数仍旧会被调用
* 在基类和派生类指针的类型转换中，尽量使用 `static_cast<>` 或 `dynamic_cast<>`，后者转换失败会返回 `nullptr`，更为安全
