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
- Particle Update 更新粒子状态 比如 Scale Color 设置颜色的变化，Scale Size 设置大小的变化，Color 可以设置颜色，可以设置为 Color from Curve，Force 是给粒子添加力，力可以设置为 Random Range Vector 为不同粒子添加范围内随机的力、Random Vector 是随机方向，Point Attraction Force 将所有的粒子都吸引到某一点，Point Force 排斥力，Curl Noise Force 卷曲噪声力，卷曲噪声向量场，每一个粒子都有一个特定的力方向，可以拿来做流体，Drag 直接将粒子的速度或旋转进行减速，应该就是摩擦力，Gravity Force 给粒子添加上重力，Collision 碰撞，SubUV Animation，Acceleration Force 加速度，Vortex Velocity 漩涡力，可以拿来做黑洞，Update Mesh Orientation 可以更改 Mesh 的方向
- Particle Spwan 添加 Add Velocity 可以设置粒子的初始速度，可以添加 Location，在特定形状内生成粒子，System Location 原点附近，Static Mesh Location 根据静态网格体的位置产生粒子，Skeletal Mesh Location 在骨骼处创建粒子，并且可以过滤需要产生粒子的骨骼，Initial Mesh Orientation 初始化网格的朝向，Sample Texture 纹理采样 SampledColor 获取采样的颜色，Kill Particles 根据条件销毁粒子 Set Bool by Float Comparison 通过比较消除粒子
- 在 Niagara 中可以创建参数，Position 可以获取粒子位置，SimulationPosition 可以获取发射器位置，然后在其他属性中可以使用新建的参数
- 想要创建火焰效果，可以新建一个半透明材质，选择一个纹理，然后将纹理的 Alpha 设为 Opacity，Particle Color 作为 Emissive Color
- 在 Sprite Renderer 中可以设置粒子的材质
- 模拟动态烟雾时，需要结合不同的 UV 纹理图，设置 Sub UV 中的 Sub Image Size
- 可以新建 Module 自定义方法，在模块输入添加想要的输入
- Audio Oscilloscope 音频示波器，Sample Audio Buffer 可以获取 Amplitude 作为音量
- Emitter Properties 中的 Local Space 代表粒子的坐标系是局部坐标系
- Greater Than
- Position 获取粒子位置
- 蓝图与粒子系统通讯 在 User Exposed 处添加参数供粒子系统使用，然后在蓝图中加上 NiagaraSystem，用 Set Niagara Variable 设置对应参数
- Render Light Render 光线渲染器可以使粒子成为光源，能够照亮环境，可以用来做闪电
- 如果要用 Mesh 作为粒子，需要添加 Mesh Renderer
- Velocity 获取粒子的速度
- 蓝图中添加 Niagara Actor 变量对其修改参数，可以利用排斥力形成根据角色位置产生排斥力的效果
- Static Beam 条，必须是 CPU，Beam Emitter Setup 可以设置起始位置和结束位置
- Generate Location Event 需要打开 Persistent ID，Receive Location Event
- Spwan Particles in grid 在网格中生成粒子，Normalized Array Location 网格的数组位置
- Particle Attribute Reader 粒子属性阅读器，Get Vector by index

## 小结

粒子系统是比较清晰简单的，设定发射器和粒子的更新状态，添加力和其他效果，可以模拟烟雾、闪电等效果，还可以自定义模块，但是总感觉还是差了点什么，离想要的效果还比较远🤔

## References

- [UE4 材质入门](https://www.bilibili.com/video/BV1GJ411j7d4)