---
layout: post
title: "Blender To UE"
description: ""
date: 2023-1-10
feature_image: images/2023.1.10/2.png
tags: [Unreal, Blender]
---

想来玩一下这个事情，把 blender 中的部分 shader 移植到 UE 中，但由于我 blender 材质系统完全不熟悉，UE 的材质也只会一点皮毛，所以这中间需要很多技术填补才能实现，暂且留个坑吧。

<!--more-->

## Grease Pencil Tree in Blender

- 使用 Sapling Tree Gen 生成树，然后 Conver To Mesh and Grease Pencil

![](../images/2023.1.10/2.png)

- 删掉 Lines Layer，uncheck Use Lights，在材质面板设置 Fill 的颜色并调整背景颜色 

![](../images/2023.1.10/3.png)

- 选择笔刷、设置颜色，选择模式为 Stroke 和 View，就可以绘制了，在材质中将 Alignment 设置为 Fixed，随便画一画就可以得到手绘的效果啦（但这个好像没办法直接移植到 UE 中）

![](../images/2023.1.10/4.png)



## Blender To UE

- 首先下了一个 Blender 文件，里面的材质效果如下，很可爱的小妹妹：

![](../images/2023.1.10/0.png)

- 试用了一下 UE 新出的插件 blende2ue，将 模型传到 UE 中，发现他不会对材质做特殊处理，就是默认的材质，所以皮肤没有次表面散射，眼球的渲染也是不对，其他部分的材质也没有 blender 中有质感

![](../images/2023.1.10/1.png)

- 那么首先来看看 blender 中的皮肤材质



## 小结

## References