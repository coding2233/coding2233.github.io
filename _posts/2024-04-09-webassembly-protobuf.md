---
title: 编译protobuf的WASM库
author: wanderer
date: 2024-04-09 14:24
categories: [Dev, Common]
tags: [protobuf,webassembly,wasm]
render_with_liquid: false
---

# 记录

虽然最新的插件在`WASM`暂时去掉`protobuf`作为通信协议，但是也准备记录一下，为`WASM`编译`protobuf`的操作。

[临时记录 https://github.com/coding2233/docker-arch-emscripten-protobuf](https://github.com/coding2233/docker-arch-emscripten-protobuf)


# v3.9.0

因为这里的版本还支持`makefile`编译
编译流程[https://github.com/coding2233/docker-arch-emscripten-protobuf/blob/main/.github/workflows/docker-publish.yml](https://github.com/coding2233/docker-arch-emscripten-protobuf/blob/main/.github/workflows/docker-publish.yml)

主要为`WASM`编译库文件的时候，需要优先应用几个`patch`， 这几个`patch`也是专门为当前版本实现的

# latest

未尝试

最新的版本，已经改成了`bazel`作为编译构建工具，需要修改对应的`patch`然后再尝试编译成`WASM`的库。
也可以尝试`xmake f --trybuild=bazel`

# xmake加载protobuf

```lua
--add_files("protocol/*.cc","protocol/lib/libprotobuf.so.o")
--add_includedirs("protocol","protocol/include")
```