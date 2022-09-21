---
layout: post
title: "emplace vs push"
date: 2022-09-21
feature_image: images/2022.8.21/0.png 
tags: [C++]
---

<!--more-->

##  queue::emplace vs queue::push

emplace 可以直接传入构造对象需要的元素，然后自己调用其构造函数，push 是只能直接传入对象或者在传入时构造对象

意思是 emplace 这样接受新对象的时候，自己会调用其构造函数生成对象然后放在容器内，而 push 只能让其构造函数构造好了对象之后，再使用复制构造函数，所以 emplace 相对于 push，直接传入构造对象需要的元素会更节省内存

比如

```C++
string s1 = "test";
queue<pair<string, int>> q;
q.emplace(s1, 0); // emplace 会直接构造 pair，而 push 不可以
q.push(pair{s1, 0}); // push 需要提前构造好对象
```
