---
title: Unity使用RayMarching渲染一个黑洞
date: 2022-02-02 17:01:46
tags: 
- unity 
- RayMarching
- 黑洞
keywords: [unity, RayMarching, 黑洞]
---

![3](/images/2022-02-02-unity-make-blackhole/shaderDemo 2022_2_2 21_01_03.png)

### raymarching介绍

从Camera发出射线，射线不断前进，直到碰到物体，返回碰到坐标的颜色。
![](/images/2022-02-02-unity-make-blackhole/2022-02-02-21-07-37.png)
### 如何判断是否碰到物体呢？

使用距离场(SDF)，以球体为例:
```
// 伪代码
func SphereSDF(vec3 p, float radius){
    return length(p) - radius;
}
```
向上面这个函数中传入一个点坐标p，以及球体的半径radius，就会返回点p到球体的距离。
如果返回值大于0，则说明点p在球体外面。
如果返回值小于0，说明点p在球体内部。
这个函数就是球体的距离场函数。

### 使用Raymarching渲染一个球体

```c#
// 将shader材质挂载到任意一个mesh上
Tags { "RenderType"="Transparent" "Queue"="Transparent"}
Blend SrcAlpha OneMinusSrcAlpha
Cull Front
...
v2f vert (appdata v)
{
    v2f o;
    o.objPos = v.vertex;
    o.vertex = UnityObjectToClipPos(v.vertex);
    o.worldPos = mul(unity_ObjectToWorld, v.vertex);
    o.uv = TRANSFORM_TEX(v.uv, _MainTex);
    return o;
}

float sphere_sdf(float3 p, float radius){
    return length(p) - radius;
}

fixed4 frag (v2f i) : SV_Target
{
    float3 start = _WorldSpaceCameraPos;
    float3 dir = normalize(i.worldPos-start);
    float dt = 0.01;
    int hitSphereFlag = 0;
    float3 p = start;
    for (int j = 0; j < 300; j++){
        float hit = sphere_sdf(p, 1);
        if (hit < 0.01){
            return fixed4(1,1,0,1);
        }
        // 光线前进
        p += dir * dt;
    }
    return 0;
}
```

![3](/images/2022-02-02-unity-make-blackhole/2022-01-21-17-14-18.png)

这里渲染出来的球体是一个体积，即使摄像机移动到球体内部，也能渲染出球体内部的画面。
为了减少迭代次数，这里`dt`可以用`hit`代替。

### 为什么要使用Raymarching？

普通的模型渲染，是以点线面为基础的，这样的模型，就是一个空架子，从外面看很正常。但是如果视角移动到模型内部，就会发现里面是空的。一般的场景中，摄像机也不会移动到模型内部去看，所以也不需要对内部进行渲染。然而在一些特殊场景中，模型内部也需要渲染出来。比如飞机穿梭在云中，就需要渲染云的内部细节，Raymarching就可以实现体积云效果。

另外，普通的光栅化默认是按照`光线沿着直线传播`这条规律来进行渲染的，而像黑洞这样的天体，它周围的时空发生了严重的扭曲，导致光线也被弯曲了。光栅化无法渲染出光线弯曲的效果，而RayMarching可以在光线传播过程中人为的修改前进方向。

### 黑洞的视觉组成

1. 视界面：中间黑色部分的球体，球体半径称为史瓦西半径，光进入该半径后无法逃逸。
2. 吸积盘：类比行星环，只不过吸积盘旋转速度极快，接近光速，这样才能保证不落入视界面；由于运动太快，所以会发光发热，形成了高温的气态物质。另外由于光线弯曲，吸积盘后面被黑洞挡住的部分也可以被观测到，在视觉上会出现在视界面的上下部分。

### 黑洞视觉实现

视界面可以用一个球体渲染。
吸积盘可以当作一个面。

光线前进过程中如果碰到视界面，直接返回黑色；
如果碰到吸积盘平面，还需要继续前进（因为这里将吸积盘渲染为了半透明），还有可能再碰到视界面或者由于光线弯曲二次碰到吸积盘，两次碰到的颜色需要混合。

这里使用一个Cube或者Sphere作为载体，剔除前面，并且渲染为透明物体。

