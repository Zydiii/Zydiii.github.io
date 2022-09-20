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
- 数据节点 TexCoord 快捷键：U 输出的是 UV 坐标，绝对世界位置 输出每个顶点的世界坐标位置，object position 物体中心的位置；VertexNormalWS 顶点法线；PixelNormalWS 像素法向量；Object Radius 物体半径；ScreenPosition 随着镜头移动，屏幕的 UV
- 实用工具节点 Lerp，BumpOffset 快捷键 B+鼠标 产生视差贴图、位移贴图、提升凹凸感、障眼法、改变 UV 分布、低成本的增加深度信息，Dapth Fade 深度消退 与其他物体交互时会出现边界融合的效果 适合烟雾 输出透明度，Distance 求两点之间的距离，Fresnel 菲涅尔节点，if frac 取小数 time 时间
- 地形节点 layer blend 添加多种纹理，LandscapeCoords 调整 UV，LandscapeSwitch 根据图层是否使用进行选择输出，landscape visibility mask 可见性模板 可以用来挖洞 高度重叠 以 Alpha 输出作为 Height LandscapeCoords 可以用于缩放
- 粒子节点 Particle Relative Time 粒子相对时间百分比 0~1，Particle Size 二维粒子大小 x、y，Particle Color 粒子颜色 从粒子系统的颜色曲线来的 也有透明度，Particle MacroUV 有粒子的部分就可以看到图片 有 UV 可以拿来做虫洞的效果 一整张图片，Particle SubUV 则是每个粒子有自己的 UV 每个粒子一张图片
- 材质函数 材质节点分为两类：表达式和函数，复杂重复的工作做成函数封装功能 可以有多个输入输出，至少有一个输出，通过设置优先级改变顺序，可以设置默认值 使用时可以直接拖拽或 Expose to Library
- 如果一些贴图值较大，可以使用 Lerp，以贴图的值作为 Alpha，将值映射到合适的范围 TexCoord
- MakeMaterialAttribute 输出材质属性 用于连接材质函数的输出，普通材质需要将输出的使用材质属性勾选上 BlendMaterialAttributes 混合材质属性，输入 Mask 可以给不同部位设置不同材质
- 当贴图在一些位置看起来效果不好时，可以根据 Position 设置 UV，并按照 Normal 混合
- 雪 次表面材质
- HeightLerp  根据高度贴图进行 Lerp，可以上移和下移
- Get Material 获取材质 Create Dynamic Material 获取动态材质实例 Set Scalar Parameter Value 设置材质参数
- 材质 Blend Mode 可以选择 Additive，前面的物体会加上后面的物体颜色
- DistanceToNearestSurface 离最近的表面的距离 PixelDepth 像素深度 SceneDepth 场景深度 SceneDepth-PixelDepth 可以得到水深 Scene Color 获得场景颜色
- ActorPosition 自身位置 Particle Relative Time 粒子的生存时间 Particle Speed 粒子的速度
- BlendAngleCorrectedNormals 混合法线
- 折射效果可以通过移动 ScreenPostion 作为 UV 传入 Scene Color 和 Scene Depth

## 小结

简单学了一些常用的节点，对于 UE 的材质编辑系统有了一个初步的认识，有一个问题比较好奇，要是材质很复杂还会采用这种节点编辑的方式吗，光是看这个水材质混乱的节点和连线我都差点蒙圈了，但是又没太查到直接写代码，接下来主要打算重点研究一下如何创建红外视效、飞机尾焰、炮弹粒子、追踪弹特效等效果，感觉我的毕设也是这个方面，那么今后是要走技术美术方向吗，可是 TA 岗又太卷了，而且我 A 的部分是基本上不太会，如果准备 TA 那么其他普通的开发岗可能也没啥项目，啊啊啊明年要找不到工作了......

## References

- [UE4 材质入门](https://www.bilibili.com/video/BV1GJ411j7d4)
