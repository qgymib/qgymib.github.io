---
title: "Valgrind Gdbserver 使用方法"
date: 2021-07-27T22:53:55+08:00
---

Valgrind 支持 gdbserver (AKA. `vgdb`)，这使得在运行valgrind时，能够同时挂上gdb来进行调试。

<!-- more -->

Valgrind启动参数如下：

```bash
valgrind -v --vgdb=full --vgdb-error=0 --leak-check=full --track-fds=yes --show-reachable=yes --trace-children=yes --tool=memcheck --num-callers=64 --log-file=valgrind.log --track-origins=yes path/to/bin
```

使用如上指令启动Valgrind之后，程序不会立即运行，这是因为valgrind在等待gdb的连接。

gdb启动参数：
```bash
gdb path/to/bin
```

随后在gdb中输入：
```bash
target remote | vgdb
```

这样gdb就连接上了valgrind。在gdb中输入 `c` 并回车，valgrind即开始运行。
