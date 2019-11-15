---
title: \[Godot\]实现3D透视效果
date: 2016-10-13 12:08:46
tags: 
---

参考：
[godot shading language文档](https://docs.godotengine.org/en/3.0/tutorials/shading/shading_language.html)
[Godot Advanced post-processing](https://docs.godotengine.org/en/3.1/tutorials/shading/advanced_postprocessing.html)
[OpenGL投影矩阵](https://zhuanlan.zhihu.com/p/74726302)
## 深度缓冲（z-buffer）与深度测试
`深度缓冲`中存放了每一个点到视点的深度信息，离视点越近深度越浅。按理说离得近的物体会挡住离得远的，那么深度越浅自然也应该优先显示出来，`深度测试`就是来做这件事的，如果当前点的深度比缓冲区中对应的点要浅，那么就覆盖显示。

## 如何透视
要想让被挡住的物体显示出来，只需将深度测试设置为always模式。在godot中，将渲染模式改为depth_draw_always。

```c
shader_type spatial;
render_mode depth_test_disable;
```

## 如何区分透视物体
为了标记透视物体，需要对物体做一些处理，不然可能会有物体本来就在前面的视觉错误。可以将物体渲染为一个纯色物体，并且关闭物体的光照和阴影。

```c
shader_type spatial;
render_mode depth_test_disable, unshaded;

void fragment() {
    ALBEDO = vec3(1.0, 0.0, 0.0); // use red for material albedo
}
```