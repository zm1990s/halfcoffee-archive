---
layout: post
title:  "hello jekyll!"
date:   2015-02-10 15:14:54
categories: jekyll
tags: jekyll
excerpt: jekyll 的 Hello World！
typora-root-url: ../../halfcoffee
---

* content
{:toc}
> Controller 在 NSX 中是比较重要的组件，在创建LSW之前必须先建立好Controller集群，建议至少、且最多部署三个Controller，分别运行在不同的主机上。

**1、失去所有Controller之后，最直观的就是无法创建或者修改LSW。**

在VCP教程中，介绍Controller 的作用是存放VTEP、ARP、以及MAC table，如果主机上的虚拟机要访问其他虚拟机，虚拟机的ARP请求会先被主机截获，主机会通过netcpa进程去访问负责某VNI的Controller来获取VM要访问VM的真实MAC，以及VM所在主机的VTEP等信息，这样可以减少ARP广播的泛洪。

而实际上Controller在这里的作用只是优化流量，如果controller完全失效，VM的ARP查询包会由每个ESXI主机的VTEP自行去处理，保证ARP得到正确的请求。具体内容见VCP单播、组播、混合模式这三种流量转发模式。



---

在介绍DLR Control VM的时候，CVM是控制层面的一个虚拟机，CVM和Edge Service Gateway进行动态路由通信，在建立完邻居并创建路由表后，会将路由信息通过Controller Cluster发送给主机，所以每个主机会有DLR CVM上的路由条目。

**2、如果Controller失效，DLR上收到的路由更新将不能通过controller发送给每台主机，主机依然按照上次收到的转发表去进行路由，如果环境中无任何变化，业务不会受影响。**

**但是，如果DLR uplink有两个不同的Edge service gateway，其中一个Edge1失效，而ESXi主机并不知道Edge1已经失效，继续给Edge1 转发数据，这时候报文将无法发送到数据中心之外。**

---


在创建或者修改 DLR 的时候，有个步骤是给每台ESXi主机上的DLR kernel module添加LIF，此操作也是通过Controller与netcpa进程通信来完成的，因此：

**3、Controller完全失效后，在DLR上创建LIF的操作将会失败**