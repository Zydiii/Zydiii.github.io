---
layout: post
title: "面向新手的 ShaderToy —— 入门篇（六）"
description: "会动的笑脸！"
date: 2022-08-26
feature_image: images/2022.8.26/2.gif
tags: [Shader, ShaderToy]
---

更加神奇的魔法！

<!--more-->

## 跟随鼠标运动的眼球

首先修复一个小问题：眼睛的高光是对称的，但是其实高光应该是相同的，但是由于我们对 uv.x 取了绝对值，所以绘制出来的效果是对称的，这样看起来比较奇怪，所以在绘制眼睛的时候，我们把 uv.x 取回原来的符号，这可以通过传入一个符号标识来实现，可以利用 `sign()` 函数取得符号。

```GLSL
vec4 Eye(vec2 uv, float side)
{
    uv -= .5;
    uv.x *= side;
    
    float d = length(uv);
    
    vec4 irisCol = vec4(.3, .5, 1., 1.);
    vec4 col = mix(vec4(1.), irisCol, S(.1, .7, d) * .5);
    
    col.rgb *= 1. - S(.45, .5, d) * .5 * SAT(-uv.y-uv.x*side);
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

![face](/images/2022.8.26/0.png)

接下来需要模拟眼珠随着鼠标移动的效果，这里利用 ShaderToy 自带的 `iMouse` 获取鼠标点击的像素位置，然后进行归一化，以及将其原点偏移至画面中心，将鼠标位置作为一个参数传入绘制函数。

```GLSL
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy;
    uv -= .5;
    uv.x *= iResolution.x / iResolution.y;
    
    vec2 m = iMouse.xy / iResolution.xy;
    m -= .5;


    fragColor = Smiley(uv, m);
}
```

为了模拟眼球跟随鼠标移动，需要在计算距离时适当地减去传入的鼠标位置即可。

```GLSL
vec4 Eye(vec2 uv, float side, vec2 m)
{
    uv -= .5;
    uv.x *= side;
    
    float d = length(uv);
    
    vec4 irisCol = vec4(.3, .5, 1., 1.);
    vec4 col = mix(vec4(1.), irisCol, S(.1, .7, d) * .5);
    col.a = S(.5, .48, d);
    
    col.rgb *= 1. - S(.45, .5, d) * .5 * SAT(-uv.y-uv.x*side);
    
    d = length(uv-m*.5);
    col.rgb = mix(col.rgb, vec3(0.), S(.3, .28, d)); // iris outline
    irisCol.rgb *= 1. + S(.3, .05, d);
    col.rgb = mix(col.rgb, irisCol.rgb, S(.28, .25, d));
    
    d = length(uv-m*.6);
    col.rgb = mix(col.rgb, vec3(0.), S(.16, .14, d));
    
    float highlight = S(.1, .09, length(uv - vec2(-.15, .15)));
    highlight +=  S(.07, .05, length(uv + vec2(-.08, .08)));
    col.rgb = mix(col.rgb, vec3(1.), highlight);
    
    return col;
}
```

![face](/images/2022.8.26/0.gif)

同时这里还添加了眼球和高光随时间变化的效果（虽然我觉得有点诡异），大体思路就是根据 `sin(iTime)`  调整眼球大小和高光偏移。

```GLSL
vec4 Eye(vec2 uv, float side, vec2 m, float smile)
{
    uv -= .5;
    uv.x *= side;
    
    float d = length(uv);
    
    vec4 irisCol = vec4(.3, .5, 1., 1.);
    vec4 col = mix(vec4(1.), irisCol, S(.1, .7, d) * .5);
    col.a = S(.5, .48, d);
    
    col.rgb *= 1. - S(.45, .5, d) * .5 * SAT(-uv.y-uv.x*side);
    
    d = length(uv-m*.5);
    col.rgb = mix(col.rgb, vec3(0.), S(.3, .28, d)); // iris outline
    irisCol.rgb *= 1. + S(.3, .05, d);
    float irisMask = S(.28, .25, d);
    col.rgb = mix(col.rgb, irisCol.rgb, irisMask);
    
    d = length(uv-m*.6);
    float pupilSize = mix(.4, .16, smile);
    float pupilMask = S(pupilSize, pupilSize*.85, d);
    pupilMask *= irisMask;
    col.rgb = mix(col.rgb, vec3(0.), pupilMask);
    
    float t = iTime*3.;
    vec2 offs = vec2(sin(t+uv.y*25.), sin(t+uv.x*25.));
    offs *= .01*(1.-smile);
    uv += offs;
    
    float highlight = S(.1, .09, length(uv - vec2(-.15, .15)));
    highlight +=  S(.07, .05, length(uv + vec2(-.08, .08)));
    col.rgb = mix(col.rgb, vec3(1.), highlight);
    
    return col;
}
```
## 跟随时间变化的嘴巴和眉毛

嘴巴的处理比较简单，将 uv 随时间进行偏移，可以形成嘴巴变大变小的效果，同理将牙齿的 y 也进行一个偏移。

```GLSL
vec4 Mouth(vec2 uv, float smile)
{
    uv -= .5;
    vec4 col = vec4(.5, .18, .05, 1.);
    
    uv.y *= 1.5;
    uv.y -= uv.x*uv.x*2.*smile;
    
    uv.x *= mix(2., 1., smile);
    
    float d = length(uv);
    col.a = S(.5, .48, d);
    
    vec2 tUV = uv;
    tUV.y += (abs(uv.x)*.5+.1)*(1.-smile);
    float td = length(tUV-vec2(0., .6));
    vec3 toothCol = vec3(1.) * S(.6, .35, d);
    
    col.rgb = mix(col.rgb, toothCol, S(.4, .37, td));
    
    td = length(uv+vec2(0., .5));
    col.rgb = mix(col.rgb, vec3(1., .5, .5), S(.5, .2, td));
    
    return col;
}
```

![face](/images/2022.8.26/1.gif)

眉毛的处理也是类似的，其根据时间将 uv 进行偏移，同时阴影的 blur 也跟着变化，从而形成物体越靠近时阴影越 sharp 的效果。

```GLSL
vec4 Brow(vec2 uv, float smile)
{
    float offs = mix(.2, 0., smile);
    uv.y += offs;
    
    float y = uv.y;
    uv.y += uv.x*mix(.5, .8, smile)-mix(.1, .3, smile);
    uv.x -= mix(-.0,.1, smile);
    uv -= .5;

    vec4 col = vec4(0.);
    
    float blur = .1;
    
    float d1 = length(uv);
    float s1 = S(.45, .45-blur, d1);
    float d2 = length(uv-vec2(.1, -.2)*.7);
    float s2 = S(.5, .5-blur, d2);
    
    float browMask = SAT(s1-s2);
    
    float colMask = remap(.7, .8, y)*.75;
    colMask *= S(.6, .9, browMask);
    colMask *= smile;
    vec4 browCol = mix(vec4(.4, .2, .2, 1.), vec4(1., .75, .5, 1.), colMask);
    
    uv.y += .15-offs*.5;
    blur += mix(.0, .1, smile);
    d1 = length(uv);
    s1 = S(.45, .45-blur, d1);
    d2 = length(uv-vec2(.1, -.2)*.7);
    s2 = S(.5, .5-blur, d2);
    float shadowMask = SAT(s1-s2);
    
    col = mix(col, vec4(0., 0., 0., 1.), S(.0, 1., shadowMask)*.5);
    
    col = mix(col, browCol, S(.2, .4,  browMask));
    
    return col;
}
```

最后为了修改整张脸的一个汇聚中心，对 uv 进行一个偏移修改，从而可以形成脸部跟随鼠标变化的效果。

```GLSL
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy;
    uv -= .5;
    uv.x *= iResolution.x / iResolution.y;
    
    vec2 m = iMouse.xy / iResolution.xy;
    m -= .5;

    uv -= m*(.25 - dot(uv, uv));
    float smile = cos(iTime)*.5+.5;
    fragColor = Smiley(uv, m, smile);
}
```

![face](/images/2022.8.26/2.gif)

## 小结

我们可以根据 `iTime` 给画面带来变化，主要通过修改 uv 实现想要的效果，要添加鼠标交互效果，可以利用 iMouse 获取鼠标点击的位置，然后适当地将其加入 uv 的偏移中，需要注意在使用 iMouse 时也不能忘记归一中心化的操作。

## References

- [Animating a smiley in ShaderToy](https://www.youtube.com/watch?v=vlD_KOrzGDc&list=PLGmrMu-IwbguU_nY2egTFmlg691DN7uE5&index=13)