---
title: Eclipse Portable
date: 2021-02-12T01:10:07+08:00
tags:
categories: PortableApps
---

符合 `PortableApps.com` 格式的 Eclipse 封装配置。

<!-- more -->

## EclipsePortable.ini

```ini
[Launch]
ProgramExecutable=eclipse\eclipse.exe
CommandLineArguments=-data %PAL:DataDir%\workspace -configuration %PAL:DataDir%\settings\configuration -vmargs -Duser.home=%PAL:DataDir%\settings\home -Djava.io.tmpdir=%PAL:DataDir%\.tmp
DirectoryMoveOK=yes
SupportsUNC=yes

[Activate]
Registry=true
XML=true

[Environment]
JAVA_HOME=%PAL:AppDir%\openjdk
PATH=%JAVA_HOME%\bin;%PATH%
HOME=%PAL:DataDir%\settings\home

[FileWrite1]
Type=Replace
File=%PAL:DataDir%\settings\p2\pools.info
Find=%PAL:LastDrive%%PAL:LastPackagePartialDir%\
Replace=%PAL:Drive%%PAL:PackagePartialDir%\

[FileWrite2]
Type=Replace
File=%PAL:DataDir%\settings\p2\profiles.info
Find=%PAL:LastDrive%%PAL:LastPackagePartialDir:DoubleBackslash%\\
Replace=%PAL:Drive%%PAL:PackagePartialDir:DoubleBackslash%\\

[FileWrite3]
Type=Replace
File=%PAL:DataDir%\settings\home\.p2\pools.info
Find=%PAL:LastDrive%%PAL:LastPackagePartialDir%\
Replace=%PAL:Drive%%PAL:PackagePartialDir%\

[FileWrite4]
Type=Replace
File=%PAL:DataDir%\settings\home\.p2\profiles.info
Find=%PAL:LastDrive%%PAL:LastPackagePartialDir:DoubleBackslash%\\
Replace=%PAL:Drive%%PAL:PackagePartialDir:DoubleBackslash%\\
```

## DefaultData

需要创建.tmp空目录，否则eclipse启动失败。
