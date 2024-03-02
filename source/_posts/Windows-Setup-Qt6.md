---
title: Windows 下安装 Qt6
date: 2024-03-02T19:36:40+08:00
updated: 2024-03-02T20:17:40+08:00
tags:
---

Windows 下建议使用 [MSYS2](https://www.msys2.org/) 来安装 Qt6。官方安装包总有各种各样的问题。

可使用 [MSYS2 镜像](https://mirrors.tuna.tsinghua.edu.cn/help/msys2/) 来加速包安装。

<!-- more -->

## 安装 Qt6

```bash
pacman -Syu
pacman -S VCS \
    mingw-w64-ucrt-x86_64-toolchain \
    mingw-w64-ucrt-x86_64-cmake \
    mingw-w64-ucrt-x86_64-clang \
    mingw-w64-ucrt-x86_64-qt6 \
    mingw-w64-ucrt-x86_64-qt-creator
```

## 配置 VScode

如果已经安装了 VSCode 且安装了 CMake 插件，则在 `settings.json` 中添加如下项目：

```json
{
    "cmake.additionalCompilerSearchDirs": [
        "C:/msys64/mingw32/bin",
        "C:/msys64/mingw64/bin",
        "C:/msys64/clang32/bin",
        "C:/msys64/clang64/bin",
        "C:/msys64/clangarm64/bin",
        "C:/msys64/ucrt64/bin"
    ]
}
```
