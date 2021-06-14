---
title: Unity中URP管线实现后处理的几种方式
date: 2021-06-14 17:01:46
tags: 
- unity 
- urp 
- postprocessing
- 后处理
keywords: [unity, urp, 渲染管线, 后处理]
---

## 1.内置渲染管线（Built-in）中的后处理
通过在Camera上挂载类似这样的脚本：
```c#
public class ExampleClass : MonoBehaviour 
{ 
    public Material mat;
    void OnRenderImage(RenderTexture src, RenderTexture dest) 
    {
        Graphics.Blit(src, dest, mat); 
    } 
}
```
就可以给相机添加后处理效果了。

## 2.使用RenderTexture
新建一个`RenderTexture`（Create->RenderTexture）,然后拖入到Camara下的OutputTexture配置项中，这样Camara就会将渲染好的图像输出到RenderTexture中，再新建一个材质，将RenderTexture拖入材质的BaseMap中，将材质放到一张Plane上（自定义的Mesh也可以），就可以看到Plane上已经显示出Camera拍摄的图像了。然后自己再自定义后处理shader，应用到材质上即可。

## 3.使用URP自带的一些后处理效果

### 在场景中新建Volume

URP提供了四种Volume：
* Global Volume：场景中所有开启了`Rendering`->`PostProcessing`选项相机都会受到添加后处理效果。
* Box Volume/Sphere Volume/Convex Mesh Volume：这三种Volume都有Collider组件，只有在碰撞范围内的Camera会添加后处理效果。

### 在Volume组件中新建Profile
然后点击Add Override就可以添加后处理效果了。

### 常用的Override介绍
Bloom：泛光效果
Depth Of Field：景深，相机对焦效果
Chromatic Aberration：色差
film grain：胶片颗粒
lens Distortion：透镜畸变
Motion Blur：运动模糊
vignette：晕映（有点望远镜黑圈的意思）

## 4.自定义后处理

找到项目中正在使用的PiplineAsset文件（在Edit->ProjectSetting->Quality->Rendering下可以找到项目正在使用的渲染管线文件），如果没有的话，新建一个Pipline Asset（Project下，右键Create->Rendering->URP->Pipline Asset）,创建完成后，项目中除了PiplineAsset文件，还会多出一个Forward Renderer文件，在Forward Renderer文件中点击Add Renderer Feature，就可以添加自定义的后处理效果了。

现在项目中还没有自定义的后处理效果，创建一个。

首先，新建一个继承ScriptableRendererFeature的脚本

```c#
public class CustomRender : ScriptableRendererFeature
{
    CustomRenderPass customRenderPass;
    [System.Serializable]
    public class BlurSetting
    {
        public RenderPassEvent Event = RenderPassEvent.AfterRenderingTransparents;
        public Material material = null;

    }

    public BlurSetting setting = new BlurSetting();

    public override void Create()
    {
        customRenderPass = new CustomRenderPass("a");
        customRenderPass.passMat = setting.material;
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        var src = renderer.cameraColorTarget;
        // renderer.cameraColorTarget就是管线渲染出来的图像，将它传给pass
        customRenderPass.Setup(src);
        renderer.EnqueuePass(customRenderPass);
    }
}
```

然后，新建Pass脚本

```c#
public class CustomRenderPass : ScriptableRenderPass
{
    public Material passMat = null;
    private RenderTargetIdentifier passSource { get; set; }

    string passTag;

    public CustomRenderPass(string tag)
    {
        passTag = tag;
    }

    public void Setup(RenderTargetIdentifier sour)
    {
        this.passSource = sour;
    }

    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        CommandBuffer cmd = CommandBufferPool.Get(passTag);
        // 给passSource添加passMat材质后再输出到passSource
        cmd.Blit(passSource, passSource, passMat);
        context.ExecuteCommandBuffer(cmd);
        CommandBufferPool.Release(cmd);
    }
}
```

完成后点击`Add Renderer Feature`，就可以看到CustomRender已经出现在选项中了。添加效果后，新建一个材质放到Setting的配置项中，就可以看到效果了。

现在，这个后处理已经添加到了渲染管线中，但是这是一个全局效果，不好管理，如果想在Volume中管理，还需要新建Volume脚本。

新建一个Volume脚本:
```c#
[System.Serializable, VolumeComponentMenu("CustomVolume/CustomRender")]
public sealed class CustomVolume1 : VolumeComponent, IPostProcessComponent
{
    [Tooltip("是否开启效果")]
    public BoolParameter enableEffect = new BoolParameter(true);
    public bool IsActive() => enableEffect==true;
    public bool IsTileCompatible() => false;
}
```

保存后在`Volume`组件中`Add Override`，已经可以看到`CustomVolume1`效果出现在选项中了。

然后修改CustomRenderPass的Excute方法：
```c#

public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
{
    CommandBuffer cmd = CommandBufferPool.Get(passTag);
    cmd.Blit(passSource, passSource, passMat);

    // 找到CustomVolume1组件，如果没找到或者未开启，直接return
    var stack = VolumeManager.instance.stack;
    CustomVolume1 customVolume1 = stack.GetComponent<CustomVolume1>();
    if (customVolume1 == null){return;}
    if (!customVolume1.IsActive())return;

    context.ExecuteCommandBuffer(cmd);
    CommandBufferPool.Release(cmd);
}

```

参考：
* https://johnyoung404.github.io/2019/12/13/Unity%E4%B8%AD%E7%9A%84%E5%90%8E%E5%A4%84%E7%90%86-post-processing/
* https://zhuanlan.zhihu.com/p/149635502
* https://www.jianshu.com/p/a5456036ab95