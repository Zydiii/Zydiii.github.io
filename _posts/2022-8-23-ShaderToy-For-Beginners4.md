---
layout: post
title: "面向新手的 ShaderToy —— 入门篇（五）"
description: "好看的笑脸！"
date: 2022-08-23
feature_image: images/2022.8.23/5.png
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

接下来从额头到脸部中央加一点高光，利用 Remap 方法，根据 uv.y 确定高光的值。

```GLSL
vec4 Head(vec2 uv)
{
    vec4 col = vec4(.9, .65, .1, 1.);
    
    float d = length(uv);
    
    col.a = S(.5, .49, d);
    
    float edgeShade = remap(.35, .5, d);
    edgeShade *= edgeShade;
    col.rgb *= 1. - edgeShade * .5;
    
    col.rgb = mix(col.rgb, vec3(.6, .3, .1), S(.47, .48, d));
    
    float highlight = S(.41, .405, d);
    highlight *= remap(.41, -.1, .75, 0., uv.y);
    col.rgb = mix(col.rgb, vec3(1.), highlight);
    
    return col;
}

```

![face](/images/2022.8.23/1.png)

同样的方法画上两个粉红的脸颊，这里有一个小技巧是我们要绘制的脸是相对于 x 轴对称的，所以在计算时可以将 uv.x 取一个绝对值，这样就可以做出一个对称的效果了。

```GLSL
vec4 Head(vec2 uv)
{
    vec4 col = vec4(.9, .65, .1, 1.);
    
    float d = length(uv);
    
    col.a = S(.5, .49, d);
    
    float edgeShade = remap(.35, .5, d);
    edgeShade *= edgeShade;
    col.rgb *= 1. - edgeShade * .5;
    
    col.rgb = mix(col.rgb, vec3(.6, .3, .1), S(.47, .48, d));
    
    float highlight = S(.41, .405, d);
    highlight *= remap(.41, -.1, .75, 0., uv.y);
    col.rgb = mix(col.rgb, vec3(1.), highlight);
    
    d = length(uv-vec2(.25, -.2));
    float cheek = S(.2, .01, d) * .4;
    cheek *= S(.17, .16, d);
    col.rgb = mix(col.rgb, vec3(1., .1, .1), cheek);
    
    return col;
}

vec4 Smiley(vec2 uv)
{
    vec4 col = vec4(.0);
    
    uv.x = abs(uv.x);
    vec4 head = Head(uv);
    
    col = mix(col, head, head.a);
    
    return col;
}
```

![](/images/2022.8.23/2.png)

## 眼睛

眼睛也是圆形，为了方便绘制，我们可以选定一个矩形区域作为眼睛要绘制的中心区域，然后将 uv 坐标映射到这个区域内，转化为 0~1：

```GLSL
vec2 within(vec2 uv, vec4 rect)
{
    return (uv - rect.xy) / (rect.zw - rect.xy);
}
```

之后我们绘制眼睛的时候就可以更专注于中心区域的绘制，而不用过多考虑偏移等因素。

```GLSL
vec4 Eye(vec2 uv)
{
    uv -= .5;
    vec4 col = vec4(1.);
    
    float d = length(uv);
    col.a = S(.5, .48, d);
    
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
    
    float highlight = S(.41, .405, d);
    highlight *= remap(.41, -.1, .75, 0., uv.y);
    col.rgb = mix(col.rgb, vec3(1.), highlight);
    
    d = length(uv-vec2(.25, -.2));
    float cheek = S(.2, .01, d) * .4;
    cheek *= S(.17, .16, d);
    col.rgb = mix(col.rgb, vec3(1., .1, .1), cheek);
    
    return col;
}
```

![eye](/images/2022.8.23/3.png)

然后是一系列圆形的绘制组合，形成卡姿兰大眼睛~画出来竟然还挺好看的，艺术家不愧是艺术家！

```GLSL
vec4 Eye(vec2 uv)
{
    uv -= .5;
    float d = length(uv);
    
    vec4 irisCol = vec4(.3, .5, 1., 1.);
    vec4 col = mix(vec4(1.), irisCol, S(.1, .7, d) * .5);
    
    col.rgb *= 1. - S(.45, .5, d) * .5 * SAT(-uv.y-uv.x);
    col.rgb = mix(col.rgb, vec3(0.), S(.3, .28, d)); // iris outline
    
    irisCol.rgb *= 1. + S(.3, .05, d);
    col.rgb = mix(col.rgb, irisCol.rgb, S(.28, .25, d));
    
    col.rgb = mix(col.rgb, vec3(0.), S(.16, .14, d));
    
    float highlight = S(.1, .09, length(uv - vec2(-.15, .15)));
    highlight +=  S(.07, .05, length(uv + vec2(-.08, .08)));
    col.rgb = mix(col.rgb, vec3(1.), highlight);
    
    col.a = S(.5, .48, d);
    
    return col;
}
```

