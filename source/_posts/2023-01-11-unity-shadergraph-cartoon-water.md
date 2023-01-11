---
title: Unity ShaderGraph卡通水
date: 2023-01-11 17:01:46
tags: 
- unity 
- ShaderGraph
- Shader
keywords: [unity, ShaderGraph, 水, Water]
---

![1](/images/2023-01-11-unity-shadergraph-cartoon-water/image_069_0001.png)

## 基本原理
边缘部分：用像素坐标减去深度，就能得到水面的深度，这样就能判断出来潜水区。
斑点部分就是一个动态的噪声。

![2](/images/2023-01-11-unity-shadergraph-cartoon-water/shaderGraph.jpg)