---
title: 让 wsl 支持 cmake
date: 2021-11-03T23:38:41+08:00
tags:
---

来源：[Every call to configure_file fails on WSL: configure_file Problem configuring file](https://stackoverflow.com/questions/62879479/every-call-to-configure-file-fails-on-wsl-configure-file-problem-configuring-fi)

<!-- more -->

1. Create a file named wsl.conf in /etc/ with the following text:

```
# /etc/wsl.conf
[automount]
options = "metadata"
enabled = true
```

2. Reboot wsl:
```
wsl.exe -t Ubuntu // (or other e.g. Debian)
```
