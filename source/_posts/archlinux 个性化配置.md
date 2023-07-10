---
title: archlinux 个性化配置
date: 2022-04-22T09:31:53+08:00
updated: 2023-07-10T20:38:28+08:00
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

## VMware 复制粘贴支持

在 KDE 中，即使使用 [Archlinux Wiki#7.6Drag and drop, copy/paste](https://wiki.archlinux.org/title/VMware/Install_Arch_Linux_as_a_guest#Drag_and_drop,_copy/paste) 中提供的方法也无法实现复制粘贴支持。

依照 [open-vm-tools#issue#568](https://github.com/vmware/open-vm-tools/issues/568#issuecomment-1575478623) 中提供的方法可解：

创建文件 `~/.config/systemd/user/app-vmware-user.service`：

```
[Unit]
Description=VMware User Agent
Documentation=https://github.com/vmware/open-vm-tools
# When the target is stopped or restarted, that will propagate to this unit too
PartOf=graphical-session.target
# Start this unit after the target is reached
After=graphical-session.target

[Service]
Type=forking
ExecStart=/usr/bin/vmware-user-suid-wrapper
# Group this service in the XDG app slice 
Slice=app.slice

[Install]
# When enabling this unit, a symlink is created so the target will start this service
WantedBy=graphical-session.target
```

启动此服务：

```
systemctl --user enable --now app-vmware-user.service
```
