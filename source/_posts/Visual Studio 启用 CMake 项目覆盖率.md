---
title: "Visual Studio 启用 CMake 项目覆盖率"
date: 2023-01-28T17:39:55+08:00
---

## 背景

在按照 [Use code coverage to determine how much code is being tested](https://learn.microsoft.com/en-us/visualstudio/test/using-code-coverage-to-determine-how-much-code-is-being-tested) 的方法尝试对 [cutest](https://github.com/qgymib/cutest) 进行行覆盖率测试时，发现无法获得行覆盖率结果。

<!-- more -->

## 解决方法

为了不污染项目的 `CMakeLists.txt`，所使用的解决方法必须仅限于修改 Visual Studio 的 `CMakeSettings.json` 文件。

+ Step 1. 打开 CMakeSettings.json

![打开 CMakeSettings.json](Visual Studio 启用 CMake 项目覆盖率/open_cmakesettings.png)

+ Step 2. 选中“显示高级变量”

![显示高级变量](Visual Studio 启用 CMake 项目覆盖率/select_advanced_variable.png)

+ Step 3. 添加 `/PROFILE` 链接选项

![添加链接选项](Visual Studio 启用 CMake 项目覆盖率/add_link_flag.png)

记得使用 `Ctrl-S` 保存 `CMakeSettings.json` 文件。

+ Step 4. 执行覆盖率测试

![执行覆盖率测试](Visual Studio 启用 CMake 项目覆盖率/perform_code_coverage.png)
