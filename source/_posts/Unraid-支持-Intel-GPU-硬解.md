---
title: Unraid 支持 Intel GPU 硬解
date: 2021-02-10T22:50:50+08:00
tags:
---

编辑go文件，添加 modprobe i915，使其内容如下：

```bash
#!/bin/bash

# Intel GPU
modprobe i915

# Start the Management Utility
/usr/local/sbin/emhttp &
```