### 代码
```c
Shader "Unlit/BlackHoleV2Shader"
{
    Properties
    {
        // 吸积盘纹理
        _MainTex ("Texture", 2D) = "white" {}
        // 吸积盘半径
        _AcDiskRadius ("_AcDiskRadius", Float) = 4 
        // 吸积盘的厚度，0.001效果较佳
        _AcThicknessHalf ("_AcThickness", Float) = 0.001
        // 黑洞半径
        _BHRadius ("_BHRadius", Float) = 0.5
        // 最大迭代次数
        _StepLimit ("_StepLimit", int) = 200
    }
    SubShader
    {
        Tags { "RenderType"="Transparent" "Queue"="Transparent"}
        Cull Front
        Blend SrcAlpha OneMinusSrcAlpha
        LOD 100

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            // make fog work
            #pragma multi_compile_fog

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                UNITY_FOG_COORDS(1)
                float4 vertex : SV_POSITION;
                float4 objPos : TEXCOORD1;
                float4 worldPos : TEXCOORD2;
                float3 origin : TEXCOORD3;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;
            float _AcDiskRadius;
            float _BHRadius;
            float _AcThicknessHalf;
            int _StepLimit;

            

            float sphere_sdf(float3 p, float r){
                return length(p) - r;
            }

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.objPos = v.vertex;
                o.worldPos = mul(UNITY_MATRIX_M, v.vertex);
                o.origin = mul(UNITY_MATRIX_M, float4(0,0,0,1));
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                fixed4 col = 0;
                float3 start = _WorldSpaceCameraPos;
                float3 ray = normalize(i.worldPos.xyz - _WorldSpaceCameraPos);
                float3 p = start-i.origin;

                int hitAcFlag = 0; // 是否碰到吸积盘，0表示没有碰到，1表示碰到了一次，2表示碰到后又穿过，3表示第二次碰到
                int hitBHFlag = 0;
                float3 hitBHP;
                float3 hitBHViewRay;
                float3 hitAcP;
                float3 hitAcP2 = float3(0,0,0);
                float GM = 0.3;
                for (int j = 0; j < _StepLimit; j++){
                    // 计算是否进入大球
                    float hitAcSphere = sphere_sdf(p, _AcDiskRadius);
                    if (hitAcSphere < 0.001){
                        float hitBH = sphere_sdf(p, _BHRadius);
                        float hitRay = abs((p.y)/ray.y);
                        if (hitBHFlag == 0 && hitBH < 0.001) {
                            // 碰到了黑洞
                            hitBHFlag = 1;
                            hitBHP = p;
                            hitBHViewRay = ray;
                            break;
                        }
                        if (hitAcFlag == 0 && abs(p.y) <= _AcThicknessHalf) {
                            // 第一次碰到吸积盘
                            hitAcFlag = 1;
                            hitAcP = p;
                        }
                        if (hitAcFlag == 1 && abs(p.y) > _AcThicknessHalf) {
                            // 从吸积盘出来了
                            hitAcFlag = 2;
                        }
                        if (hitAcFlag == 2 && abs(p.y) <= _AcThicknessHalf) {
                            // 第二次碰到吸积盘
                            hitAcFlag = 3;
                            hitAcP2 = p;
                            break;
                        }
                        // 取最小值步进
                        float curDt = min(hitBH, hitRay);
                        // 这里如果curDt过大时，会导致弯曲不够正确，所以最大值取到0.1
                        curDt = min(0.1, curDt);
                        if (hitAcFlag == 1){
                            // 第一次进入盘，要出来
                            curDt = max(0.001, curDt); 
                        }
                        // 计算光线弯曲
                        p += curDt * ray;
                        float r2 = dot(p, p);
                        float3 a = GM/r2*normalize(-p);
                        ray += a*curDt;
                        ray = normalize(ray);
                    } else {
                        p += hitAcSphere * ray;
                    }
                }
                if (hitBHFlag == 1) {
                    // 碰到了黑洞
                    col = fixed4(0,0,0,1);
                    // 黑洞边缘发光
                    col.gb = pow(1-dot(normalize(hitBHP),-hitBHViewRay),3)*2;
                    // 靠近盘的地方发光
                    col.gb += pow(1-abs(hitBHP.y/_BHRadius),5);
                }
                if (hitAcFlag >= 1) {
                    // 碰到了吸积盘
                    float distH = length(hitAcP.xz);
                    // 纹理采样坐标
                    // v是距离中心距离，映射到0～1
                    // u是弧度值，映射到0～1
                    float v = smoothstep(0, 1, distH/_AcDiskRadius);
                    float u = (atan2(hitAcP.x, hitAcP.z)/UNITY_PI * v)/2 - _Time.y;
                    float tx = tex2D(_MainTex, float2(u,v)).r;
                    if (hitAcFlag == 3){
                        // 第二次碰到吸积盘
                        float distH2 = length(hitAcP2.xz);
                        float v2 = smoothstep(0, 1, distH2/_AcDiskRadius);
                        float u2 = (atan2(hitAcP.x, hitAcP.z)/UNITY_PI * v)/2 - _Time.y;
                        float tx2 = tex2D(_MainTex, float2(u2,v2)).r;
                        // 两次碰到吸积盘颜色混合
                        // 第一次碰到的颜色
                        col = col*(1-tx) + fixed4(0,1,1,1)*tx;
                        col.a *= abs(1-(distH-_BHRadius)/_AcDiskRadius)*5;
                        // 第二次碰到的颜色
                        fixed4 col1 = fixed4(0,1,1,1)*tx2;
                        col1.a *= abs(1-(distH2-_BHRadius)/_AcDiskRadius)*5;
                        // 混合
                        col = col1*(1-col.a) + col*col.a;
                    } else {
                        // 颜色混合
                        col = col*(1-tx) + fixed4(0,1,1,1)*tx;
                        if (hitBHFlag != 1) {
                            // 吸积盘越靠外透明度越低
                            col *= abs(1-(distH-_BHRadius)/_AcDiskRadius)*2;
                        }
                    }
                }
                return col;
            }
            ENDCG
        }
    }
}
```
