---
title: 
date: 2019-11-16 17:01:46
tags: 
---

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