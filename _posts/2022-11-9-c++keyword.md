---
layout: post
title: "C++ 中的一些关键字"
date: 2022-11-09
feature_image: images/2022.8.21/0.png 
tags: [C++]
---

<!--more-->

## explicit

- 指定构造函数或转换函数 (C++11起)为显式, 即它不能用于隐式转换和复制初始化.
- explicit 指定符可以与常量表达式一同使用. 函数若且唯若该常量表达式求值为 true 才为显式. (C++20起)

```C++
class Point {
public:
    int x, y;
    explicit Point(int x = 0, int y = 0)
        : x(x), y(y) {}
};

void displayPoint(const Point& p) 
{
    cout << "(" << p.x << "," 
         << p.y << ")" << endl;
}

int main()
{
    displayPoint(Point(1));
    Point p(1);
    displayPoint(1); // 不加 explicit 能运行，会触发隐式调用
    Point p = 1; // 不加 explicit 能运行，会触发隐式调用
}
```

## static_cast & dynamic_cast

- static_cast相当于传统的C语言里的强制转换，该运算符把expression转换为new_type类型，用来强迫隐式转换如non-const对象转为const对象，编译时检查，用于非多态的转换，可以转换指针及其他，但没有运行时类型检查来保证转换的安全性
- dynamic_cast主要用于类层次间的上行转换和下行转换，还可以用于类之间的交叉转换（cross cast）。在类层次间进行上行转换时，dynamic_cast和static_cast的效果是一样的；在进行下行转换时，dynamic_cast具有类型检查的功能，比static_cast更安全。dynamic_cast是唯一无法由旧式语法执行的动作，也是唯一可能耗费重大运行成本的转型动作