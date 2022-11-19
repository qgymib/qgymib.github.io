---
title: x86_64 下使用 opcode 进行大于 4GB 地址的远距离跳转
date: 2021-11-04T11:26:45+08:00
tags:
---

解决方案来源：[JMP instruction - Hex code](https://stackoverflow.com/a/53876008)

在实现 [inline_hook](https://github.com/qgymib/inline_hook) 方案时，在 x86_64 平台下需要支持大于 4GB 地址范围的远距离跳转，然而常用跳转指令存在几个限制：
1. 依赖寄存器或内存
2. 不依赖寄存器或内存的则跳转距离过短

<!-- more -->

解决方案：

```
ff 25 00 00 00 00           jmp qword ptr [rip]      jmp *(%rip)
yo ur ad dr re ss he re     some random assembly
```
