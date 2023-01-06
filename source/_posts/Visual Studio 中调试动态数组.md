---
title: "Visual Studio 中调试动态数组"
date: 2023-01-06T13:37:51+08:00
---

来源：[在 Visual C++ 调试器监视窗口中展开数组指针](https://learn.microsoft.com/zh-cn/troubleshoot/developer/visualstudio/cpp/debuggers/expand-pointer-debugger-watch-window)

<!-- more -->

## 背景

我们可能需要在 Visual Studio 中查看一些动态申请的数组内容：

![variable_to_be_debug](Visual Studio 中调试动态数组/variable_to_be_debug.png)

## 解决方法

### Step 1. 添加动态数组作为监视变量

右键点击需要查看的动态数组，选择 _添加监视_ 将其加入到监视窗口。

![add_watchpoint](Visual Studio 中调试动态数组/add_watchpoint.png)

### Step 2. 修改监视变量作为数组

在监视窗口中双击对应变量，在变量名最后输入 `,4` 并按下 _Enter_。其中数字为动态数组实际长度。

![double_click](Visual Studio 中调试动态数组/double_click.png)

### Step 3. 展开数组查看内容

![finish](Visual Studio 中调试动态数组/finish.png)
