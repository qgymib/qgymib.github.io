---
title: "使用 flatpak 分发的 vscode"
date: 2023-06-26T10:33:56+08:00
---

[flatpak](https://flatpak.org) 的虚拟化机制使得 vscode 无法直接使用宿主机中所安装的工具链。

一般情况下，为了使用指定的工具链，有如下几种方法：
1. 安装 `org.freedesktop.Sdk.Extension.*` 系列 SDK 扩展：[URL](https://github.com/flathub/com.visualstudio.code)。
2. 使用 toolbox 在宿主机创建隔离环境，并连接 vscode：[URL](https://raduzaharia.medium.com/using-the-vscode-flatpak-distribution-a275d59ff1c7)

现在介绍一种更简单的方法：
1. 确保宿主机 [开启 SSH 服务](https://wiki.archlinux.org/title/OpenSSH#Server_usage)
2. 安装 [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) 扩展
3. 在 `Remote - SSH` 中连接到 `localhost:22`
