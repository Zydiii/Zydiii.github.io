---
layout: post
title: "Post Process in UE"
description: ""
date: 2022-10-16
feature_image: images/2022.10.5/0.gif
tags: [Unreal]
---

<!--more-->

后处理可以创建很多有趣的效果，接下来就来学习一下吧~

## Night Vision

- 首先新建一个第一人称游戏，然后在 FirstPersionCharacter 蓝图中添加一个 PostProcess 组件，调整上面的一些参数

### 使用查找表（LUT）进行颜色校正

- 在 PostProcess 中可以设置 LUT，可以提供更精细的色彩变换，从而可用于去饱和度之类的用途，UE 里面使用的默认查找表是中性色调 LUT，是一张 256*16 的纹理，我们可以在 PS 中对该 LUT 进行 Adjustment，调整其饱和度，从而生成新的 LUT，比如我想要一个全灰的场景，我就可以先截取一张场景图到 PS，然后调整颜色

![](../images/2022.10.16/3.png)

- 然后 copy 官方的 LUT，将调整应用到该 LUT 生成新的 LUT，这样就得到了不同的颜色视效，好神奇有木有

![](../images/2022.10.16/2.png)

![](../images/2022.10.16/4.png)

- LUT 本质就是通过一个表，把一个颜色映射为另一个颜色，大致的思路就是将原来的颜色作为坐标，去查找 LUT 上对应的颜色，这样就可以将原来的色调映射到想要的色调了
- 创建材质，类型设置为后处理材质，这里是为了模拟抖动的效果利用一张噪声贴图调整 UV，SceneTexture:PostProcessInput0 这个节点其实就是SceneColor，也就是场景本来的颜色。SceneColor 只能用于 MaterialDomain 为 Surface（表面）的材质
- 创建两个 UserWidget，第一个是显示相机窗口，加上了一点图片，第二个是用来过渡的，是全黑色的，然后加上了一段动画让不透明度由 0 到 1 到 0，形成过渡效果，在 Event Construct 的时候 PlayAnimation 即可

![](../images/2022.10.16/0.png)

- 然后在角色蓝图中添加相关逻辑，在 BeginPlay 时 Create Widget，Add to Viewport，Set Visibility，然后检测键盘按下 C，创建过渡 UI 并添加到视口，因为有动画可以添加 Delay，然后设置 Visibility，PostProcess 可以通过设置 Enabled 来设置是否开启后处理效果，Flip Flop 这个节点挺方便的

![](../images/2022.10.16/1.png)

![](../images/2022.10.16/0.gif)

- 所以总结起来，上面的方法就是通过调整 LUT 和后处理节点的一些参数来实现夜视效果

### Material

- RadialGradientExponential 径向梯度指数，我感觉就是生成了一张从圆心到四周产生渐变的 Mask，然后将这个 Mask 与 SceneTexture 和自定义个颜色相乘，就可以形成一种绿色的夜视效果了

![](../images/2022.10.16/6.png)

![](../images/2022.10.16/5.png)

- 相机也是可以直接加后处理材质的，可以通过 Blend Weight 调整效果，这种方法本质上就是对场景叠加了一层颜色







## 小结



## References

- [UE4 Tutorial: Night Vision Camera (Outlast)](https://www.youtube.com/watch?v=O-0DDBn04mY)
- [使用查找表（LUT）进行颜色校正](https://docs.unrealengine.com/4.27/zh-CN/RenderingAndGraphics/PostProcessEffects/UsingLUTs/)
- [LUT（look up table）调色的原理与代码实现](https://www.jianshu.com/p/d09aeea3b732)
- [UE4 FPS Night Vision l Unreal Engine 4.26 (Tutorial)](https://www.youtube.com/watch?v=3Eo7vM90XCI)