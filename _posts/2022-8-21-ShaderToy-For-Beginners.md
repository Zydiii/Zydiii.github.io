---
layout: post
title: "面向新手的 ShaderToy —— 入门篇（二）"
description: ""
date: 2022-08-21
feature_image: images/2022.8.21/4.png 
tags: [Shader, ShaderToy]
---

这一篇利用上一篇画圆的技巧来绘制更复杂的形状吧~

<!--more-->

## 添加画圆函数

Shader 中的函数和 C++ 里的函数类似，这里我们将画圆的逻辑封装成 `drawCircle()` 函数，就可以很方便的绘制多个圆：

```GLSL
float drawCircle(vec2 uv, vec2 center, float radius, float blur)
{
    float d = length(uv - center);
    float c = smoothstep(radius, radius - blur, d);
    return c;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy; // 0 < > 1
    uv -= 0.5; // -0.5 < > 0.5
    uv.x *= iResolution.x / iResolution.y;

    float c = drawCircle(uv, vec2(.0, .0), .4, .01);
    c += drawCircle(uv, vec2(0.5, .3), .1, .01);


    fragColor = vec4(vec3(c),1.0);
}
```

![circle](/images/2022.8.21/1.png)

在绘制第二个圆的时候我们用了加法，因此将两个圆都绘制了出来，如果我们用减法，那么就可以组合出想要的一些形状：

```GLSL
float drawCircle(vec2 uv, vec2 center, float radius, float blur)
{
    float d = length(uv - center);
    float c = smoothstep(radius, radius - blur, d);
    return c;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy; // 0 < > 1
    uv -= 0.5; // -0.5 < > 0.5
    uv.x *= iResolution.x / iResolution.y;

    float c = drawCircle(uv, vec2(.0, .0), .4, .02);
    c -= drawCircle(uv, vec2(-.13, .2), .07, .01);
    c -= drawCircle(uv, vec2(.13, .2), .07, .01);


    fragColor = vec4(vec3(c),1.0);
}
```

![eye](/images/2022.8.21/2.png)

再来加个嘴巴~

```GLSL
float drawCircle(vec2 uv, vec2 center, float radius, float blur)
{
    float d = length(uv - center);
    float c = smoothstep(radius, radius - blur, d);
    return c;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy; // 0 < > 1
    uv -= 0.5; // -0.5 < > 0.5
    uv.x *= iResolution.x / iResolution.y;

    float c = drawCircle(uv, vec2(.0, .0), .4, .02);
    c -= drawCircle(uv, vec2(-.13, .2), .07, .01);
    c -= drawCircle(uv, vec2(.13, .2), .07, .01);
    
    float mouth = drawCircle(uv, vec2(.0, .0), .3, .02);
    mouth -= drawCircle(uv, vec2(.0, .1), .3, .02);
    c -= mouth;

    fragColor = vec4(vec3(c),1.0);
}
```

![mouth](/images/2022.8.21/3.png)

## 加点颜色

至此我们绘制的颜色都介于黑白之间，如果我们想要给画面加上颜色，就需要再添加一个颜色，将之前计算得到的 float 作为一个 mask 求得最终的颜色。

```GLSL
float drawCircle(vec2 uv, vec2 center, float radius, float blur)
{
    float d = length(uv - center);
    float c = smoothstep(radius, radius - blur, d);
    return c;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy; // 0 < > 1
    uv -= 0.5; // -0.5 < > 0.5
    uv.x *= iResolution.x / iResolution.y;

    float mask = drawCircle(uv, vec2(.0, .0), .4, .02);
    mask -= drawCircle(uv, vec2(-.13, .2), .07, .01);
    mask -= drawCircle(uv, vec2(.13, .2), .07, .01);
    
    float mouth = drawCircle(uv, vec2(.0, .0), .3, .02);
    mouth -= drawCircle(uv, vec2(.0, .1), .3, .02);
    mask -= mouth;
    
    vec3 bg = vec3(1.0, 1.0, .0);
    vec3 col = bg * mask;

    fragColor = vec4(col, 1.0);
}
```

![mouth](/images/2022.8.21/4.png)

最后把我们的绘制笑脸逻辑封装成函数方便调用：

```GLSL
float drawCircle(vec2 uv, vec2 center, float radius, float blur)
{
    float d = length(uv - center);
    float c = smoothstep(radius, radius - blur, d);
    return c;
}

float drawSmilyFace(vec2 uv, vec2 center, float size)
{
    uv -= center;
    uv /= size;
    
    float mask = drawCircle(uv, vec2(.0, .0), .4, .02);
    mask -= drawCircle(uv, vec2(-.13, .2), .07, .01);
    mask -= drawCircle(uv, vec2(.13, .2), .07, .01);
    
    float mouth = drawCircle(uv, vec2(.0, .0), .3, .02);
    mouth -= drawCircle(uv, vec2(.0, .1), .3, .02);
    mask -= mouth;
    return mask;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy; // 0 < > 1
    uv -= 0.5; // -0.5 < > 0.5
    uv.x *= iResolution.x / iResolution.y;

    float mask = drawSmilyFace(uv, vec2(.0, .0), 1.0);
    vec3 bg = vec3(1.0, 1.0, .0);
    vec3 col = bg * mask;

    fragColor = vec4(col, 1.0);
}
```

## References

- [ShaderToy Tutorial Part 2 - Building stuff with circles](https://www.youtube.com/watch?v=GgGBR4z8C9o&list=PLGmrMu-IwbguU_nY2egTFmlg691DN7uE5&index=2)
- [ShaderToy Tutorial Part 3 - Making a Rectangle](https://www.youtube.com/watch?v=bigjgiavOM0&list=PLGmrMu-IwbguU_nY2egTFmlg691DN7uE5&index=3)

