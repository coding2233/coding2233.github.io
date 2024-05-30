---
title: Unity热更代码以及资源更新管理的一点思路
author: wanderer
date: 2024-05-30 18:45
categories: [Dev, Unity]
tags: [unity,asset,injectfix,ifix,hybridclr,xlua,ilrunrime]
render_with_liquid: false
---

多年前在首次接触ILRuntime的时候，才真正了解热更代码对游戏开发有多大的意义。之前虽然听说过代码热更新，但是实际上没有觉得热更代码是一个优先级很高的选择(主要也跟自己当时项目和经验的限制有关)。但是自从用了热更新后，我又从一个极端跑到了另一个极端，一直固执的去实现全热更，虽然有些项目可以做到只需要发布一次，业务可以随时更新。但是还是无法达到一个完美的状态，为了全热更不得不妥协很多东西，比如原生平台的功能开发交互、C/C++的插件扩展。

现在我大概是明白了，全热更实际上只是一个伪需求。这里我准备把代码更新和资源更新、版本管理做一个简单的整理。

![](/assets/drawio/unity_asset.drawio.svg)
