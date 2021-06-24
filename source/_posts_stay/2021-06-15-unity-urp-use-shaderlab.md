---
title: URP管线中如何使用ShaderLab
date: 2021-06-15 17:01:46
tags: 
- unity 
- urp 
- postprocessing
- shader
- 着色器
keywords: [unity, urp, 渲染管线, 着色器, shader]
---

## 1.URP管线初始化的几种方式

1.在创建工程时就选择URP管线。

2.对于已有的工程，想要转到URP管线，需要先到`Package Manager`安装`Universal RP`，然后在`Project`下右键->`Create->Rendering->URP->Pipline Asset`。创建完成后，`Project`下会多出`UniversalRenderPipelineAsset`和`UniversalRenderPipelineAsset_Renderer`两个文件，打开`Edit`->`Project Setting`，将`UniversalRenderPipelineAsset`文件分别拖入`Graphics/Scritable Render Pipline Settings`和`Quality/Rendering`下，这样项目就设置好了URP管线。

如果此时工程中已经有材质了，那么都会变成洋红色，因为URP不支持老的材质，需要对材质进行修改，点击`Edit->Render Pipline->URP->Upgrader Project Materials to UniversalRP Materials`，这样就可将工程中所有的unity内置的材质转为URP材质。如果是自定义的Shader，就需要手动一个一个修改了。

## 2.使用ShaderLab

