---
title: "CMake 中将 list 作为参数传递给外部命令"
date: 2023-02-04T15:09:23+08:00
tags: [CMake]
---

> 来源：[How to pass list variable in CMake's execute_process?](https://stackoverflow.com/questions/52206191/how-to-pass-list-variable-in-cmakes-execute-process)

```cmake
string(REPLACE ";" "," escaped_list "${source_list}")
```

<!-- more -->

## 原理

CMake 中的 list 本质上是使用 `;` 分割的字符串，`;` 在 CMake 中有特殊意义。因此如果想要将 list 传递给外部程序，需要将其中的 `;` 转义。

## 转义序列选择

转义序列应该依据使用场景来进行选择。由于 `;` 的特殊性，如果将 `;` 转义为 `\\;`，则其特性表现在不同平台上可能不一样：

+ 在 Linux/Unix 平台，转义表现正常
+ 在 Windows 平台，转义表现为多了两个 `\\` 字符

因此，除非你的 `CMakeLists.txt` 不涉及跨平台，否则不应将其转义为 `\\;`。
