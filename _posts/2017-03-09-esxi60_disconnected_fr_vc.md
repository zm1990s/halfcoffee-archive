---
layout: post
title:  "ESXi 6.0 从 vCenter断开且不能登录管理解决办法"
date:   2017-03-09
categories: vSphere
tags: vsphere vmware 6.0 VSAN esxcli 排错 troubleshooting
---

* content
{:toc}

>现象：在生产环境使用vSphere6.0(u1b及u2)后，出现过至少**4次 ESXi 主机从vCenter断开(vCenter上显示主机未响应)**，这时 ESXi 主机上的**虚拟机运行正常**，但是无法通过web client以及vSphere client登录主机进行管理，ESXi 的 esxcli 和 ssh 均能正常运行。在使用传统EMC存储以及VSAN时均遇到过此问题。



关于此问题 VMware 有两个KB，[KB2146548](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2146548) 和 [kb2146187](http://kb.vmware.com/kb/2146187) ，两个错误都是由于一个文件锁导致的，且官方的说法都是没有永久的解决办法，只能停止相关进程或者重启主机。

笔者在前几次碰到这个问题时是在ESXi 版本6.0 u1b 3380124。因为项目完成时6.0u2还未出来，后来主机已经业务化没有更新到6.0u2。为了及时恢复业务没有给vmware开case处理，直接对主机进行了断电操作，业务虚拟机发生HA恢复，然后等主机重启时升级到6.0u2，完成后系统一直正常运行。

后来VSAN环境的一台6.0u2主机也出现此问题，当时也以为没有办法去处理，贸然的重启了主机，结果在重启时卡在了VSAN初始化达20小时以上，数据安全性受到了影响（当时是FTT=1）。

一直到昨天，最后一台6.0u1b的主机出现相同的故障，刚好在现场，确定业务暂未受影响，为VMware开了case，抓了日志，VMware售后提出的解决方案是重启 hostd 和 vpxa 进程，而此操作对我这边环境并没有太大帮助，于是自己摸索着去看 vmkernel.log 和 hostd.log 的日志，发现在通过vSphere client登陆 ESXi 主机时， hostd.log 中有以下日志：

	2017-03-08T05:55:24.316Z info hostd[32740B70][Originator@6876 sub=Vimsvc.ha-eventmgr opID=CF4B779B-00000003-a74a] Event 65 : User root@192.168.200.18 logged in as VMware vSphere Client/6.0.0

	2017-03-08T05:55:54.449Z warning hostd[32B8CB70][Originator@6876 sub=Hostsvc opID=abb9a514] Hostctl error in GetLatencyThreshold(): Error interacting with configuration file /etc/vmware/lunTimestamps.log: Timout while waiting for lock, /etc/vmware/lunTimestamps.log.LOCK, to be released. Another process has kept this file locked for more than 30 seconds. The process currently holding the lock is localcli(533723026).  This is likely a temporary condition.  Please try your operation again.

通过关键词 /etc/vmware/lunTimestamps.log.LOCK 搜索到了前面提到的两个 KB ，我的情况属于 localcli 锁住了 /etc/vmware/lunTimestamps.log 文件，KB 中提到如果主机已经未响应，则需要重启后再禁用crontab定时任务，在我目前的环境下重启会影响业务，所以最好能通过其他方式解决。

后尝试了下**删除 /etc/vmware/lunTimestamps.log.LOCK 文件 (或使用mv命令重命名)**，之后esxi立刻连接到主机，单独登陆ESXi也可以正常登陆，登陆后发现有VM DRS，集群和主机也无单独报警，虚拟机运行正常。问题解决








