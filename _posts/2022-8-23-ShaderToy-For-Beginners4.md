---
layout: post
title: "面向新手的 ShaderToy —— 入门篇（五）"
description: "好看的笑脸！"
date: 2022-08-23
feature_image: images/2022.8.22/1.gif
tags: [Shader, ShaderToy]
---

让我们绘制一个更加好看的笑脸~

<!--more-->

## 从圆形开始绘制脸

按照之前所学的，我们先从圆形的脸蛋开始绘制，为了避免反复书写很长的 `smoothstep`，这里可以使用 `#define` 简化一下。

将头部、眼睛、嘴巴分成几个函数单独绘制。那么先开始绘制头部，首先就是按照以前的方法绘制圆形，不过这里最终是按照 a 通道混合颜色的，所以主要是通过设置 a 通道的值来作为 mask。

这里还将边界的一圈单独拿出来设置了一下，在边界范围内的 mask 介于 0 ~ 1 之间平滑过渡，形成一圈阴影的效果，最后再在最外圈绘制一圈边框。

```GLSL
#define S(a, b, t) smoothstep(a, b, t)
#define SAT(x) clamp(x, 0., 1.)

float remap(float a, float b, float t)
{
    return SAT((t - a) / (b - a));
}

float remap(float a, float b, float c, float d, float t)
{
    return remap(a, b, t) * (d - c) + c;
}

vec4 Eye(vec2 uv)
{
    vec4 col = vec4(.0);
    
    return col;
}

vec4 Mouth(vec2 uv)
{
    vec4 col = vec4(.0);
    
    return col;
}

vec4 Head(vec2 uv)
{
    vec4 col = vec4(.9, .65, .1, 1.);
    
    float d = length(uv);
    
    col.a = S(.5, .49, d);
    
    float edgeShade = remap(.35, .5, d);
    edgeShade *= edgeShade;
    col.rgb *= 1. - edgeShade * .5;
    
    col.rgb = mix(col.rgb, vec3(.6, .3, .1), S(.47, .48, d));
    
    return col;
}

vec4 Smiley(vec2 uv)
{
    vec4 col = vec4(.0);
    
    vec4 head = Head(uv);
    
    col = mix(col, head, head.a);
    
    return col;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy;
    uv -= .5;
    uv.x *= iResolution.x / iResolution.y;


    fragColor = Smiley(uv);
}
```

![face](/images/2022.8.23/0.png)


## 小结

## References

- [Making a smiley in ShaderToy](https://www.youtube.com/watch?v=ZlNnrpM0TRg&list=PLGmrMu-IwbguU_nY2egTFmlg691DN7uE5&index=5)