---
layout: post
title:  "一次 VSAN 误操作后的恢复纪实"
date:   2018-3-3
categories: VSAN
tags: VSAN esxcli unicastagent storage recovery
typora-root-url: ../../halfcoffee
---

* content
{:toc}

##摘要

> 从VSAN 6.6 开始，vSphere支持在部署VCSA时创建VSAN，然后将需要部署的VCSA存放在VSAN存储上。在此之前如果要部署VSAN，必须先找个服务器本地硬盘装好VCSA，然后再创建VSAN，最后再将vCenter迁移回VSAN环境。
>
> 按照VMware最佳实践，管理集群和计算集群需要分开，但很多小型环境受资金限制仍然会将所有vSphere管理组件和业务虚拟机一起放到VSAN运行，这样如果存储出现问题，管理虚拟机也会故障，未来不方便收集日志或者修复(这里要重点提的是：**VSAN目前没有因为软件问题导致数据丢失的案例，大家听过的VSAN故障均为误操作或者硬件兼容性问题**，所以大家大可放心去(正常)使用)。
>
> 今天要说的就是一个因为误操作导致VSAN存储不可用，而所有管理虚拟机均不可访问的恢复过程。这个恢复过程可以帮助展示VSAN的一些工作原理，让大家更熟悉VSAN，避免在重要环境发生误操作。生产环境还是建议走正规渠道开CASE，不要自己琢磨怎么修复，以免误操作数据丢失。


## 事件起因

VSAN环境由3台ESXi组成，每台服务器双万兆给VSAN用（vDS），双VSAN给业务虚拟机用（vDS），管理网络使用千兆（vSS），vCenter 运行在VSAN上。**因为误操作，导致 VSAN vDS 的 Uplink 没有关联任何物理接口，导致VSAN网络故障，进而所有虚拟机处于无法访问状态，但可以登陆每台ESXi的 web UI。**

故障现象如下图：

虚拟机状态为Invalid，无法进行任何的管理

<img src="/pics/vsanrestore1.png" width="800">

VSAN 存储的容量有问题，只有单个ESXi主机的容量（正常应该为36GB）

<img src="/pics/vsanrestore2.png" width="800">

## 修复思路

在检查环境后，确定是主机之间VSAN网络通信故障，登陆任意一台ESXi主机均不能ping通其他主机的 VSAN 地址：

<img src="/pics/vsanrestore3.png" width="700">

使用`esxcli vsan cluster get`命令看到每台ESXi主机上VSAN Cluster 成员只有自己，没有其他两台机器

<img src="/pics/vsanrestore4.png" width="500">

<img src="/pics/vsanrestore5.png" width="500">

登陆ESXi，发现已经无法管理VSAN VDS，例如给其添加Uplink操作：

<img src="/pics/vsanrestore6.png" width="800">

可以看到每台主机有VSAN的VMkernel，且类型标记为 vsan。

<img src="/pics/vsanrestore7.png" width="900">

从VSAN 6.6开始，所有VSAN之间的通信全部采用单播模式，单播相对于组播的优势是组网简单，但是缺点是**需要一个中心节点来让所有节点相互知晓，这个节点就是vCenter**，简而言之，在创建VSAN 6.6时，vCenter必须处于可用状态。但是，vCenter只是**自动**地让VSAN节点之间相互知晓，我们**可以手动为VSAN每个节点指定其他邻居**。VSAN 6.6 可以通过命令 `esxcli vsan cluster unicastagent list`来查看此ESXi主机的VSAN邻居有哪些。下图中vsan-2的邻居有两个，分别为 10.0.0.1和10.0.0.3，且都启用了Unicast，NodeUUID也可以看到。<img src="/pics/vsanrestore8.png" width="900">对比vsan-1的cluster信息，可以看到此处的NodeUUID就是cluster get命令获取到的Local UUID。

<img src="/pics/vsanrestore9.png" width="500">



检查邻居表正常，修复过程就变为：

1、删除原来的VSAN VMkernel

2、新建一个标准虚拟交换机vSS，为其关联合适的Uplink

3、创建新的VSAN VMkernel，将其关联到新建的vSS，将其服务设定为VSAN

##修复过程

1、原来VSAN使用vmk1通信，但这个VMkernel已经无法管理，需要删除重建（或者保留这个配置，临时新建一套VSAN网络）

<img src="/pics/vsanrestore7.png" width="900">

<img src="/pics/vsanrestore101.png" width="900">

<img src="/pics/vsanrestore102.png" width="400">

2、创建新的标准交换机

<img src="/pics/vsanrestore10.png" width="900">

<img src="/pics/vsanrestore11.png" width="600">

在其他主机上执行同样的操作

3、添加VMkernel，为其设置IP地址。在Services栏，可以看到不能将VMkernel标记为VSAN，需要使用命令行手动将VMkernel标记为VSAN。

<img src="/pics/vsanrestore12.png" width="500">

新建的VMkernel Services为空的。

<img src="/pics/vsanrestore14.png" width="900">

执行命令`esxcli vsan network ip add -i vmk1 -T vsan`将VMkernel 1标记为VSAN服务。

<img src="/pics/vsanrestore15.png" width="500">

配置完成后刷新web UI，看到vmk1 services已经变为VSAN。

<img src="/pics/vsanrestore16.png" width="900">

在其他主机上执行同样的操作。配置完毕后，三个新的VMkernel应该可以相互通信。

<img src="/pics/vsanrestore13.png" width="400">

使用命令`esxcli vsan cluster get`查看VSAN集群状态，所有节点均在集群中

<img src="/pics/vsanrestore17.png" width="900">

返回Web UI，查看VSAN存储容量变为正常

<img src="/pics/vsanrestore18.png" width="900">

虚拟机状态也为正常，可以进行管理。

<img src="/pics/vsanrestore19.png" width="900">

修复完毕！