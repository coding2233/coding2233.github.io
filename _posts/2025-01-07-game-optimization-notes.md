---
title: 游戏优化纪要
author: 船长
date: 2025-01-07 10:36
categories: [Dev, Game]
tags: [unity,game,engine]
render_with_liquid: false
---

### 不同平台的渲染科普

1. PC使用IMR(Immediate Rendering): 直接渲染管线,渲染一整张图
2. 移动端使用TBR(Tile-Based Rendering): 分块渲染管线
3. 并不绝对，不同的设备可能有不同的处理

[手机GPU和PC显卡有何不同？小芯片为何能玩大型游戏？](https://www.bilibili.com/video/BV1hk6kY1EwT/)

[https://github.com/Swung0x48/TriangleBin](https://github.com/Swung0x48/TriangleBin)

[参考资料: 移动GPU架构](https://wingstone.github.io/posts/2020-09-17-mobile-gpu-architecture/)

### 优劣势
1. TBR功耗低(带宽小)
2. IMR效率高
3. TBR MSAA性能基本没消耗，blending消耗也低
4. TBR后期会访问整张RT, 消耗大量带宽
5. TBR不支持大量顶点数据, FrameData空间有限
6. TBR渲染前一定要Clear,不然会从缓存重新读取上一帧RT
7. 避免alpha test破环渲染队列

### 优化方向
1. CPU占用(shader预编译、逻辑运算、线程...)
2. GPU占用(设备瓶颈、shader复杂度、顶点数量、DrawCall、OverDraw、透明效果、粒子特效、后期效果...)
3. 内存占用(GC、脚本内存管理、资源管理)
4. 性能分析自动化
5. 完善/自定义分析工具
6. 性能数据收集、分析
7. OverHead移动端遵循硬件指令大量少次
8. 剔除、合批
9. 

### GPU分析工具
1. [snapdragon-profiler](https://www.qualcomm.com/developer/software/snapdragon-profiler)
2. [Unity FrameDebugger](https://docs.unity3d.com/2021.3/Documentation/Manual/FrameDebugger.html)
3. [renderdoc](https://renderdoc.org/)