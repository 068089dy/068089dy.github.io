---
title: Unity开发阶段快速编译
date: 2026-01-28 16:54:46
tags: 
- unity
- 开发
- 快速编译
- DOTS
keywords: [unity, DOTS，快速编译, 开发]
---
环境：
Unity：6000.2.10f1


## 1.关闭自动检测资源 / 代码的修改并刷新项目
Edit → Preferences → Asset Pipeline
设置：
关闭 Auto Refresh
Import Worker Count：1
* 关闭后改代码需要`Ctrl+R`才会重新编译代码

## 2.关闭Rider自动刷新Unity
Rider → Settings → Unity Engine 
关闭Automatically refresh assets

## 3.关闭Unity自动保存
Project Setting->Editor->PlayMode
关闭Allow Auto Save

## 3.关闭Burst编译（性能测试/打包时需要开启）
Jobs → Burst 
关闭Enable Compilation


## 4.（主要）关闭Reload Domain and Scene
Project Settings → Editor
找到 Enter Play Mode Settings
Do not reload domain or Scene
* 关闭后改代码需要`Ctrl+R`才会重新编译代码，而且会有静态变量不刷新的问题