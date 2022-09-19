---
layout: post
title: "Unreal Engine 5 —— 入门篇（二）"
description: ""
date: 2022-09-19
feature_image: images/2022.8.24/0.jpg
tags: [Unreal]
---

<!--more-->

## 一些材质相关知识点

- 取消连线时按下 Alt + 鼠标左键，vector3 快捷键是按下 3 + 鼠标左键，Multiply M + 鼠标，Lerp L + 鼠标
- BlendMode 中可以选择物体材质是否透明，透明物体常见的做法是以一张 Mask 贴图的输出作为不透明度，材质类型选择透明模式
- 世界场景位置偏移是对网格体顶点进行操作，VertexNormalWS 可以获取顶点法向量
- Sine_Remappeed 将值按照 Sin 映射
- 常数节点 Constant 一维、二维、三维、四维，拆分多维向量 BreakOutFloat2Components、BreakOutFloat3Components、BreakOutFloat4Components，组合多维向量 MakeFloat2、MakeFloat3、MakeFloat3，提取分量 ComponentMask
- 参数节点 可以将常数节点转变为参数节点，有向量参数、布尔参数，在蓝图中可以通过 Set Scalar Parameter Value on Materials 修改材质参数
- 布尔参数可以接 Switch
- 数值运算 Add Substract Multiply Divide 1-x Abs Ceil Floor Clamp Fmod
- 向量运算 Dot 获取从顶点到相机的向量 Cemera Vector，Append 也可以用来构造向量，Cross，Panner 平移运动，Rotator 旋转运动 可以用来改变 UV 输入





## 小结

## References

- [UE4 材质入门](https://www.bilibili.com/video/BV1GJ411j7d4)
