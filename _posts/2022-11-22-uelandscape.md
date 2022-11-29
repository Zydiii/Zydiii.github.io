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
- 要想添加草地，只需要在 GT_Ground 中添加想要的草 mesh 即可，说实话好漂亮，这么好用的插件为什么才 100 star！Remove Grass Layer 可以取消绘制草地

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
- 不知道发生了什么草地变成了这样，像是开启了什么优化，但是不记得有开过这个设置，而且第一次打开 UE 时还是正常的，之后就不对了，没 run 的时候草地也是正常的，run 起来就不对了。好像是我开了 Hardware Ray Tracing，关掉就很正常，好奇怪。

![](../images/2022.11.22/5.png)

- RunTime Virtual Output 可以拿来做 Layer Blend，新建 RVT Volume 写入 RVT Material 和 RVT Height，在需要混合的物体材质中添加 OpenLand RVT Blend With Attribute 节点即可和地形混合材质
- 其他材质要添加脚印效果需要将 Base Color、Normal、Roughness 写入 OpenLand RVT Prepare Output 和 Runtime Virtual Texture Output，Read Openland Deform Data 生成新的 Normal 和 Height
- OpenLand RVT Sample 输出的 ShowGrass 作为草材质的 Opacity Mask 可以隐藏一些不想要的草

## Ultra Dynamic Weather

- 利用这个插件可以添加晴天、下雨、雷雨、下雪、沙尘暴、雾天等天气，很强，还可以实时修改地形材质，以及为角色添加粒子效果
- 要添加天气支持，需要在 OpenLand 材质中添加 UDW Support 节点，以及添加 RVT Invalidator

![](../images/2022.11.22/6.png)

![](../images/2022.11.22/7.png)

- Change Weather 节点可供在蓝图中修改天气，也可以设置随机天气
- 雪地自带了 Trail 效果，但是我没有找到可以自定义 Trail 形状的，用了它的雪地就不能用 OpenLand 的脚印了，感觉可能会有问题

## Tank

- Tank 的运动可以用 Chaos Vehicle 来做，还要修改 Physical Asset，不过我们应该不需要控制车辆的运动，反而是根据别人给好的路径点更新位置，这个需要考虑插值问题
- 如果要弄 Chaos，需要调整坦克车模型的骨架，为其创建 Physic Asset，还挺复杂的...
- 同时如何将导航数据映射到车辆上？可能车辙印也会有问题
- 我觉得也许不需要用 Chaos，而是用 UE 的导航系统，因为到时候发过来的也是导航点，用 UE 自带的导航系统也许可以比较方便的让车行驶过去
- 周末还是试了一下 Chaos 发现还是不能正常控制车辆移动，所以暂定方案还是就用 CharacterMovement + NavAI 控制坦克移动吧，需要学一下 CharacterMovement，不过现在最关键的问题还是雪地的脚印，似乎需要将交互点调高一点脚印比较正常，但是这个不能修改印记样式，并且脚印只有在雪地里才有，我试了一下如果用 Openland 的 deform，在雪地里他不能改 normal，看起来比较奇怪，我把 OpenLand_UDW_Support 的 Normal 输出改了，就用的 Openland 原来的输出，没有从 UDW 中拿，这样看 normal 正常了许多

![](../images/2022.11.22/8.png)


- 同时不知道为什么二者都会存在部分地形无法显示印记的问题，感觉和检测有关系，并且 RVT 闪烁的很厉害。感觉也许需要学一下自己弄 trails，不用他自带的 RVT


## Trails














## 小结

## References

- [Open Land & Unreal Engine - Landscape Basics](https://www.youtube.com/watch?v=s9w_WapqTLg&t=211s)
- [UE4 Tutorial: Tank Tracks with Vigliante Free Assets Part 1/2](https://www.youtube.com/watch?v=qoW_LE_V5ms&t=553s)
- [UE5 Tutorial - Chaos Tank Blueprint](https://www.youtube.com/watch?v=i8RR5kNouwI&t=1038s)
- [Setup drivable vigilante chaos vehicle - Tutorial Unreal Engine 5](https://www.youtube.com/watch?v=tR3sSMD-D0E)