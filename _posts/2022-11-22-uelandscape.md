---
layout: post
title: "UE Landscape"
description: ""
date: 2022-11-22
feature_image: images/2022.11.22/0.png
tags: [Unreal]
---

<!--more-->

## OpenLand

- Openland 提供了自动地形材质，看起来还不错，可以调整很多参数
- 想要快速生成不规则地形，可以使用 Landmass，需要首先开启 landscape 的 edit layer，然后在 sculptor 中选择 blueprint 添加，可以在 PaitLayer 中添加地形 Layer 混合，创建 Layer 时需要选择 non-weight
- 要想添加草地，只需要在 GT_Ground 中添加想要的草 mesh 即可，说实话好漂亮，这么好用的插件为什么才 100 star！

![](../images/2022.11.22/0.png)

- 地形材质贴图也是可以修改的，同时还提供了 Smart Mask 可供修改 Mask 范围以生成植被
- 我有一点好奇，我想打开一个看起来并不复杂的树木资源，但是这个 loading 始终卡在 100%，可能 10 分钟都没有动静，还重启过 UE 和电脑也不行，在 UE 下遇到这个问题很多次了，打开一个东西死活打不开，但是电脑明明处于负载很低的状态，有点好奇这是什么原因？

![](../images/2022.11.22/1.png)

![](../images/2022.11.22/3.png)

![](../images/2022.11.22/2.png)

- 使用 Openland Blend 可以利用 RVT 做材质混合
- 开启 RVT 可以提升性能
- 创建道路需要新建 Layer 添加 Splines，按住 Ctrl + 鼠标左键添加路径点

![](../images/2022.11.22/4.png)

- 新建自己的材质 Layer 需要新建 Material Function，然后添加 Layer Blend


## 小结

## References

- [Open Land & Unreal Engine - Landscape Basics](https://www.youtube.com/watch?v=s9w_WapqTLg&t=211s)