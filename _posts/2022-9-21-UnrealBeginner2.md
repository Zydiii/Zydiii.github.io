---
layout: post
title: "Unreal Engine 5 —— 入门篇（三）"
description: ""
date: 2022-09-21
feature_image: images/2022.8.24/0.jpg
tags: [Unreal]
---

这一篇主要针对粒子系统，来学习如何做出一些特别的粒子效果。个人感觉还是不能走 TA，就业面太窄了，可是其实不管什么岗位好像感觉都没有什么拿得出手的项目，怎么办呢......

<!--more-->

## Niagara

- Properties 设置发射器在 CPU 还是 GPU 上运行
- Emitter Update 设置发射器生命周期，比如 Loop Behavior 可以控制生成粒子的次数，Spawn Rate 设置每秒生成的粒子数量，Emitter State 中可以设置 Life Cycle 由 System 控制
- Initialize Particle 初始化粒子，比如设置生命周期、颜色、大小
- Particle Update 更新粒子状态 比如 Scale Color 设置颜色的变化，Scale Size 设置大小的变化，Color 可以设置颜色，可以设置为 Color from Curve，Force 是给粒子添加力，力可以设置为 Random Range Vector 为不同粒子添加范围内随机的力、Random Vector 是随机方向，Point Attraction Force 将所有的粒子都吸引到某一点，Point Force 排斥力，Curl Noise Force 卷曲噪声力，卷曲噪声向量场，每一个粒子都有一个特定的力方向，可以拿来做流体，Drag 直接将粒子的速度或旋转进行减速，应该就是摩擦力，Gravity Force 给粒子添加上重力，Collision 碰撞，SubUV Animation，Acceleration Force 加速度，Vortex Velocity 漩涡力，可以拿来做黑洞
- Particle Spwan 添加 Add Velocity 可以设置粒子的初始速度，可以添加 Location，在特定形状内生成粒子，System Location 原点附近
- 在 Niagara 中可以创建参数，Position 可以获取粒子位置，SimulationPosition 可以获取发射器位置，然后在其他属性中可以使用新建的参数
- 想要创建火焰效果，可以新建一个半透明材质，选择一个纹理，然后将纹理的 Alpha 设为 Opacity，Particle Color 作为 Emissive Color
- 在 Sprite Renderer 中可以设置粒子的材质
- 模拟动态烟雾时，需要结合不同的 UV 纹理图，设置 Sub UV 中的 Sub Image Size
- 可以新建 Module 自定义方法，在模块输入添加想要的输入
- Audio Oscilloscope 音频示波器，Sample Audio Buffer 可以获取 Amplitude 作为音量
- Emitter Properties 中的 Local Space 代表粒子的坐标系是局部坐标系
- Greater Than
- Position 获取粒子位置

## 小结

## References

- [UE4 材质入门](https://www.bilibili.com/video/BV1GJ411j7d4)