![eye](/images/2022.8.23/4.png)

## 嘴巴

嘴巴需要有一个嘴角上扬的效果，那么在圆形判断离中心点的距离时，嘴角的距离需要适当地减去一部分，从而可以绘制更多的部分，这里用二次曲线作为减去的值，从而可以实现嘴巴的效果。

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

vec2 within(vec2 uv, vec4 rect)
{
    return (uv - rect.xy) / (rect.zw - rect.xy);
}

vec4 Eye(vec2 uv)
{
    uv -= .5;
    float d = length(uv);
    
    vec4 irisCol = vec4(.3, .5, 1., 1.);
    vec4 col = mix(vec4(1.), irisCol, S(.1, .7, d) * .5);
    
    col.rgb *= 1. - S(.45, .5, d) * .5 * SAT(-uv.y-uv.x);
    col.rgb = mix(col.rgb, vec3(0.), S(.3, .28, d)); // iris outline
    
    irisCol.rgb *= 1. + S(.3, .05, d);
    col.rgb = mix(col.rgb, irisCol.rgb, S(.28, .25, d));
    
    col.rgb = mix(col.rgb, vec3(0.), S(.16, .14, d));
    
    float highlight = S(.1, .09, length(uv - vec2(-.15, .15)));
    highlight +=  S(.07, .05, length(uv + vec2(-.08, .08)));
    col.rgb = mix(col.rgb, vec3(1.), highlight);
    
    col.a = S(.5, .48, d);
    
    return col;
}

vec4 Mouth(vec2 uv)
{
    uv -= .5;
    vec4 col = vec4(.5, .18, .05, 1.);
    
    uv.y *= 1.5;
    uv.y -= uv.x*uv.x*2.;
    
    float d = length(uv);
    col.a = S(.5, .48, d);
    
    float td = length(uv-vec2(0., .6));
    vec3 toothCol = vec3(1.) * S(.6, .35, d);
    
    col.rgb = mix(col.rgb, toothCol, S(.4, .37, td));
    
    td = length(uv+vec2(0., .5));
    col.rgb = mix(col.rgb, vec3(1., .5, .5), S(.5, .2, td));
    
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
    
    float highlight = S(.41, .405, d);
    highlight *= remap(.41, -.1, .75, 0., uv.y);
    col.rgb = mix(col.rgb, vec3(1.), highlight);
    
    d = length(uv-vec2(.25, -.2));
    float cheek = S(.2, .01, d) * .4;
    cheek *= S(.17, .16, d);
    col.rgb = mix(col.rgb, vec3(1., .1, .1), cheek);
    
    return col;
}

vec4 Smiley(vec2 uv)
{
    vec4 col = vec4(.0);
    
    uv.x = abs(uv.x);
    vec4 head = Head(uv);
    vec4 eye = Eye(within(uv, vec4(.03, -.1, .37, .25)));
    vec4 mouth = Mouth(within(uv, vec4(-.3, -.4, .3, -.1)));
    
    col = mix(col, head, head.a);
    col = mix(col, eye, eye.a);
    col = mix(col, mouth, mouth.a);

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

![eye](/images/2022.8.23/5.png)

## 小结

感觉一切都好神奇！明明只是用了圆形的绘制方法，但是竟然可以形成一张看起来还比较立体的笑脸效果，真的太厉害了！在这一篇主要学到了四个小技巧：1）可以将要绘制的区域设定为一个 Rect，然后将 uv 映射到这个 Rect 中间绘制，这样就不需要被其他元素影响 2）为了形成更复杂的形状，可以在计算距离的时候加减一些函数值，从而生成不那么规则的形状 3）善用 Smoothstep、clamp、mix、线性映射等方法，绘制出想要的效果
4）在绘制对称的形状时，可以只考虑绘制一半，然后将 uv 取绝对值

## References

- [Making a smiley in ShaderToy](https://www.youtube.com/watch?v=ZlNnrpM0TRg&list=PLGmrMu-IwbguU_nY2egTFmlg691DN7uE5&index=5)