---
layout: post
title: "面向新手的 ShaderToy —— 进阶篇（五）"
description: "RayMarching"
date: 2022-09-06
feature_image: /images/2022.9.6/1.gif
tags: [Shader, ShaderToy]
---

<!--more-->

## 设置材质

如何给特定物体设置我们想要的材质呢？我们可以判断当前这个点是否为我们想要绘制的某个物体，思路就是判断我们最终求得的距离与哪个物体表面的距离是相等的。

Copy 一份 `GetDist` 改写为 `GetMat`，首先定义几种材质序号，然后在 `GetMat` 中判断最终的 d 属于哪个物体，返回对应的材质。需要注意的是，因为我们绘制球和细绳是在统一个函数中，所以光是一个距离并不能判断到底是小球还是细绳，因此我们还需要改写一下 `sdBall`，将其返回值增加一个维度，用于记录是小球还是细绳材质。

因为返回值改变了，对应地需要修改一下调用函数的地方，取出适合的值，最终根据得到的材质为像素点赋予不同的颜色：

```GLSL
const int MAT_BASE = 1;
const int MAT_BAR = 2;
const int MAT_BALL = 3;
const int MAT_LINE = 4;

vec2 sdBall(vec3 p, float a){
    p.y -= 1.01;
    p.xy *= Rot(a);
    p.y += 1.01;


    float ball = length(p)-.15;
    float ring = length(vec2(length(p.xy-vec2(0., .15))-.03, p.z))-.01;
    ball = min(ball, ring);
    
    p.z = abs(p.z);
    float line = sdLineSeg(p, vec3(0, .15, 0), vec3(0, 1.01, .4))-.005;
    
    float d = min(ball, line);
    
    return vec2(d, d == ball ? MAT_BALL : MAT_LINE);
}

float GetDist(vec3 p) {
    float base = sdBox(p, vec3(1, .1, .5))-.1;
    float bar = length(vec2(sdBox(p.xy, vec2(.8, 1.4))-.15, abs(p.z)-.4))-.04;
    
    float a = sin(iTime*3.),
          a1 = min(0., a),
          a5 = max(0., a);
          
    float b1 = sdBall(p-vec3(.6, .5, 0.), a1).x,
          b2 = sdBall(p-vec3(.3, .5, 0.), (a+a1)*.05).x,
          b3 = sdBall(p-vec3(0, .5, 0.), a*.05).x,
          b4 = sdBall(p-vec3(-.3, .5, 0.), (a+a5)*.05).x,
          b5 = sdBall(p-vec3(-.6, .5, 0.), a5).x;
    
    float balls = min(b1, min(b2, min(b3, min(b4, b5))));
    
    float d = min(base, bar);
    d = min(d, balls);
    
    d = max(d, -p.y); // cut off the bottom
    
    return d;
}

vec2 Min(vec2 a, vec2 b){
    return a.x < b.x ? a : b;
}

int GetMat(vec3 p) {
    float base = sdBox(p, vec3(1, .1, .5))-.1;
    float bar = length(vec2(sdBox(p.xy, vec2(.8, 1.4))-.15, abs(p.z)-.4))-.04;
    
    float a = sin(iTime*3.),
          a1 = min(0., a),
          a5 = max(0., a);
          
    vec2 b1 = sdBall(p-vec3(.6, .5, 0.), a1),
          b2 = sdBall(p-vec3(.3, .5, 0.), (a+a1)*.05),
          b3 = sdBall(p-vec3(0, .5, 0.), a*.05),
          b4 = sdBall(p-vec3(-.3, .5, 0.), (a+a5)*.05),
          b5 = sdBall(p-vec3(-.6, .5, 0.), a5);
    
    vec2 balls = Min(b1, Min(b2, Min(b3, Min(b4, b5))));
    
    float d = min(base, bar);
    d = min(d, balls.x);
    
    base = max(base, -p.y);
    
    d = max(d, -p.y); // cut off the bottom
    
    int mat = 0;
    
    if(d == base)
        mat = MAT_BASE;
    else if(d == bar)
        mat = MAT_BAR;
    else if(d == balls.x)
        mat = int(balls.y);
    
    return mat;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = (fragCoord-.5*iResolution.xy)/iResolution.y;
	vec2 m = iMouse.xy/iResolution.xy;

    vec3 ro = vec3(0, 3, -3);
    ro.yz *= Rot(-m.y*PI+1.);
    ro.xz *= Rot(-m.x*TAU);
    
    vec3 rd = GetRayDir(uv, ro, vec3(0,0.75,0), 2.);
    vec3 col = vec3(0);
   
    float d = RayMarch(ro, rd);

    if(d<MAX_DIST) {
        vec3 p = ro + rd * d;
        vec3 n = GetNormal(p);
        vec3 r = reflect(rd, n);

        float dif = dot(n, normalize(vec3(1,2,3)))*.5+.5;
        col = vec3(dif);
        
        int mat = GetMat(p);
        
        if(mat == MAT_BASE)
            col *= .1;
        else if(mat == MAT_BAR)
            col *= vec3(0, 0, 1);
        else if(mat == MAT_BALL)
            col *= vec3(1, 0, 0);
        else if(mat == MAT_LINE)
            col *= vec3(0, 1, 0);
        
    }
    
    col = pow(col, vec3(.4545));	// gamma correction
    
    fragColor = vec4(col,1.0);
}
```

