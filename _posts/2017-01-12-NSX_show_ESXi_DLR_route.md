---
layout: post
title:  "ESXi命令行查看NSX DLR路由"
date:   2017-01-12
categories: NSX
tags: vsphere vmware 6.2.2 NSX esxcli net-vdr 排错
---

* content
{:toc}


> 摘要：在 NSX 路由排错时，可能需要在ESXi主机上分析DLR路由学习情况，此时必须通过ssh连接到主机用命令查看

**操作步骤：**

1、SSH 连接到 ESXi 主机

2、运行 `net-vdr --instance -l` 命令获取 DLR 的名称

```
[root@ESXi1:~] net-vdr --instance -l
VDR Instance Information :
Vdr Name:                   default+edge-cd4d8d68-9444-421d-9260-513106a2123d
Vdr Id:                     0x00001392
Number of Lifs:             3
Number of Routes:           5
State:                      Enabled Universal Local-Egress
Controller IP:              10.10.50.20
Control Plane IP:           10.10.50.11
Control Plane Active:       Yes
Num unique nexthops:        2
Generation Number:          0
Edge Active:                No
```

3、运行`net-vdr --route  *DLR名称* -l` 查看详细路由信息

```
[root@ESXi1:~] net-vdr --route  default+edge-cd4d8d68-9444-421d-9260-513106a2123d -l

VDR default+edge-cd4d8d68-9444-421d-9260-513106a2123d Route Table
Legend: [U: Up], [G: Gateway], [C: Connected], [I: Interface]
Legend: [H: Host], F: Soft Flush [E: ECMP]
Destination      GenMask          Gateway          Flags    Ref Origin   UpTime     Interface
-----------      -------          -------          -----    --- ------   ------     ---------
0.0.0.0          0.0.0.0          172.16.1.2       UG       1   AUTO     8692       139200000002
1.0.0.0          255.0.0.0        172.16.1.3       UG       1   AUTO     406        139200000002
172.16.1.0       255.255.255.248  0.0.0.0          UCI      1   MANUAL   11251      139200000002
192.168.10.0     255.255.255.0    0.0.0.0          UCI      1   MANUAL   11715      13920000000a
192.168.11.0     255.255.255.0    0.0.0.0          UCI      1   MANUAL   11699      13920000000b

```



