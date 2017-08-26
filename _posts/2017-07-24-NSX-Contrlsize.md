---
layout: post
title:  "NSX:如何调整Contoller内存大小"
date:   2017-07-24
categories: NSX
tags: NSX Controller size
typora-root-url: ../../halfcoffee
---

* content
{:toc}
> 摘要：NSX Controller默认使用4G内存，在NSX Controller部署完成后默认不允许修改Controller大小，可以通过修改NSX Controller vmx配置文件来修改Controller的CPU核内存大小，也可以从源头下手，直接修改NSX manager中Controller部署文件的配置。建议只在测试环境中这样操作

1、正常部署NSX manager

2、将NSX manager虚拟机关机，挂载任意Linux live cd

3、挂载NSX主目录，进入下图中的目录，找到nsx-controller-4.0.6-build500802.vxlan.ovf文件

4、使用sudo chmod +w 命令为.vxlan.ovf文件赋予写操作权限

<img src="/pics/nsx-cs1.png" width="800">

<img src="/pics/nsx-cs2.png" width="800">

5、使用vi文本编辑器打开ovf文件，找到内存所在行，修改VirtualQuantity中的4096，将其改为2048（最小需要2GB内存）

<img src="/pics/nsx-cs3.png" width="800">

<img src="/pics/nsx-cs4.png" width="800">

6、修改完成后存盘，重启让NSX manager正常开机，部署NSX Controller，则发现部署出来的Controller内存变为2GB

<img src="/pics/nsx-cs5.png" width="800">