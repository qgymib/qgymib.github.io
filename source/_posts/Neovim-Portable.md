---
title: Neovim Portable
date: 2021-08-09T16:35:22+08:00
updated: 2021-08-18T11:57:02+08:00
tags:
categories: PortableApps
---

符合 `PortableApps.com` 格式的 NeoVim 封装配置。

<!-- more -->

## NeovimPortable.ini

```ini
[Launch]
ProgramExecutable=Neovim\bin\nvim-qt.exe
DirectoryMoveOK=yes
SupportsUNC=yes

[Activate]
Registry=true
XML=true

[Environment]
XDG_CONFIG_HOME=%PAL:DataDir%
XDG_DATA_HOME=%PAL:DataDir%
NEOVIM_HOME=%PAL:AppDir%\Neovim
HOME=%PAL:DataDir%
PATH=%NEOVIM_HOME%\bin;%PATH%
; for spacevim
PYTHON3_HOST_PROG=%PYTHON3_HOME%\python.exe

[RegistryKeys]
nvim-qt=HKCU\Software\nvim-qt

[FileWrite1]
Type=Replace
File=%PAL:DataDir%\nvim-data\rplugin.vim
Find=%PAL:LastDrive:ForwardSlash%%PAL:LastPackagePartialDir:ForwardSlash%
Replace=%PAL:Drive:ForwardSlash%%PAL:PackagePartialDir:ForwardSlash%
```

## 添加 Python3 支持

1. 下载embed版本的python3（[我使用的版本](https://www.python.org/ftp/python/3.9.6/python-3.9.6-embed-amd64.zip)），解压至 `/App/Tools/python-3.9.6` 目录
2. `NeovimPortable.ini` 中添加环境变量 `PYTHON3_HOME=%PAL:AppDir%\Tools\python-3.9.6`
3. `NeovimPortable.ini` 中追加 `PATH`: `%PYTHON3_HOME%;%PYTHON3_HOME%\Scripts`
4. 编辑文件 `python39._pth`，添加一行 `Lib`，并去除 `#import site` 前的注释，使其内容类似如下：
```
python39.zip
Lib
.

# Uncomment to run site.main() automatically
import site
```
5. 下载 [get-pip.py](https://pip.pypa.io/en/stable/installation/) 放入 `%PYTHON3_HOME%`，并在此目录下执行 `.\python .\get-pip.py`
6. 打开 NeovimPortable，在终端中执行 `pip install pynvim`

## 添加 nodejs 支持

1. 下载 `zip` 格式的nodejs并放入 `App/Tools` 目录
2. 编辑 `NeovimPortable.ini` 并添加环境变量 `%PAL:AppDir%/Tools/node`
3. 在 NeovimPortable 终端中执行 `npm install -g neovim`
4. 在 NeovimPortable 终端中执行 `npm install -g yarn`
