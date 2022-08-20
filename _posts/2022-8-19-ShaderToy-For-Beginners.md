---
layout: post
title: "面向新手的 ShaderToy —— 入门篇（一）"
description: "完全面向新手的 ShaderToy 教程，向伟大的 IQ 大神致敬🖖"
date: 2022-08-19
feature_image: images/2022.8.19/0.png 
tags: [Shader, ShaderToy]
---

大部分搞图形的人都知道 ShaderToy，我大三下的时候听杨老师提起过，但是碍于图形学基础不扎实，面对绚丽的画面除了发出几声感叹之外并没能理解到底是怎么回事。

研一上学期听了 Taichi 的图形课，才逐渐了解了 ShaderToy，在 Taichi 里面复现的乐乐女神的可爱的小雨伞还被选作 Taichi 日历的插图，唉，什么时候能和乐乐女神一样厉害呢......跟着 Taichi 图形课学习的日子可能是我读研以来为数不多的开心的一段日子？（因为是个人博客所以带的情绪会稍微多一点，感觉比知乎更自由~）

但是终归不想满足于浅尝辄止，现在打算好好整理一下 ShaderToy 的知识，那就从这篇面向新手（也就是我这种类型的菜鸡）的入门篇开始吧~

<!--more-->

## Shader 是什么

Shader 可以理解为一段拥有输入和输出的程序，这些程序是为我们屏幕生成画面服务的。在 ShaderToy 中主要会用到的 Shader 是 Pixel Shader，也称图像着色器，官网的解释如下：

> 为了通过计算每个像素的颜色来生成过程图像着色器需要执行 `mainImage()` 函数。
> 
> 每生成一个像素点时此函数都会被调用，并且此程序的责任是提供正确的输入并从中获得输出颜色并将其分配给屏幕像素。
> 
> 设计原型: 
> 
> void mainImage( out vec4 fragColor, in vec2 fragCoord );
> 
> 其中 fragCoord 包括着色器用来计算颜色的像素坐标。坐标是以像素为单位，精确范围从 0.5 到 -0.5, 覆盖渲染表面。
> 
> 其中分辨率通过 iResolution 参数传递给着色器。
> 
> 产生的结果颜色由四位向量储存在 fragColor 里, 最后一位被接受端忽略。 在未来用于叠加多个渲染目标时，结果作为“输出”参数储存。

上面的解释乍一看比较长哈，有点难理解，换一句话来说就是，每生成一个像素点时都会调用 `mainImage()`
 函数，该函数的作用是接收像素的坐标 fragCoord，输出该像素的颜色 fragColor。

比如说我有如下一段代码，输出的 fragColor 无论如何都为 (1, 1, 1, 1)：

```GLSL
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    fragColor = vec4(1., 1., 1.,1.0);
}
```

编译的结果就为一块白色：

![白色](/images/2022.8.19/1.png)

至于像素的坐标，传入的像素坐标以像素为单位，从 0 ~ iResolution，iResolution 可以理解为系统自带的一个全局变量，它的 x 代表屏幕的宽度，y 代表屏幕的高度。

![坐标系](/images/2022.8.19/2.png)

通常我们不直接拿来用 fragCoord，在图形学里常做的一个操作是归一化，我们需要将像素坐标转换为 0 ~ 1 范围内：

```GLSL
// Normalized pixel coordinates (from 0 to 1)
vec2 uv = fragCoord / iResolution.xy;
```

像素从屏幕左下角为零点，然后 x 向右增加至 1，y 向上增加至 1，比如我们把颜色改成这样：

```GLSL
fragColor = vec4(uv.x, 0., 0.,1.0);
```

可以得到下面的图像：

![x](/images/2022.8.19/3.png)

可以看到最左边一列是黑色，说明最左边是 0，最右边一列是红色，说明最右边是 1。

再来一个，比如这样：

```GLSL
fragColor = vec4(uv.x, uv.y., 0.,1.0);
```

