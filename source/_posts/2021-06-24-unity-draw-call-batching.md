---
title: Unity批处理
date: 2021-06-24 17:01:46
tags: 
- unity 
- DrawCall 
- 批处理
keywords: [unity, DrawCall, 批处理]
---

## 1.为什么使用批处理？

商店进货时，进货量越大，总的运输成本就越低。比如一次运一千箱；和每次运一箱，运一千次，肯定后者的开销更大。

图形渲染时也一样，CPU在向GPU发起DrawCall时，每次要尽可能多的传输数据，来减少调用次数。

## 2.如何减少DrawCall

1.在建模时，将大量的模型合并成一个网格，这样CPU就可以一次DrawCall把数据传给GPU。但是这种方法不够灵活，模型在unity中无法修改。

2.使用批处理。

## 3.批处理

要使用批处理，需要先在Edit->ProjectSetting->Player->OtherSetting下勾选`Static Batching`和`Dynamic Batching`。

批处理的物体必须拥有相同的材质。（在建模时，将所有模型的贴图放到一个图集中，这样就可以使用同一个材质了。）

### 静态批处理

勾选物体`Inspecter->Static`下的`Batching Static`，unity就会在运行初始对该物体进行静态批处理。

静态批处理的物体在运行时是不可以移动的，并且会消耗大量内存。

### 动态批处理

对于动态的物体，unity会自动的进行动态批处理，不过只会针对点数较小的模型。
缺点：会增加CPU运行时的开销。


参考：https://docs.unity3d.com/cn/current/Manual/DrawCallBatching.html