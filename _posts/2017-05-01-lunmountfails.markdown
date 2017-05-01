---
layout: post
title:  "Esxi LUN状态为非活动(已卸载)，手动挂载失败解决办法"
date:   2017-05-01 15:14:54
categories: vSphere
tags: vSphere ESXi 排错 troubleshooting 已卸载
typora-root-url: ../../halfcoffee
---

* content
{:toc}

<img src="/pics/lunmount.jpg" width="500">

> 摘要：一套vSphere环境断电，物理硬件全部恢复后发现vCenter没有自动开机，排查后发现所有主机的上某个LUN都处于非活动（已卸载）状态。

## 问题现象

登陆所有ESXi，在数据存储中可以看到此LUN为非活动状态，但是查看此LUN的路径全部是**活动**的（同存储上其他LUN在主机上均可以正常挂载使用，登陆存储未看到异常报警），重新扫描所有无法解决此问题，通过vSphere Client 选中此LUN执行**挂载**操作会报未知错误。

## 解决方法

按照**[此KB2004605](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2004605)**，可以通过esxcli命令行去查看存储卷相关的详细信息。

`esxcli storage filesystem list`

<img src="/pics/lunmount2.png" width="800">

可以看到vmware02这个卷未挂载，类型未知。

使用下列命令手动执行挂载，报告有Lock，详细看VMkernel日志

```
~# esxcli storage filesystem mount -u57889c95-a7569072-ec6e-0025b500001a

Sysinfoerror on operation returned status : Lock was not free. Please seethe VMkernel log for detailed error information
```

按照提示查看日志，发现下列行

```
~ # tail -100 /var/log/vmkernel.log 
2017-04-24T02:19:07.224Z cpu2:35389 opID=d5f911f6)WARNING: LVM: 13127: The volume on the device naa.60a980004431577850244463372d6653:1 locked, possibly because some remote host encountered an error during a volume operation and could not recover.
2017-04-24T02:19:07.224Z cpu2:35389 opID=d5f911f6)WARNING: LVM: 4938: If you are _sure_ this is the case, please break the device lock with `vmkfstools -B /vmfs/devices/disks/naa.60a980004431577850244463372d6653:1`
2017-04-24T02:19:07.224Z cpu2:35389 opID=d5f911f6)LVM: 11786: Failed to open device naa.60a980004431577850244463372d6653:1 : Lock was not free
```

按照日志中的提示，输入下列行来解锁。

```
~ # vmkfstools -B /vmfs/devices/disks/naa.60a980004431577850244463372d6653:1
```
```
VMware ESX Question:
LVM lock on the device naa.60a980004431577850244463372d6653:1 will be forcibly broken. See the vmkfstools or ESX documentation for information on breaking the LVM lock.

Ensure that multiple servers are not accessing this device.

Continue to break lock?
0) _Yes
1) _No

Select a number from 0-1: 0

Successfully broke LVM device lock for /vmfs/devices/disks/naa.60a980004431577850244463372d6653:1
```

再次通过命令行挂载`~# esxcli storage filesystem mount -u57889c95-a7569072-ec6e-0025b500001a`，执行成功没有报错，通过vSphere Client也可以看到主机正常挂载LUN，问题解决。