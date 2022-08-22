---
layout: post
title: "面向新手的 ShaderToy —— 入门篇（四）"
description: "加一点变化！"
date: 2022-08-22
feature_image: images/2022.8.21/4.png 
tags: [Shader, ShaderToy]
---

光看静止的图片是否有些无聊了呢，让我们试试让画面动起来吧！

<!--more-->

## 波形

我们想要对传入的 uv 进行一些变化，生成波形的效果，那么就需要一点数学的知识。我们有这个简单的二次曲线 $-\left(x\ -\ 0.5\right)\left(x\ +\ 0.5\right)$:

![xx](/images/2022.8.22/0.png)

这意味着根据这样的二次曲线计算出 y 的偏移值，可以形成波形：

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
    
    float x = uv.x;
    float y = uv.y;
    
    float m = (x -.5) * (x + .5);
    y += m;
    
    float mask = Rect(vec2(x, y), -.5, .5, -.1, .1, .01);
    vec3 col = vec3(1.,1.,1.) * mask;

    fragColor = vec4(col,1.0);
}
```

![xx](/images/2022.8.22/1.png)

让我们把它弄平滑一点：

![aa4](/images/2022.8.22/2.png)

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
    
    float x = uv.x;
    float y = uv.y;
    
    float m = -(x -.5) * (x + .5);
    m = m * m * 4.;
    y -= m;
    
    float mask = Rect(vec2(x, y), -.5, .5, -.1, .1, .01);
    vec3 col = vec3(1.,1.,1.) * mask;

    fragColor = vec4(col,1.0);
}
```

![smooth](/images/2022.8.22/3.png)

做曲线相对来说可以用三角函数比较方便，这里我们用 `sin`：

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
    
    float x = uv.x;
    float y = uv.y;
    
    float m = sin(x*8.)*.1;
    y -= m;
    
    float mask = Rect(vec2(x, y), -.5, .5, -.1, .1, .01);
    vec3 col = vec3(1.,1.,1.) * mask;

    fragColor = vec4(col,1.0);
}
```

![smooth](/images/2022.8.22/4.png)

已经有一种小水流的感觉了！

## 根据时间的变化

我们可以通过 `iTime` 获得运行时间，将其作为偏移传入 `sin` 我们就可以得到动态的效果：

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
    
    float t = iTime;
    
    float x = uv.x;
    float y = uv.y;
    
    float m = sin(x*8.+t)*.1;
    y -= m;
    
    float mask = Rect(vec2(x, y), -.5, .5, -.1, .1, .01);
    vec3 col = vec3(1.,1.,1.) * mask;

    fragColor = vec4(col,1.0);
}
```

![smooth](/images/2022.8.22/0.gif)

这样看有些许单调，让我们根据 x 设置一下 blur 值：

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

float Remap01(float a, float b, float t)
{
    return (t - a) / (b - a);
}

float Remap(float a, float b, float c, float d, float t)
{
    return Remap01(a, b, t) * (d - c) + c;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy; // 0 <> 1
    uv -= .5; // -0.5 <> 0.5
    uv.x *= iResolution.x / iResolution.y;
    
    float t = iTime;
    
    float x = uv.x;
    float y = uv.y;
    
    float m = sin(x*8.+t)*.1;
    y -= m;
    
    float blur = Remap(-.5, .5, .01, .25, x);
    blur *= pow(blur * 4., 2.);
    
    float mask = Rect(vec2(x, y), -.5, .5, -.1, .1, blur);
    vec3 col = vec3(1.,1.,1.) * mask;

    fragColor = vec4(col,1.0);
}
```

![blur](/images/2022.8.22/1.gif)

## 小结

这一篇好像也比较简单，关键就是利用 `iTime` 来构造一些变化，比如坐标的偏移，从而形成动态的效果。根据一些函数，如 `sin`，可以来绘制好看的波形。

## References

- [ShaderToy Tutorial Part 4 - Domain distortion](https://www.youtube.com/watch?v=jKuXA0trQPE&list=PLGmrMu-IwbguU_nY2egTFmlg691DN7uE5&index=4)