---
title: "在 Linux 客户机中启用 HGFS"
date: 2023-01-31T17:12:33+08:00
tags:
---

## 背景

VMware Workstation 中启用共享文件夹之后，`/mnt/hgfs`仅会出现一次，重启即消失，需要手动挂载。

<!-- more -->

## 修复方法1

来源：[Enabling HGFS Shared Folders on Fusion or Workstation hosted Linux VMs for open-vm-tools (74650)](https://kb.vmware.com/s/article/74650)

前置要求：
+ Open-vm-tools 版本大于等于 10.0.0
+ Linux 客户机支持 fuse
+ 内核版本 >= 3.10 (若 open-vm-tools 版本小于 10.3.0. 则内核版本必须大于等于 4.0)
+ 支持 systemd

1. 创建文件 `/etc/systemd/system/mnt-hgfs.mount`，内容如下：
  ```
  [Unit]
  Description=VMware mount for hgfs
  DefaultDependencies=no
  Before=umount.target
  ConditionVirtualization=vmware
  After=sys-fs-fuse-connections.mount
  
  [Mount]
  What=vmhgfs-fuse
  Where=/mnt/hgfs
  Type=fuse
  Options=default_permissions,allow_other
  
  [Install]
  WantedBy=multi-user.target
  ```

2. 创建文件 `/etc/modules-load.d/open-vm-tools.conf`，内容如下：
  ```
  fuse
  ```
  若文件已存在，则将上述内容追加。

3. 开启系统服务：
  ```
  sudo systemctl enable mnt-hgfs.mount
  ```

4. 确保 `fuse` 内核模块已加载：
  ```
  sudo modprobe -v fuse
  ```

## 修复方法2

来源：[/mnt/hgfs does not get mounted after reboots for shared folders](https://communities.vmware.com/t5/VMware-Fusion-Discussions/mnt-hgfs-does-not-get-mounted-after-reboots-for-shared-folders/td-p/2889090)

1. 建立 `/mnt/hgfs/` 文件夹：
  ```
  sudo mkdir -p /mnt/hgfs/
  ```

2. 挂载共享文件夹：
  ```
  sudo /usr/bin/vmhgfs-fuse .host:/ /mnt/hgfs/ -o subtype=vmhgfs-fuse,allow_other
  ```

3. 在 `/etc/fstab` 中添加永久映射：
  ```
  vmhgfs-fuse   /mnt/hgfs    fuse    defaults,allow_other    0    0
  ```
