---
title: UV扰动ShaderGraph
date: 2023-01-28 17:01:46
tags: 
- unity 
- Shader
- ShaderGraph
keywords: [unity, ShaderGraph, UV扰动, Shader]
---
![](/images/2023-01-28-unity-urp-uvdisturbance/head.png)

## 原理
用一张Plane，采样屏幕纹理，然后用一张噪声图+时间扰乱UV即可。

记得要在渲染管线中勾选`Opaque Texture`。
![](/images/2023-01-28-unity-urp-uvdisturbance/飞书20230128-114546.jpg)

## Shader

![](/images/2023-01-28-unity-urp-uvdisturbance/飞书20230128-114708.jpg)

![](/images/2023-01-28-unity-urp-uvdisturbance/飞书20230128-114852.jpg)




