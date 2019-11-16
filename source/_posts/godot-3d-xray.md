---
title: Godot实现3D透视效果
date: 2019-11-16 17:01:46
tags: 
---

![3](/images/godot-3d-xray/a.gif)

参考：
[godot shading language文档](https://docs.godotengine.org/en/3.0/tutorials/shading/shading_language.html)
[Godot Advanced post-processing](https://docs.godotengine.org/en/3.1/tutorials/shading/advanced_postprocessing.html)
[OpenGL投影矩阵](https://zhuanlan.zhihu.com/p/74726302)

## 深度缓冲（z-buffer）与深度测试
`深度缓冲`中存放了每一个点到视点的深度信息，离视点越近深度越浅。按理说离得近的物体会挡住离得远的，那么深度越浅自然也应该优先显示出来，`深度测试`就是来做这件事的，如果当前点的深度比缓冲区中对应的点要浅，那么就覆盖显示。

## 如何透视
要想让被挡住的物体显示出来，只需关闭深度测试即可。在godot中，将渲染模式改为depth_test_disable。

```c
shader_type spatial;
render_mode depth_test_disable;
```
![1](/images/godot-3d-xray/xray-0.png)

为了标记透视物体，我们将物体渲染为红色，不然可能会产生物体本来就在前面的视觉错误。

```c
shader_type spatial;
render_mode depth_test_disable;

void fragment() {
    ALBEDO = vec3(1.0, 0.0, 0.0); // use red for material albedo
}
```
![2](/images/godot-3d-xray/xray-2.png)

## 区分被遮挡的部分
现在将被挡住的部分显示为红色，没有被挡住的不变化。这里使用了深度纹理来获取像素点的深度，当像素点被挡住时，depth会小于1，没被挡住时大于1。
```
shader_type spatial;

render_mode depth_test_disable;

void fragment() {
	float depth = texture(DEPTH_TEXTURE, SCREEN_UV).x;
	if (depth < 1.0){
		ALBEDO = vec3(1.0, 0.0, 0.0);
	}
}
```
![3](/images/godot-3d-xray/a.gif)
