---
title: archlinux 个性化配置
date: 2022-04-22T09:31:53+08:00
updated: 2022-06-10T13:52:46+08:00
tags:
---

## 安装

[Installation guide](https://wiki.archlinux.org/title/Installation_guide)

或

[Arch Linux USB](https://mags.zone/arch-usb.html)

<!-- more -->

## 通用配置

[General recommendations](https://wiki.archlinux.org/title/General_recommendations)

+ 添加用户
+ 安装 Xorg / Xfce / lightdm

```
# pacman -S xorg-server \
    plasma-meta kde-system-meta kde-utilities-meta \
    xfce4 xfce4-goodies alacarte \
    lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings

# systemctl enable lightdm.service && systemctl start lightdm.service
```

## AUR && archlinuxcn

[Arch Linux 中文社区仓库](https://www.archlinuxcn.org/archlinux-cn-repo-and-mirror/)

镜像：
```
## 清华大学 (北京) (ipv4, ipv6, http, https)
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

```
# pacman -Syy && pacman -S yay
```

## 中文化

[Localization (简体中文)/Simplified Chinese (简体中文)](https://wiki.archlinux.org/title/Localization_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)/Simplified_Chinese_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

