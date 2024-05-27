---
title: 基于Playable的动画系统的简单实现
author: wanderer
date: 2024-05-22 17:38
categories: [Dev, Unity]
tags: [unity,animation,playable,animator]
render_with_liquid: false
---

想偷懒果然是不行的，找了一个第三方控制器，动画由`Animator`实现，初步体验感觉还行，改着改着就不对了，不是能很好的实现开发需求。
只有准备基于[Playables](https://docs.unity.cn/cn/current/Manual/Playables.html)实现一个真正可用的动画版本，还是要保证简单可用就行。


参考

[https://assetstore.unity.com/packages/tools/animation/animancer-lite-116516](https://assetstore.unity.com/packages/tools/animation/animancer-lite-116516)

[https://www.lfzxb.top/nkgmoba-animsystem-dawn-blossoms-plucked-at-dusk/](https://www.lfzxb.top/nkgmoba-animsystem-dawn-blossoms-plucked-at-dusk/)

[https://musoucrow.github.io/2022/10/16/anim_graph/](https://musoucrow.github.io/2022/10/16/anim_graph/)


设计

![](/assets/drawio/playable_animation.drawio.svg)