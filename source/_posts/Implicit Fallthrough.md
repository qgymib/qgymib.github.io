---
title: "Implicit Fallthrough"
date: 2021-07-28T10:17:35+08:00
---

## 背景
GCC7 以上加入了对于 `switch ... case` 语句中 `fall through` 的检测。此检测项在开启 `-Wextra` 编译选项时会产生告警：

```
error: this statement may fall through [-Werror=implicit-fallthrough=]
```

<!-- more -->

## 解决方案

### 添加注释
在 `case` 语句的末尾添加注释 `//-fallthrough` 。此注释能够同时消除 `lint` 的告警。
示例如下：
```c
switch(a)
{
    case 0:
        b = 0;
        //-fallthrough
    case 1:
        c = 1;
        //-fallthrough
    default:
        d = 2;
        break;
}
```

### 修改编译选项
编译选项增加 `-Wimplicit-fallthrough` 或 `-Wno-implicit-fallthrough` 。

## GCC手册
[GCC Online Doc](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
