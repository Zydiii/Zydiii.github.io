---
layout: post
title: "C++20 —— 二进制神器 popcount"
description: "可以返回数字二进制中含有 1 的个数"
date: 2022-08-23
feature_image: images/2022.8.21/0.png 
tags: [C++20]
---

<!--more-->

## std::popcount

返回值中 1位的个数

```C++
#include <bit>
#include <bitset>
#include <cstdint>
#include <iostream>
 
int main()
{
    for (const std::uint8_t i : { 0, 0b11111111, 0b00011101 }) {
        std::cout << "popcount( " << std::bitset<8>(i) << " ) = "
                  << std::popcount(i) << '\n';
    }
}
```

- `std::bitset<N>` 表示一个 N 位的固定大小序列，是标准库中的一个存储 0/1 的大小不可变容器
- `0b` c++14 带来了 `0b` 或者 `0B` 开头表示二进制串的字面常量方式