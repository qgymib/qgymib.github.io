---
title: "Install Arch Linux on a removable medium"
date: 2023-07-04T16:03:26+08:00
---

This post is a complement to [Install Arch Linux on a removable medium](https://wiki.archlinux.org/title/Install_Arch_Linux_on_a_removable_medium).

It is recommend to following introduction of [Arch Linux USB](https://mags.zone/help/arch-usb.html).

<!-- more -->

## Disable hibernation

To disable hibernation, use following command:

```bash
systemctl mask suspend.target hibernate.target sleep.target
```