![](../images/2022.9.6/8.png)

## 环境贴图

在下方点击 iChannel0，选择一个 Cubemap，Cubemap 就是由一个正方体构成的，每一个面有一张环境贴图，这些贴图可以无缝衔接：

![](../images/2022.9.6/9.png)

我们可以使用 `texture(iChannel, direction)` 直接获取这根光线对应的环境贴图像素颜色：

```GLSL
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = (fragCoord-.5*iResolution.xy)/iResolution.y;
	vec2 m = iMouse.xy/iResolution.xy;

    vec3 ro = vec3(0, 3, -3);
    ro.yz *= Rot(-m.y*PI+1.);
    ro.xz *= Rot(-m.x*TAU);
    
    vec3 rd = GetRayDir(uv, ro, vec3(0,0.75,0), 2.);
    vec3 col = texture(iChannel0, rd).rgb;
   
    float d = RayMarch(ro, rd);

    if(d<MAX_DIST) {
        vec3 p = ro + rd * d;
        vec3 n = GetNormal(p);
        vec3 r = reflect(rd, n);

        float dif = dot(n, normalize(vec3(1,2,3)))*.5+.5;
        col = vec3(dif);
        
        int mat = GetMat(p);
        
        if(mat == MAT_BASE)
            col *= .1;
        else if(mat == MAT_BAR)
            col *= vec3(0, 0, 1);
        else if(mat == MAT_BALL)
            col *= vec3(1, 0, 0);
        else if(mat == MAT_LINE)
            col *= vec3(0, 1, 0);
        
    }
    
    col = pow(col, vec3(.4545));	// gamma correction
    
    fragColor = vec4(col,1.0);
}
```

![](../images/2022.9.6/10.png)

当光线打到物体上后会发生反射折射，这里我们考虑光线的镜面反射，使用 `reflect(raydir, normal)` 得到反射后的光线，然后再对环境贴图做一次采样即可，并将最终得到的颜色赋给对应像素：

```GLSL
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = (fragCoord-.5*iResolution.xy)/iResolution.y;
	vec2 m = iMouse.xy/iResolution.xy;

    vec3 ro = vec3(0, 3, -3);
    ro.yz *= Rot(-m.y*PI+1.);
    ro.xz *= Rot(-m.x*TAU);
    
    vec3 rd = GetRayDir(uv, ro, vec3(0,0.75,0), 2.);
    vec3 col = texture(iChannel0, rd).rgb;
   
    float d = RayMarch(ro, rd);

    if(d<MAX_DIST) {
        vec3 p = ro + rd * d;
        vec3 n = GetNormal(p);
        vec3 r = reflect(rd, n);
        vec3 ref = texture(iChannel0, r).rgb;

        float dif = dot(n, normalize(vec3(1,2,3)))*.5+.5;
        col = vec3(dif);
        
        int mat = GetMat(p);
        
        if(mat == MAT_BASE)
            col = .1*ref;
        else if(mat == MAT_BAR)
            col = ref;
        else if(mat == MAT_BALL)
            col = ref;
        else if(mat == MAT_LINE)
            col *= .05;
    }
    
    col = pow(col, vec3(.4545));	// gamma correction
    
    fragColor = vec4(col,1.0);
}
```

![](../images/2022.9.6/11.png)



## 小结

## References

- [Newton's Cradle: Setting up Materials](https://www.youtube.com/watch?v=Agf188Q8EAc&list=PLGmrMu-IwbguU_nY2egTFmlg691DN7uE5&index=44)