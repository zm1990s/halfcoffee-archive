---
layout: post
title:  "OSPF 5种网络类型及适用场景"
date:   2017-03-15
categories: OSPF
tags: ospf cisco network type 网络类型
---

* content
{:toc}


> 摘要：在学OSPF的时候，大家都会讲到5种网络类型，以及哪种物理网络下使用哪种，但是很少提及在什么时候需要修改网络类型，如何根据实际环境优化配置

## OSPF 网络类型

OSPF网络类型有两种，Broadcast和NBMA(非广播多点访问)

- 广播，顾名思义，一个路由器发送的hello包其他同网段的多个路由器都可以收到，适用于一对多的类型。但是在某些情况下，即使三台设备的三个接口在同一个网段内，但是有些不支持接收组播报文的设备，需要使用NBMA。
- 对于默认是NBMA的FR、X.25、SMDS网络，也可以将其网络类型修改为broadcast，减少neighbor的配置


配置NBMA和broadcast都要求已经配置好虚电路（ virtual circuits）或者是全互联结构。但是不一定所有物理环境都这样，可能因为花费或者网络结构不是全互联，这时候需要将网络设置为point-to-multipoint

- point-to-multicast 相比 point-to-point 和NBMA有以下优点：
- 配置简单，不需要配置邻居，假设已经有IP 子网，不需要DR选举
- 花费更少，不需要网络结构为全互联
- 更加可靠，当虚电路故障时依然能维护连接不中断。




## 参考文章

1、 [OSPF network types](http://packetlife.net/blog/2008/jun/19/ospf-network-types/)

2、 [虚电路（Virtual Circuit)的概念](http://blog.csdn.net/bailyzheng/article/details/7785564)

