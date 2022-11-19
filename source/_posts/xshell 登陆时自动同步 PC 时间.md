---
title: "Xshell 登陆时自动同步 PC 时间"
date: 2021-07-27T22:55:59+08:00
---

对于部分没有 RTC 芯片的设备，重启会导致时间重置。如下 VBS 脚本可在xshell登陆时自动同步PC时间：

```vbs
Sub Main
    xsh.Screen.Synchronous = True
	xsh.Screen.Send "sudo TZ='Asia/Shanghai' date -s """ & FormatDateTime(Now(), vbGeneralDate) & """" & VbCr
End Sub
```

在Xshell的会话属性中，选择“连接->登陆脚本->连接会话时运行脚本”，选择上述脚本即可。
