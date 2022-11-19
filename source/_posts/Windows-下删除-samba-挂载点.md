---
title: Windows 下删除 samba 挂载点
date: 2021-02-11T10:02:02+08:00
tags:
---

## 查看当前网络连接

```powershell
PS C:\Users\qgymib>net use
会记录新的网络连接。


状态       本地        远程                      网络

-------------------------------------------------------------------------------
OK           X:        \\192.168.1.221\I-Userfiles\XDRHW
                                                Microsoft Windows Network
OK           Y:        \\192.168.1.18\592        Microsoft Windows Network
OK           Z:        \\192.168.1.66\6001413        Microsoft Windows Network
OK                     \\192.168.2.209\UserDesktop
                                                Microsoft Windows Network
已断开                 \\192.168.1.100\share       Microsoft Windows Network
```

## 删除指定网络连接

```powershell
PS C:\Users\qgymib>net use \\192.168.1.100\share /delete
\\192.168.1.100\share 已经删除。
```
