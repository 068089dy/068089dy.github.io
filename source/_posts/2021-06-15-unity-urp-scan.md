---
title: Unity后处理实现地形扫描特效（URP）
date: 2021-06-21 17:01:46
tags: 
- unity 
- urp 
- postprocessing
- shader
- 着色器
- 后处理
keywords: [unity, urp, 着色器, shader, 后处理, postprocessing]
---

![3](/images/2021-06-14-unity-urp-postprocessing/1.gif)

## 1.原理

每个`fragment`中都包含了深度信息，可以用深度信息和远近平面的值计算出每个`fragment`到`Camera`的距离，然后将指定距离范围内的点用不同的颜色显示。最后动态的调节这个指定范围就能实现扫描地形的效果了。

## 2.着色器具体操作


在片元着色器中，算出像素点到Camera的距离：

注：_ProjectionParams参数，float4类型，x = 1.0(或如果当前使用翻转投影矩阵渲染则为-1.0),y是相机的近平面,z是相机的远平面，w是1 / FarPlane
```c
float rawDepth = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, sampler_CameraDepthTexture, input.uv);
float depth = Linear01Depth(rawDepth, _ZBufferParams);
float dist = depth * _ProjectionParams.z;
```

然后在指定范围内给出相应的颜色：
```c
if (dist < _ScanDistance && dist > _ScanDistance-_ScanWidth) {
    float l = (dist - _ScanDistance + _ScanWidth) / _ScanWidth;
    col = col * (1-l) + _BaseColor * l;
}
```

完整的Shader代码：
```glsl
Shader "Custom/Scan"
{
	Properties
	{
		_MainTex("Base (RGB)", 2D) = "white" {}
		_BaseColor("Base Color",Color) = (1,1,1,1)
		_ScanDistance("ScanDistance", Range(0,100)) = 1
		_ScanWidth("_ScanWidth", Range(0, 10)) = 1
	}
	SubShader
	{
		Tags { "RenderType" = "Transparent" "RenderPipeline" = "UniversalPipeline" }
		Pass
		{
			HLSLPROGRAM

			#pragma vertex vert
			#pragma fragment frag

			#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

			sampler2D _MainTex;
			SAMPLER(sampler_MainTex);

			TEXTURE2D(_CameraDepthTexture);
			SAMPLER(sampler_CameraDepthTexture);
			float4 _CameraDepthTexture_TexelSize;
			float4 _BaseColor;
			float _ScanDistance;
			float _ScanWidth;

			struct Attributes
			{
				float4 positionOS : POSITION;
				float2 uv : TEXCOORD0;
			};

			struct Varyings
			{
				float4 vertex : SV_POSITION;
				float2 uv : TEXCOORD0;
			};

			Varyings vert(Attributes input)
			{
				Varyings output;

				VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
				output.vertex = vertexInput.positionCS;
				output.uv = input.uv;

				return output;
			}

			float4 frag(Varyings input) : SV_Target
			{
				float4 color = tex2D(_MainTex, input.uv);
				float rawDepth = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, sampler_CameraDepthTexture, input.uv);
				float depth = Linear01Depth(rawDepth, _ZBufferParams);
				float dist = depth * _ProjectionParams.z;
				if (dist < _ScanDistance && dist > _ScanDistance - _ScanWidth && depth < 1) {
					float l = (dist - _ScanDistance + _ScanWidth) / _ScanWidth;
					color = color * (1 - l) + _BaseColor * l;
				}
				return color;
			}
			ENDHLSL
		}
	}
	FallBack "Diffuse"
}
```


## 3.使用RendererFeature添加后处理

RendererFeature使用可以看[这篇](https://068089dy.github.io/2021/06/14/2021-06-14-unity-urp-postprocessing/#4-%E8%87%AA%E5%AE%9A%E4%B9%89%E5%90%8E%E5%A4%84%E7%90%86)。

新建一个RendererFeature，代码如下：
```c#
public class ScanRenderFeature : ScriptableRendererFeature
{
    ScanRenderPass renderPass;

    [System.Serializable]
    public class ScanSetting
    {
        public Material material = null;
    }
    public ScanSetting setting = new ScanSetting();

    public override void Create()
    {
        renderPass = new ScanRenderPass();
        renderPass.material = setting.material;
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        var src = renderer.cameraColorTarget;
        // renderer.cameraColorTarget就是管线渲染出来的图像，将它传给pass
        renderPass.Setup(src);
        renderer.EnqueuePass(renderPass);
    }

    public class ScanRenderPass: ScriptableRenderPass
    {
        public Material material = null;
        RenderTargetIdentifier passSource;
        
        public void Setup(RenderTargetIdentifier sour)
        {
            this.passSource = sour;
            // 需要在天空盒渲染完成后在处理，不然天空会变成黑的
            this.renderPassEvent = RenderPassEvent.AfterRenderingSkybox;
        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
        {
            if (!renderingData.cameraData.postProcessEnabled) return;
            CommandBuffer cmd = CommandBufferPool.Get("passTag");
            // 给passSource添加passMat材质后再输出到passSource
            cmd.Blit(passSource, passSource, material);
            context.ExecuteCommandBuffer(cmd);
            CommandBufferPool.Release(cmd);
        }
    }
}
```

将ScanRenderFeature添加至管线后，新建一个材质，添加上面的Shader，然后将材质配置到Setting中。调节材质的参数，就可以看到效果了。