---
title: openSUSE 在终端中支持 OneClickInstall
date: 2021-09-29T22:58:32+08:00
tags:
---

openSUSE 中关于 OneClickInstall 的 [FAQ](https://en.opensuse.org/openSUSE:One_Click_Install_UserFAQ) 目前是过期的，`/sbin/OCICLI` 已经不再提供。

<!-- more -->

## 安装

```bash
sudo zypper install yast2-metapackage-handler
```

## 使用

```bash
OneClickInstallCLI <YMP URL>
```
