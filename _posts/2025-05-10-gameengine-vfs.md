---
title: 虚拟文件系统
author: 船长
date: 2025-05-19 23:01
categories: [Dev,GameEngine]
tags: [c++,vfs]
render_with_liquid: true
math: true
---

# 一、Virtual File System C++

[https://github.com/nextgeniuspro/vfspp](https://github.com/nextgeniuspro/vfspp)

# 二、基本功能

1. 挂载本地文件系统

```c++
auto rootFS = std::make_unique<NativeFileSystem>(GetBundlePath() + "Documents/");
//
vfs->AddFileSystem("/", std::move(rootFS));
```

可以挂载Editor中的工程目录，也可以挂在运行平台的读写目录

2. 挂载zip文件(只读)

挂载打包后的资源，或者`Android`的内置资源

3. 挂载内存

Editor或者运行时的场景


# 三、 一切皆文件

尝试将资源、场景渲染、逻辑都基于虚拟文件系统实现