![xy](/images/2022.8.19/4.png)

左下角为黑色，说明是 (0, 0)，左上角为绿色，说明是 (0, 1)，右下角为红色，说明是 (1, 0)，右上角为黄色，说明为 (1, 1)。

## 画一个圆

圆形由中心点和半径确定，我们通过计算像素坐标离中心点的距离来判断是否应该绘制圆形。

其中会用到 `length()` 函数，它帮助我们计算向量的长度。来看下面一段代码和结果：

```GLSL
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy;
    
    float d = length(uv);
    float col = d;

    fragColor = vec4(vec3(col), 1.0);
}
```

![length](/images/2022.8.19/5.png)

观察最左边一列，从下到上逐渐变白，说明长度由 0 逐渐变为 1，其他地方也是如此，开始变白的地方说明长度变为 1。

但是通常画面应该是居中的，而不是从左下角开始，那么我们就需要对像素左边进行一个小小的变换，将其挪到中心位置 (0.5, 0.5)：

```GLSL
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy;
    uv -= .5;
    
    float d = length(uv);
    float col = d;

    fragColor = vec4(vec3(col), 1.0);
}
```

![0.5length](/images/2022.8.19/6.png)

经过变换像素的坐标范围变成了 -0.5 ~ 0.5，屏幕最中心的位置为 (0, 0)。

了解了 `length()` 的作用之后就可以画圆了，思路很简单，就是判断像素离中心点的距离是否小于半径，如果小于则为白色，否则为黑色。

```GLSL
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy;
    uv -= .5;
    
    float d = length(uv);
    float col;

    if(d < .3)
        col = 1.;
    else
        col = 0.;
    

    fragColor = vec4(vec3(col), 1.0);
}
```

![circol](/images/2022.8.19/7.png)

等等，说好的画圆，咋个变成椭圆了呢？其实原因很简单，是因为我们屏幕的长宽不同，而坐标范围归一化之后是相同的，所以就会出现 x、y 方向上比例不一致的情况。因此我们需要对其中一边的坐标进行补偿，让其乘以 aspect (不知道怎么翻译，大概就是长/宽)。

```GLSL
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy;
    uv -= .5;
    uv.x *= iResolution.x / iResolution.y;
    
    float d = length(uv);
    float col;

    if(d < .3)
        col = 1.;
    else
        col = 0.;
    

    fragColor = vec4(vec3(col), 1.0);
}
```

![circol0](/images/2022.8.19/8.png)

## 消除锯齿

在消除锯齿时通常会用到 `smoothstep`，为什么会产生锯齿，通常是因为我们的边界条件设置的太严格了，从而会形成一个一个像素分布在边界上的感觉。而抗锯齿通常用来模糊边界，让像素的分布不要那么突兀。

在这个例子中，我们不对边界卡得那么死，利用 `smoothstep` 稍微平滑一下：

![circol0](/images/2022.8.19/9.png)

```GLSL
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy;
    uv -= .5;
    uv.x *= iResolution.x / iResolution.y;
    
    float d = length(uv);
    float r = 0.3;
    float col = smoothstep(r, r-0.01, d);

    fragColor = vec4(vec3(col), 1.0);
}
```

![smooth](/images/2022.8.19/10.png)

这里的魔法就是在 r-0.01 ~ r 之间的像素采取了平滑处理，取值在 0 ~ 1 之间平滑过渡，所以边界看起来会有一点糊糊的感觉，消除了锯齿。

将范围改大一点，比如 r-0.1~r，可以看到外面这一层过渡圈范围变大了。

![smooth1](/images/2022.8.19/11.png)

## 小结

有了本篇的基础应该能看懂不少简单的 ShaderToy 例子了，关键就是要知道 `mainImage()` 的作用，清楚输入输出的作用，坐标系需要弄清楚，那么就可以开始 ShaderToy 之旅啦~

## References

- https://www.youtube.com/watch?v=u5HAYVHsasc&t=82s