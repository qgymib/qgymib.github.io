---
title: Windows 10 下添加路由表
date: 2021-08-23T13:53:49+08:00
tags:
---

使用管理员权限执行以下指令：

```
route add 10.0.0.0 mask 255.255.0.0 172.16.0.1
```
