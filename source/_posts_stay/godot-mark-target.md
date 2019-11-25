---
title: 
date: 2019-11-16 17:01:46
tags: 
---

## 获取物体在观察空间的坐标


首先，用物体的世界坐标减去摄像机的世界坐标，得到物体相对于摄像机的位置。

接下来，就要针对摄像机的旋转来变换物体的坐标，就是一个简单的基变换，（由于刚才已经得到了物体相对于摄像机的位置，所以现在相当于将摄像机移动到了世界原点，物体也做了相应的移动），有如下公式：

```
世界坐标系·物体的世界坐标 = 相机坐标系·物体相对于相机的坐标
```

化简得：

```
物体相对于相机的坐标 = 物体的世界坐标·相机坐标系的转置
```

```
var relation_pos = $MeshInstance.global_transform.origin - $Camera.global_transform.origin
var relation_coord = $Camera.global_transform.basis.transposed() * relation_pos
```

## 获取物体投影到近平面的坐标
```
var proportion_y = (relation_coord.y)/(relation_coord.z)
var proportion_x = (relation_coord.x)/(relation_coord.z)
```

```
func _process(delta):
	var relation_pos = $MeshInstance.global_transform.origin - $Camera.global_transform.origin
	var relation_coord = $Camera.global_transform.basis.transposed() * relation_pos
	print($Control/Sprite.position.y)
	var proportion_y = (relation_coord.y)/\
		(relation_coord.z)
	var proportion_x = (relation_coord.x)/\
		(relation_coord.z)
	$Control/Sprite.position.y = 300+300* proportion_y
	$Control/Sprite.position.x = 514-300* proportion_x
```