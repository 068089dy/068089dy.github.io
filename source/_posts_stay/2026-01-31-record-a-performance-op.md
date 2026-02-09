---
title: 记录一次Unity性能优化
date: 2026-01-31 16:54:46
tags: 
- unity 
- ECS
- 面向数据
- 高性能
- DOTS
keywords: [unity, ECS, DOTS, 面向数据]
---
环境：
Unity：6000.2.10f1

性能优化三大类
合批
 DOTS Instancing
 GPU Instancing
 SRP Batcher
 动态合批
 静态合批
减少Overdraw
减少资源大小
 


## 优化前
![](/images/2023-01-28-unity-urp-uvdisturbance/op1.png)
## 优化后

## 方法一：合批
使用同一个材质，开启GPU Instancing
Enable GPU Instancing
降低材质复杂度 lit-> simplelit
关闭阴影



* DOTS Instancing：按照实体区分材质属性，不是shader级别的instancing，是DOTS专用的instancing技术，能大幅降低DrawCall数量，同时节省CPU和GPU资源开销，提升渲染性能。
```
#pragma target 4.5
#pragma multi_compile _ DOTS_INSTANCING_ON
// 不用加DOTS_INSTANCING_BUFFER_START，

```
* GPU Instancing：一次 DrawCall 画 N 个相同 Mesh。降低DC，同Mesh同材质(可以不同参数，Shader要支持Instancing)。
```
// 通过 MaterialPropertyBlock 设置每个实例的不同参数
#pragma multi_compile_instancing
// ===== GPU Instancing 属性 =====
UNITY_INSTANCING_BUFFER_START(Props)
    UNITY_DEFINE_INSTANCED_PROP(float4, _BaseColor)
UNITY_INSTANCING_BUFFER_END(Props)

float4 color = UNITY_ACCESS_INSTANCED_PROP(Props, _BaseColor);
```
* SRP Batcher：减少 CPU → GPU 的状态切换开销，只是 加速 DrawCall 提交，让 SetPass 更快，不降低DC。同 Shader Variant 即可。对 CPU 极其友好
```shaderlab
CBUFFER_START(UnityPerMaterial)
    float4 _BaseColor;
CBUFFER_END
```
* 动态合批：同一种材质，动态物体
* 静态合批：同一种材质，静态物体

DOTS Instancing + SRP Batcher 是叠加关系
贴图合并

| 技术                  | 减 DrawCall | CPU 省 | GPU 省 | DOTS 可用 | 现代推荐  |
| ------------------- | ---------- | ----- | ----- | ------- | ----- |
| 动态合批                | ✅          | ❌     | ❌     | ❌       | ❌     |
| 静态合批                | ✅          | ❌     | ⚠️    | ❌       | ❌     |
| GPU Instancing      | ✅          | ✅     | ✅     | ⚠️      | ⚠️    |
| SRP Batcher         | ❌          | ✅     | ❌     | ✅       | ✅     |
| **DOTS Instancing** | ✅✅         | ✅✅    | ✅     | ✅       | ⭐⭐⭐⭐⭐ |


什么是SetPass？
SetPass是渲染管线中的一个重要步骤，它涉及将当前的渲染状态（如着色器程序、材质属性等）设置到GPU上，以便进行绘制操作。每当渲染引擎需要切换到不同的材质或着色器时，就会触发一次SetPass调用。这一过程通常比较耗时，因为它涉及与GPU的通信和状态切换。
## 方法二：减少Overdraw
通过减少半透明材质的使用，优化场景中的遮挡关系，降低Overdraw。
Overdraw是指在渲染过程中，同一像素被多次绘制的现象。高Overdraw会导致GPU负担加重，影响渲染性能。通过优化场景中的遮挡关系，减少不必要的半透明材质使用，可以有效降低Overdraw，从而提升整体渲染效率。
UI的OverDraw同样的理，尽量减少重叠的UI元素，使用更高效的UI渲染技术（如Canvas的分层渲染）来优化UI的Overdraw问题。
## 方法三：使用性能分析工具
利用Unity Profiler、Frame Debugger等工具，找出性能瓶颈，针对性地进行优化。
性能分析工具可以帮助开发者深入了解应用程序的运行状况，识别出性能瓶颈所在。通过分析CPU和GPU的使用情况、内存分配等指标，开发者可以有针对性地进行优化，从而提升整体性能。
## 总结



#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"