---
layout: post
title: "DirectX11 初探（四）"
description: ""
date: 2022-10-27
feature_image: images/2022.10.27/17.png
tags: [DirectX11]
---

<!--more-->

## Cursor

- 使用 ::ShowCursor 显示或隐藏鼠标 维护了一个 count，当 count 大于等于 0 时会显示鼠标，小于 0 时隐藏鼠标
- 需要对 Imgui 的鼠标响应单独做处理，因为虽然窗口能看不到鼠标了，但是仍然能够和菜单交互，通过设置 ImGui::GetIO().ConfigFlags
- GetClientRect 获取窗口的坐标 rect
- MapWindowPoints 将 (映射) 一组相对于一个窗口的坐标空间的点转换为相对于另一个窗口的坐标空间
- ClipCursor 将鼠标限制在屏幕上的矩形区域内
- WM_ACTIVATE 当窗口切换到最上层会触发该信号
- 要接收鼠标点击事件，需要使用 RegisterRawInputDevices 将 mouse raw input device 进行注册
- 用 WM_INPUT 捕获鼠标点击事件，需要使用 GetRawInputData 获取一个 byte 数组数据

## Camera

- 提供移动、旋转操作，移动时需要朝着相机的正方向移动
- 使用 clamp 防止正上方正下方视角抖动
- 使用 XMMatrixLookAtLH 计算投影矩阵

## Material Loading

修改 Mesh 结构加载纹理，修改 Shader 显示纹理

- 在利用 Specular Map 计算高光时，计算公式为 specularPower = pow(2.0, 13.0 * shininessMap)

![](../images/2022.10.27/0.png)

![](../images/2022.10.27/3.png)

## Bindable Codex

在之前的架构中，相同类型的 Drawable 实例可能会公用一些 Bindable 对象，但是对于不同实例他们之间的 Bindable 是不共享的，所以我们添加上 CodeX 架构，让其存储 Bindable 对象，这样不管我们的绘制实例是什么，都可以去拿到一些自己需要的 Bindable

![](../images/2022.10.27/2.png)

- 拥有一个 map 存储 bindable，每个 bindable 用唯一 string 标识，比较关键的点就是给 Bindable 对象生成唯一标识
- 单例，接口负责存取 bindable 对象
- 因为 Shader 是可以复用的，所以修改 VertexShader，将它的 typeid 和 name 作为唯一 id，添加 Resolve 方法，在 CodeX 中查找是否已经有该 Shader 对象，若无则加到 map 中
- std::static_pointer_cast 将 shared_ptr 指针进行静态指针类型转换，一般用在继承行为中，当子类想要获取父类中的一些属性时，将父类的 shared_ptr 转化为子类的 shared_ptr

## Normal Mapping

- 利用 Normal Mapping 可以在较低数量的面片模型上呈现更多细节
- HLSL 的 bool 有 4byte，在 C++ 中使用 BOOL，其实就是 int
- 使用 sample 从 normal map 中采样，然后将 x y 映射为 -1~1 作为该顶点的 Normal，z 取负数，为什么 z 不需要转到 -1~1 呢，因为 z 为负数的我们看不到，所以不用管 -1~0，取负数是为了让 normal 朝向相机，因为一般 z 轴是向内的（但是这种 mapping 不推荐甚至是不正确的，还是应该把所有分量都映射到 -1~1）同时，因为 DirectX 和 OpenGL 的 Texture Coordinate 的 y 轴是相反的，需要将 y 变成 1~-1

![](../images/2022.10.27/4.png)

![](../images/2022.10.27/5.png)

加上 Normal Map 视觉效果真的提升好大~

![](../images/2022.10.27/6.png)

![](../images/2022.10.27/7.png)

- 但是这里有一个问题，我们从 Vertex Shader 获取到的 Pos 是 view space，但是 normal mapping 得到的 normal 是物体的 object space，所以我们需要将 modelView、modelViewProj 矩阵也传入到 Pixel Shader 中
- 在使用 Object Space 的时候，对于一个立方体来说，单纯地转入物体的观察矩阵并不能体现每个面的法线关系，但是如果贴图用的同一张，那么算出来的法线就会有问题，所以我们需要 Tangent Space（TBN），将 TBN 变换到 view space，而且要记得 normal 需要归一化 normalize

![](../images/2022.10.27/8.png)

![](../images/2022.10.27/9.png)

- CommandLineToArgvW 分析 Unicode 命令行字符串，并返回指向命令行参数的指针数组，以及此类参数计数
- std::tie 可用于解包 std::pair ，因为 std::tuple 拥有从 pair 的转换赋值；std::tie会将变量的引用整合成一个tuple，从而实现批量赋值

## Sponza Crytek Scene

- 终于知道了这个很经典的场景原来是叫 Sponza，导入生成我们的基础场景，好激动！看起来像模像样的，虽然效果上还差了很多~

![](../images/2022.10.27/10.png)

- 可以注意到，这个窗帘看上去会有很多奇怪的纹理，这是由降采样引起的的，当我们离得比较远时，采样点就会变稀疏，比如我们在看一个花丛，靠近时可以看到很多清晰的花，但是当我们远离时，可能就只能看到一团花草，颜色像混在一起了

![](../images/2022.10.27/11.png)

![](../images/2022.10.27/12.png)

- 为了解决这个问题，需要用到 MipMapping，它通过将几块像素合并生成更小的采样图。使用 GenerateMips 生成 MipMap，同时 Sampler 也需要指定 LOD

![](../images/2022.10.27/13.png)

- 当我们的视角平行于地面时，我们会观察到地板好像变糊了，这也是由于采样造成的，在 y 轴，我们的 MipMap 可能已经采样到四级了，但是在 x 轴，我们其实应该按照一级采样，但是仍然采样的四级，所以就像被拉伸了一样有点糊，这个问题可以通过修改 Sampler 的 fliter 为 D3D11_FILTER_ANISOTROPIC 来解决

![](../images/2022.10.27/16.png)

![](../images/2022.10.27/14.png)

![](../images/2022.10.27/15.png)

## Alpha Blending

- 添加一个 Bindable 类 Blender，保存一个 BlendState








## 小结



## References

- [C++ 3D DirectX Tutorial [Cursor Hide / Cursor Confine]](https://www.youtube.com/watch?v=RQTBTfjs7GM&list=PLqCJpWy5Fohd3S7ICFXwUomYW0Wv67pDD&index=36)
