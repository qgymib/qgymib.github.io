---
title: MyLifeOrganized Portable
date: 2021-02-22T01:09:19+08:00
tags:
categories: PortableApps
---

符合 `PortableApps.com` 格式的 MyLifeOrganized 封装配置。

<!-- more -->

## MyLifePrganizedPortable.ini

```ini
[Launch]
ProgramExecutable=MLO\mlo.exe
CommandLineArguments=%PAL:DataDir%\organized.ml
DirectoryMoveOK=yes
SupportsUNC=yes

[Activate]
Registry=true
XML=true

[Environment]
HOME=%PAL:DataDir%

[RegistryKeys]
MyLifeOrganized.net=HKCU\Software\MyLifeOrganized.net

[RegistryCleanupIfEmpty]
1=HKCU\Software\MyLifeOrganized.net

[DirectoriesMove]
Roaming.MyLifeOrganized=%APPDATA%\MyLifeOrganized
Local.MyLifeOrganized=%LOCALAPPDATA%\MyLifeOrganized

[FilesMove]
mlo5.keym=%PAL:AppDir%\MLO

[DirectoriesCleanupIfEmpty]
1=%APPDATA%\MyLifeOrganized
2=%LOCALAPPDATA%\MyLifeOrganized

[FileWrite1]
Type=Replace
File=%PAL:DataDir%\settings\MyLifeOrganized.net.reg
Find=%PAL:LastDrive%%PAL:LastPackagePartialDir:DoubleBackslash%\\
Replace=%PAL:Drive%%PAL:PackagePartialDir:DoubleBackslash%\\
```

## 其他

将 MyLifeOrganized 的注册文件 `mlo5.keym` 放在 `Data` 目录下即可。
