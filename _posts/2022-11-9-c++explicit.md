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