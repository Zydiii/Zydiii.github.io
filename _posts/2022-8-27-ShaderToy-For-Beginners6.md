---
layout: post
title: "面向新手的 ShaderToy —— 进阶篇（一）"
description: "RayMarching"
date: 2022-08-27
feature_image: /images/2022.8.27/0.gif
tags: [Shader, ShaderToy]
---

经常听到 Ray Marching 方法，但是从来没有实现过，这一篇就让我们入门 Ray Marching 吧~

<!--more-->

## RayMarching 计算距离

Ray Marching 也可以称为 SphereTracing，它主要是利用与最近物体相交得到的距离画一个球，这个球以内的区域都是安全区域，不会有其他物体，否则最近距离就会更小，然后在光线的路径上行进该最短距离，去寻找下一个最短距离。当行进的距离足够远或者相交的距离足够近时，意味着光线差不多到达了物体表面，这个时候就可以将在该光线上行进的距离作为相机在该方向上相交的物体的距离。

![](/images/2022.8.27/0.png)

所以其实关键的点在计算最近距离，这里用的是一个平面和球体，所以距离计算起来很简单，与平面的距离直接就是相机的高度，与球体的距离就是相机离球心的距离减去半径，二者之中取最短的即可。

![](/images/2022.8.27/1.png)

实现起来就很简单了，将像素颜色设置为求得的距离，因为距离会大于 1 导致画面全白，所以对距离进行一个缩小处理，就可以大致看出来一个场景了。

```GLSL
#define MAX_STEPS 100
#define MAX_DIST 100.
#define SURF_DIST .01

float GetDist(vec3 p)
{
    vec4 s = vec4(0, 1, 6, 1);
    
    float sphereDist = length(p - s.xyz) - s.w;
    float planeDist = p.y;
    
    float d = min(sphereDist, planeDist);
    return d;
}

float RayMarch(vec3 ro, vec3 rd)
{
    float d = 0.;
    
    for(int i = 0; i < MAX_STEPS; i++)
    {
        vec3 p = ro + rd * d;
        float ds = GetDist(p);
        d += ds;
        if(d > MAX_DIST || ds < SURF_DIST)
            break;
    }
    
    return d;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = (fragCoord - .5*iResolution.xy) / iResolution.y;

    vec3 col = vec3(0.);
    
    vec3 ro = vec3(0, 1, 0);
    vec3 rd = normalize(vec3(uv.x, uv.y, 1));
    
    float d = RayMarch(ro, rd);
    d /= 6.;
    col = vec3(d);

    fragColor = vec4(col,1.0);
}
```

![](/images/2022.8.27/2.png)

## 光照

那么光照如何计算呢？这里需要知道光照方向和法线方向，光照直接向量减法即可，而计算法线方向巧妙地用到了计算 slope 的方法，分别在 x、y、z 方向进行微小偏移，然后再计算偏移后的距离，最后用之前的距离减去这些距离可以得到在 x、y、z 方向上的 slope。（其实有一点没理解）

![](/images/2022.8.27/3.png)

将法线绘制出来看看效果：

```GLSL
vec3 GetNormal(vec3 p)
{
    float d = GetDist(p);
    vec2 e = vec2(.01, 0.);
    
    vec3 n = d - vec3(
                 GetDist(p - e.xyy),
                 GetDist(p - e.yxy),
                 GetDist(p - e.yyx));
                 
    return normalize(n);
}
```

![](/images/2022.8.27/4.png)

然后是光照的效果，这里将光照方向和法线方向的 dot 作为颜色值输出了：

```GLSL
float GetLight(vec3 p)
{
    vec3 lightPos = vec3(0., 5., 6.);
    lightPos.xz += vec2(sin(iTime), cos(iTime))*2.;
    vec3 l = normalize(lightPos - p);
    vec3 n = GetNormal(p);
    
    float dif = clamp(dot(n, l), 0., 1.);
    return dif;
}
```

![](/images/2022.8.27/5.png)

## 阴影

在 RayMarching 和 RayTracing 中阴影的判断都比较方便，我们可以从光线 hit 的点向光照做 Ray Marching，如果得到的距离小于该点实际与光源的距离，说明它们之间有其他物体遮挡，光源不能直接打到这个点，则说明该点处于阴影中。

![](/images/2022.8.27/6.png)

在计算时需要注意一点，这点在 smallpt 中也遇到了，就是在阴影时需要将该点沿着法线方向偏移一小段距离，否则无论如何都会计算到中间有物体相交，其实是因为判断时和自己相交了。

```GLSL
#define MAX_STEPS 100
#define MAX_DIST 100.
#define SURF_DIST .01

float GetDist(vec3 p)
{
    vec4 s = vec4(0, 1, 6, 1);
    
    float sphereDist = length(p - s.xyz) - s.w;
    float planeDist = p.y;
    
    float d = min(sphereDist, planeDist);
    return d;
}

float RayMarch(vec3 ro, vec3 rd)
{
    float d = 0.;
    
    for(int i = 0; i < MAX_STEPS; i++)
    {
        vec3 p = ro + rd * d;
        float ds = GetDist(p);
        d += ds;
        if(d > MAX_DIST || ds < SURF_DIST)
            break;
    }
    
    return d;
}

vec3 GetNormal(vec3 p)
{
    float d = GetDist(p);
    vec2 e = vec2(.01, 0.);
    
    vec3 n = d - vec3(
                 GetDist(p - e.xyy),
                 GetDist(p - e.yxy),
                 GetDist(p - e.yyx));
                 
    return normalize(n);
}

float GetLight(vec3 p)
{
    vec3 lightPos = vec3(0., 5., 6.);
    lightPos.xz += vec2(sin(iTime), cos(iTime))*2.;
    vec3 l = normalize(lightPos - p);
    vec3 n = GetNormal(p);
    
    float dif = clamp(dot(n, l), 0., 1.);
    float d = RayMarch(p + n * SURF_DIST * 2., l);
    if(d < length(lightPos-p))
        dif *= .1;
    return dif;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = (fragCoord - .5*iResolution.xy) / iResolution.y;

    vec3 col = vec3(0.);
    
    vec3 ro = vec3(0, 1, 0);
    vec3 rd = normalize(vec3(uv.x, uv.y, 1));
    
    float d = RayMarch(ro, rd);
    
    vec3 p = ro + rd * d;
    
    float dif = GetLight(p);
    
    col = vec3(dif);
    
    fragColor = vec4(col,1.0);
}
```

![](/images/2022.8.27/0.gif)

## 小结

本篇实现了 RayMarching 的基础算法，该算法是很多三维 ShaderToy 的基础，其实思想很简单，就是沿着光线去找和其他物体的相交情况，每次行进的距离是与其他物体相交的最短距离，因为这样可以保证在这个球形区域内没有其他物体，重复这样的过程直到距离足够远，或者相交的距离已经足够近。

求得了距离之后，使用求梯度的小技巧去计算物体表面的法线，以及和光线的相交情况，从而计算颜色。在计算阴影的时候，从交点往光源做 RayMarching，如果中间还有其他物体，则求得的距离一定小于实际与光源的距离，这个时候可以将颜色取很小形成阴影效果，这里需要注意的是需要将该点进行适当地偏移，从而避免求出来与自己相交。

## References

- [Animating a smiley in ShaderToy](https://www.youtube.com/watch?v=vlD_KOrzGDc&list=PLGmrMu-IwbguU_nY2egTFmlg691DN7uE5&index=13)