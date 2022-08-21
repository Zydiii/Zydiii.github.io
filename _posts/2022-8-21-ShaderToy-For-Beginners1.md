---
layout: post
title: "面向新手的 ShaderToy —— 入门篇（三）"
description: "新伙伴——矩形"
date: 2022-08-21
feature_image: images/2022.8.21/4.png 
tags: [Shader, ShaderToy]
---

这一篇将要绘制我们的新朋友——矩形！

<!--more-->

## 分界线

首先来做一个 x 轴的分界线，左边为 0，右边为 1，同时用上可爱的 `smoothstep`：

```GLSL
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy; // 0 <> 1
    uv -= .5; // -0.5 <> 0.5
    uv.x *= iResolution.x / iResolution.y;
    
    float mask = smoothstep(-.1, .1, uv.x);
    vec3 col = vec3(1.,1.,1.) * mask;

    fragColor = vec4(col,1.0);
}
```

![x](/images/2022.8.21/6.png)

把边界线封装一下，做成左右两边界：

```GLSL
float Bound(float p, float start, float end, float blur)
{
    float step1 = smoothstep(start - blur, start + blur, p);
    float step2 = smoothstep(end + blur, end - blur, p);
    return step1 * step2;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy; // 0 <> 1
    uv -= .5; // -0.5 <> 0.5
    uv.x *= iResolution.x / iResolution.y;
    
    float mask = Bound(uv.x, -.2, .2, .01);
    vec3 col = vec3(1.,1.,1.) * mask;

    fragColor = vec4(col,1.0);
}
```

![bounds](/images/2022.8.21/7.png)

在 `Bound()` 中为何是 step1 * step2 呢？且看下面这张图：

![bounds](/images/2022.8.21/5.png)

蓝色函数可以看作是左边界的 smoothstep，红色函数看作是右边界的 smoothstep，如果需要让它们中间的函数值为 1，其他为 0，那么就需要是绿色的这段函数曲线，而它正是红、蓝两个函数相乘的结果。要注意右边界的 smoothstep 是反过来的，传入函数中的参数也应该是反过来的。

## 矩形

有了边界绘制矩形好像也没什么好说的，传进去上下左右边界值就行了：

```GLSL
float Bound(float p, float start, float end, float blur)
{
    float step1 = smoothstep(start - blur, start + blur, p);
    float step2 = smoothstep(end + blur, end - blur, p);
    return step1 * step2;
}

float Rect(vec2 uv, float left, float right, float bottom, float up, float blur)
{
    float bound1 = Bound(uv.x, left, right, blur);
    float bound2 = Bound(uv.y, bottom, up, blur);
    return bound1 * bound2;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy; // 0 <> 1
    uv -= .5; // -0.5 <> 0.5
    uv.x *= iResolution.x / iResolution.y;
    
    float mask = Rect(uv, -.2, .2, -.3, .3, .01);
    vec3 col = vec3(1.,1.,1.) * mask;

    fragColor = vec4(col,1.0);
}
```

![bounds](/images/2022.8.21/8.png)

## 小结

感觉这一篇比画圆简单hhh，关键就是根据 uv 判断是否处于矩形中，利用好 `smoothstep` 达到更好的绘制效果~

## References

- [ShaderToy Tutorial Part 3 - Making a Rectangle](https://www.youtube.com/watch?v=bigjgiavOM0&list=PLGmrMu-IwbguU_nY2egTFmlg691DN7uE5&index=3)

