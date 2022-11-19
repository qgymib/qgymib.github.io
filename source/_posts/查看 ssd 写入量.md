---
title: "查看 SSD 写入量"
date: 2021-07-27T22:35:07+08:00
mathjax: true
---

## 背景

在Linux下查看SSD写入量并没有十分好用的工具，因此这里列出如何通过SMART信息计算SSD写入量。

<!-- more -->

## 查看SMART信息

使用smartctl查看SMART信息：

```bash
[qgymib@archlive ~]$ sudo smartctl -A /dev/sdc
[sudo] password for qgymib:
smartctl 7.2 2020-12-30 r5155 [x86_64-linux-5.11.16-arch1-1] (local build)
Copyright (C) 2002-20, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART Attributes Data Structure revision number: 1
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x0000   100   100   000    Old_age   Offline      -       0
  5 Reallocated_Sector_Ct   0x0000   100   100   000    Old_age   Offline      -       0
  9 Power_On_Hours          0x0000   100   100   000    Old_age   Offline      -       4
 12 Power_Cycle_Count       0x0000   100   100   000    Old_age   Offline      -       58
160 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       0
161 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       69
163 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       25
164 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       716
165 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       2
166 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       0
167 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       0
168 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       3000
169 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       100
175 Program_Fail_Count_Chip 0x0000   100   100   000    Old_age   Offline      -       0
176 Erase_Fail_Count_Chip   0x0000   100   100   000    Old_age   Offline      -       0
177 Wear_Leveling_Count     0x0000   100   100   050    Old_age   Offline      -       0
178 Used_Rsvd_Blk_Cnt_Chip  0x0000   100   100   000    Old_age   Offline      -       0
181 Program_Fail_Cnt_Total  0x0000   100   100   000    Old_age   Offline      -       0
182 Erase_Fail_Count_Total  0x0000   100   100   000    Old_age   Offline      -       0
192 Power-Off_Retract_Count 0x0000   100   100   000    Old_age   Offline      -       30
195 Hardware_ECC_Recovered  0x0000   100   100   000    Old_age   Offline      -       988
196 Reallocated_Event_Count 0x0000   100   100   016    Old_age   Offline      -       0
197 Current_Pending_Sector  0x0000   100   100   000    Old_age   Offline      -       0
198 Offline_Uncorrectable   0x0000   100   100   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x0000   100   100   050    Old_age   Offline      -       0
232 Available_Reservd_Space 0x0000   100   100   000    Old_age   Offline      -       100
241 Total_LBAs_Written      0x0000   100   100   000    Old_age   Offline      -       6913
242 Total_LBAs_Read         0x0000   100   100   000    Old_age   Offline      -       4380
245 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       11456
```



## 计算写入量

其中 `Total_LBAs_Written` 即为写入的 `LBA` 数量。`LBA` 大小与芯片型号有关，一般来讲，`1LBA` 可能等于如下几种值：`512B` / `32MB` / `1GB` 。为了确定 `1LBA` 的具体大小，可以通过写入特定大小的文件，并观察 `Total_LBAs_Written` 的变化来确定：

```bash
dd if=/dev/zero of=1G.file bs=1G count=1
```

此处我的SSD中`Total_LBAs_Written` 从 `6860` 增加至 `6892` ，从而计算可知 `1LBA` 大小为：

$$
\frac{1024MB}{6892-6860} = 32MB
$$
