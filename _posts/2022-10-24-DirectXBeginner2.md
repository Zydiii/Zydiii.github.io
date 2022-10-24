---
layout: post
title: "DirectX11 初探（三）"
description: ""
date: 2022-10-24
feature_image: images/2022.10.24/0.png
tags: [DirectX11]
---

<!--more-->

## Graphics

### Init

- HWND：窗口句柄
- SwapChain 实现了一个或多个 surface 用来存储输出到输出设备之前的各种渲染数据，使数据后台加速，这样一帧绘制好了才会将完整的帧显示在屏幕上，前后缓冲区形成了一个 Swap Chain，用 IDXGISwapChain 接口表示，该接口保存了前后缓冲区纹理，并提供了用于调整缓冲区尺寸的方法和呈现方法
- DXGI_SWAP_CHAIN_DESC 描述了一个 Swap Chain，BufferDesc 描述了我们要创建的后台缓冲区的属性，宽度、高度、像素格式，SampleDesc 描述了多重采样数量和质量界别，BufferUsage 设为 DXGI_USAGE_RENDER_TARGET_OUTPUT 即将场景渲染到后台缓冲区，BufferCount 是后台缓冲区的数量，OutputWindow 要渲染到的窗口的句柄，Windowded 窗口模式，SwapEffect 设为DXGI_SWAP_EFFECT_DISCARD，让显卡驱动程序选择最高效的显示模式，Flags 可选的标志值。如果设为DXGI_SWAP_CHAIN_FLAG_ALLOW_MODE_SWITCH，那么当应用程序切换到全屏模式时，Direct3D会自动选择与当前的后台缓冲区设置最匹配的显示模式。如果未指定该标志值，那么当应用程序切换到全屏模式时，Direct3D会使用当前的桌面显示模式。
- 在试验 BufferDesc.Format 其他类型时有的不能正常绘制，没有找到原因
- ID3D11Resource interface：提供所有资源的常规操作
- __uuidof 获取某种结构、接口及其指针、引用、变量所关联的 GUID，类似于 typeof
- ID3D11Texture2D 的 BindFlag 决定了绑定到 pipeline 的方式，比如 D3D11_BIND_DEPTH_STENCIL 是说把该纹理作为 depth stencil target
- D3D 描述资源时，用的是 Model-View 模式，ID3D11Resource是Model，负责数据创建销毁、内存布局之类的功能，View的话有以下几种：
  - ID3D11ConstantBufferView（CBV），表示Resource是只读的Buffer。
  - ID3D11ShaderResourceView（SRV），表示Resource是只读的Texture。
  - ID3D11RenderTargetView（RTV），表示Resource是只写的RenderTarget。
  - ID3D11DepthStencilView（DSV），表示Resource是只写的DepthStencil。
  - ID3D11UnorderedAccessView（UAV），表示Resource可以随机读写。
- ComPtr &会把管理的对象reset掉的，之后取指针地址。getaddressof就是单纯取指针地址。get是取与此相关联的接口。
- CreateRenderTargetView 创建渲染目标视图，OMSetRenderTargets 将渲染目标绑定到设备上
- 所以总结起来，DX11 的初始化可以分为以下几个步骤：
  - 创建 device、context、swapchain：D3D11CreateDeviceAndSwapChain
  - 为 backbuffer 创建 view：GetBuffer、CreateRenderTargetView
  - 创建 depth stencil 及其 view：CreateDepthStencilState、OMSetDepthStencilState、CreateTexture2D、CreateDepthStencilView
  - 将 view 绑定到 context 中：OMSetRenderTargets
  - 设置 viewport：RSSetViewports
- 最后再将 Imgui 初始化：ImGui_ImplDX11_Init

### Frame

- 在每一帧开始前，先调用 ImGui 的 NewFrame，然后 context 清空 rendertargetview 和 depthstencilview，用颜色填充背景
- 每一帧完成准备后，先是 ImGui render，然后是 swapchain present，IDXGISwapChain::Present 就是将一张渲染好的图片呈现给用户
- ID3D11DeviceContext::DrawIndexed方法：根据顶点和索引缓冲区进行绘制






## Bindable

- ID3D11DeviceContext interface：可以生成渲染命令的设备上下文
- ID3D11Device interface：设备接口，一个虚拟适配器，可以用来创建资源、运行渲染
- device 可以理解为我们的物理设备，它可以创建资源、着色器对象、状态对象等，还可以检查硬件特性，诊断调试等，可以将 device 看作程序中使用的各种资源的提供者。而 device context 主要包含了环境和设置，它可以使用 device 创建的资源设置管道以及建立渲染指令，提供了操作设备创建的资源的方法，device context 可以看作是设备产生的资源的消费者，分为实时渲染（即时上下文）和延时渲染（延迟上下文），即时上下文会在调用时立即提交以在驱动程序中执行，延迟上下文记录来自次要线程的一系列命令，生成一个命令列表对象，该对象可以稍后在即时上下文中回放。



![](../images/2022.10.24/1.png)



## 小结



## References

- [C++ 3D DirectX Tutorial [Node Tree / Transform Propagation]](https://www.youtube.com/watch?v=K7VqSmCiGiM&list=PLqCJpWy5Fohd3S7ICFXwUomYW0Wv67pDD&index=34)
- [DirectX11基础 ---设备(Device)和设备上下文(Device context)](https://blog.csdn.net/u014128662/article/details/90679565)
- [Direct3D11 Device(设备对象)，Device Context(设备上下文)官方SDK翻译](https://blog.csdn.net/pizi0475/article/details/7786306)
- [【DirectX11】【学习笔记（1）】初始化DirectX11](https://blog.csdn.net/qq_40947718/article/details/84100283)
