---
layout: post
title: "利用 priority_queue 实现堆"
date: 2022-09-28
feature_image: images/2022.8.21/0.png 
tags: [C++]
---

<!--more-->

## priority_queue

priority_queue 又称优先队列，本质上是一个堆，默认是大顶堆，如果想要小顶堆，需要在声明的时候加上两个参数 `priority_queue<int,vector<int>,greater<int> >que`，同时也可以自定义比较方法

```
// 重载 < 运算符，实现大顶堆
//（堆顶元素：先根据x，选择x最大的元素；若两个元素的x值相同，再根据y，选择两者中y较大的元素） 
bool operator<(My_Type a,My_Type b)
{
    // 定义排序规则 
    if(a.x==b.x) return a.y<b.y;
    return a.x<b.x; 
}
// 定义优先队列
priority_queue<My_Type>que; 

// 仿函数，实现大顶堆 
struct cmp
{
    // 定义排序规则 
    bool operator() (My_Type a,My_Type b )
    { 
        if(a.x==b.x)return a.y<b.y;
        return a.x<b.x; 
    }
}; 
// 定义优先队列
priority_queue<My_Type,vector<My_Type>,cmp>que;
```