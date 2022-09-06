---
layout: post
title: "JSBSim To Unreal"
description: ""
date: 2022-08-26
feature_image: images/2022.8.24/0.jpg
tags: [Unreal, JSBSim]
---

<!--more-->

要想在 Unreal 中使用 JSBSim 官方已经有一套解决方案，但是，JSBSim 提供的 aircraft 有限，我尝试把那边需求的 Flightgear 中的 F-35B 的配置文件加进去之后，Unreal 会直接闪退，可是报错中没有发现有效信息......

后来经过排查，我发现问题出现在 `<property>` 这个语句中。由于我并不会 JSBSim，所以不是很能理解，粗略看了一下官方手册，这个 `<property>` 相当于全局变量，而正确的 .xml 文件的 property 都是可以在官方手册中找到的，出错的那些文件找不到，我估计是 Flightgear 自己加的，但是怎么加上去的，目前不清楚。

而官方的这些 property 根据我的观察是需要在 C++ 代码里面绑定的：`PropertyManager->Tie()`，所以在 UE 中可能是因为找不到这个 property 从而触发了空指针之类的错误。

而这些 property 是为 flight_control 服务的，也就是飞行控制的核心逻辑了，但是我没看懂这些 flight_control 是在哪里定义的，还是说他们自己就代表了一种函数？但是我把官方示例中的 flight_control copy 过来运行还是会崩溃，甚至直接把这个 block 删掉也不行，但是我在官方的 xml 文件把 flight_control 这块删掉也仍然可以正常运行，这意味着其他部分可能也存在问题......

看来直接套用是不太行的，得把 JSBSim 的基本工作流搞明白才能自由添加自己想要的配置。由于锅还没有分到我头上，我目前不打算继续研究了，之后有具体的任务之后再来参考一下这个 debug 经验......

## References

- [A DIY Flight Simulator Tutorial](https://dev.epicgames.com/community/learning/tutorials/mmL/a-diy-flight-simulator-tutorial)
- [jsbsim/UnrealEngine/](https://github.com/JSBSim-Team/jsbsim/tree/master/UnrealEngine)