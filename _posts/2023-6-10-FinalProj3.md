---
layout: post
title: "毕设记录（三）"
description: ""
date: 2023-6-10
feature_image: images/2023.6.10/0.png
tags: [毕设]
---

<!--more-->

## 旧项目 Unity 模型资源处理与导出

- （咱就是说能不能做一点正儿八经写代码的工作 = =）

### Jet_Engine_Flames

- 在 Unity 中安装 FBX Exporter，这样可以正确导出材质，选择要导出的对象，然后在 GameObject 菜单下面 Export，格式选择 Binary

![](../images/2023.6.10/1.png)

- 玻璃材质可能需要手动处理，Blender 里面玻璃材质需要将 transmission 设置为 1，roughness 设置为 0.01，或者直接在 Unreal 里面设置，将材质设置为半透明模式，然后调整参数

![](../images/2023.6.10/2.png)

- 导出时为了将材质纹理一并导出，Path Type 选择 Copy
- 需要注意一下坐标系，x 轴是正方向，缩放之后记得 Ctrl + A 应用 All Transform

![](../images/2023.6.10/0.png)

- 选中所有 Mesh 右键 Set Origin 为 Geometry Center（选机体作为 Parent，然后 Shift + S 将选中的 Mesh Move To Cursor 移动到世界中心，然后适当调整高度）
- 检查一下 Mesh 之间的层级关系，保证有个 Parent Node，导出为 FBX，Unreal 导入时勾选 Skeleton Mesh
- Unity 和 UE 的效果对比：

![](../images/2023.6.10/4.png)

![](../images/2023.6.10/3.png)

### daodan1

- 这个没有贴图材质，只改了颜色

![](../images/2023.6.10/5.png)

![](../images/2023.6.10/6.png)

### daodan2

![](../images/2023.6.10/7.png)

![](../images/2023.6.10/8.png)

### daodan3

![](../images/2023.6.10/9.png)

![](../images/2023.6.10/10.png)

### daodan4

![](../images/2023.6.10/11.png)

![](../images/2023.6.10/12.png)

### daodan5

![](../images/2023.6.10/13.png)

![](../images/2023.6.10/14.png)

### daodan6

![](../images/2023.6.10/15.png)

![](../images/2023.6.10/16.png)

### file

![](../images/2023.6.10/17.png)

![](../images/2023.6.10/18.png)

### haizhi9

![](../images/2023.6.10/19.png)

![](../images/2023.6.10/20.png)

### jian11b

![](../images/2023.6.10/21.png)

![](../images/2023.6.10/22.png)

### luzhi9

![](../images/2023.6.10/23.png)

![](../images/2023.6.10/24.png)

### yun8

![](../images/2023.6.10/25.png)

![](../images/2023.6.10/26.png)

### 模型（obj 格式）

- 我觉得为什么这些模型这么奇怪，材质很多都不对，根本没有纹理

![](../images/2023.6.10/27.png)

![](../images/2023.6.10/28.png)

### Super_Spitfire

- Blender 下面的材质变成了全透明，有一点奇怪，但 UE 是好的，没有发现怎么解决，不过这个飞机还挺可爱的

![](../images/2023.6.10/36.png)

![](../images/2023.6.10/37.png)

![](../images/2023.6.10/38.png)

### yjj

- 3dmax 格式下的模型看起来是对的，但是 FBX 导入到 Blender 就不太对了，甚至不能直接导入 UE 会报错。导出 obj 倒是可以导入，但 mesh 多少有点问题

![](../images/2023.6.10/30.png)

![](../images/2023.6.10/29.png)

![](../images/2023.6.10/31.png)

![](../images/2023.6.10/32.png)

![](../images/2023.6.10/33.png)

### MZ0139_侦察预警运输机集合

- 这些 3dmax 的文件绝对有问题啊，纹理材质路径根本就不对，感觉没有必要再导出了

![](../images/2023.6.10/34.png)

![](../images/2023.6.10/35.png)

### Turbosquid - The Russian Arrows 2

- 听赵主任的话去买了淘宝的二手模型，发现这个模型是之前学长买过的！可恶
- 3DS 格式，Unity 能直接读取，但是材质有的不正确，所以正确的路线还是应该从 3DS 里面导出 FBX，然后导入 Blender 处理再导入 Unreal





## References

