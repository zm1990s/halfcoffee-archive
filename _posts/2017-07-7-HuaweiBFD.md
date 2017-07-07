---
layout: post
title:  "华为设备 BFD + 静态路由的几种配置方式"
date:   2017-07-07
categories: 网络
tags: 华为 huawei bfd 静态路由
typora-root-url: ../../halfcoffee
---

* content
{:toc}
> 摘要：网络环境中使用了静态路由以及二层接口卡，如果链路故障则无法和vlan接口进行up down的联动，因此也就不能实现静态路由故障切换。需要依赖于其他检测技术，例如BFD

## 什么是 BFD

BFD全称Bidirectional Forwarding Detection，双向转发检测，可以实现快速检测并监控网络中链路问题或者路由转发问题，与其他协议配合能实现自动或者半自动的切换（例如静态浮动路由切换），以保证网络高可用性。

从名称中可以看出，这个协议一般需要检测与被检测端建立会话（session），使用共同的参数进行链路故障检测，其参数有些类似于ospf的hello包，有最小发送时间（min-tx-time），最小接收时间（min-rx-time），检测倍数（Detect time multiplier）。检测时间=最小接收时间*检测倍数。在检测时间内未收到对端发来的hello，则bfd认为链路故障。

为防止bfd抖动，检测时间需要大于链路延迟。

## 静态路由为什么需要和BFD联动

以下图为例，一个普通网络中双路由器，运行静态路由协议，PC的网关指向VRRP虚拟地址。

<img src="/pics/bfd1.jpg" width="800">



这种网络结构，如果不加入检测机制，设备或者链路的故障都会造成网络通信问题。

<img src="/pics/bfd2.jpg" width="800">

<img src="/pics/bfd3.jpg" width="800">

## 解决方案

第一种解决办法就是依赖于VRRP主备切换，这种方式流量比较优化，缺点是影响比较大，如果两端路由器只接了彼此，那么vrrp怎么切换都不会有太大影响，如果两端路由器上有其他很多业务，则vrrp切换会对其他业务有影响。

<img src="/pics/bfd4.jpg" width="800">

第二种方式只是做路由的切换，可以将影响控制在指定网络，比较适合于较为复杂的网络环境。

<img src="/pics/bfd5.jpg" width="800">

## 0、VRRP 联动 BFD 实现切换

本端主路由器：

```
bfd
bfd test1 bind peer-ip 192.168.1.2 interface E0/0/0 one-arm-echo
 discriminator local 1
 min-echo-rx-interval 100
 commit

interface E0/0/0
 ip address 193.1.1.1 255.255.255.0
 vrrp vrid 1 virtual-ip 193.1.1.3
 vrrp vrid 1 priority 120
 vrrp vrid 1 preempt-mode 
 vrrp vrid 1 track bfd-session test1 reduced 40
```

本端备路由器：

```
interface E0/0/0
ip address 193.1.1.2 255.255.255.0
 vrrp vrid 1 virtual-ip 193.1.1.3
 vrrp vrid 1 priority 100
 vrrp vrid 1 preempt-mode 
```



## 1、单臂回声功能删除静态路由

单臂回声功能比较适合于对端不支持BFD的情况，在上面提到的网络环境中，使用单臂回声功能只能实现和静态路由的联动。

<img src="/pics/bfd6.jpg" width="300">

具体配置如下：

```
bfd
```

//全局开启bfd功能
```
bfd test1 bind peer-ip 192.168.1.2 interface E0/0/0 one-arm-echo
discriminator local 1
commit
```


//配置bfd session，test1为session名称，interface为本端出接口，单臂回声只支持直连检测。

//discriminator local 为本地的会话标识符，只要和本地其他 bfd session不冲突即可。

```
ip route-static 0.0.0.0 0.0.0.0 192.168.1.2 track bfd-session test1
```

//配置主静态路由，默认优先级为60，关联bfd检测，如果bfd报告故障，此路由会从路由表中删除

```
ip route-static 0.0.0.0 0.0.0.0 193.1.1.2 preference  200
```

//配置浮动静态路由，在主路由失效后生效



## 2、与对端设备BFD对接，关闭本端接口

这种方式可以同时针对物理三层口以及SVI口进行操作。根据文档，华为只能和华为设备对接，且只支持单跳。

> 实际上如果使用三层口对接，物理链路故障会让三层口状态变为down，实现静态路由切换，但如果使用MSTP等链路，即使和对端通信异常接口也是up状态，需要借助BFD提高检测准确性。	

<img src="/pics/bfd7.jpg" width="400">

具体配置如下：

```
bfd
```

//全局开启bfd功能

```
bfd test1 bind peer-ip default-ip interface E0/0/0
discriminator local 1
discriminator remote 2
process-interface-status
commit
```

//配置bfd session，test1为session名称。

//discriminator 需要填写本端以及对端的标识符。

//process-interface-status 表示当bfd检测到故障时，宣告E0/0/0处于down状态，这样与此接口关联的静态路由会自动从路由表中删除

```
ip route-static 0.0.0.0 0.0.0.0 192.168.1.2
```

//配置主静态路由，默认优先级为60，关联bfd检测，如果bfd报告接口故障，此路由会从路由表中删除

```
ip route-static 0.0.0.0 0.0.0.0 193.1.1.2 preference  200
```

//配置浮动静态路由，在主路由失效后生效

## 3、与对端设备BFD对接，删除静态路由

此功能支持多跳检测，不像单臂回声只支持直连的检测

具体配置如下：

```
bfd
``` 

//全局开启bfd功能

```
bfd test1 bind peer-ip 192.168.1.2 source-ip 192.168.1.1 auto
min-rx-interval 500
min-tx-interval 500
detect-multiplier 3
commit
```

//配置bfd session，test1为session名称。

//discriminator 由BFD自己去协商，不同厂商对接可能需要调整rx、tx时间和检测倍数。

```
ip route-static 0.0.0.0 0.0.0.0 192.168.1.2 track bfd-session test1
```

//配置主静态路由，默认优先级为60，关联bfd检测，如果bfd报告故障，此路由会从路由表中删除

```
ip route-static 0.0.0.0 0.0.0.0 193.1.1.2 preference  200
```

//配置浮动静态路由，在主路由失效后生效


## 参考文章

1、 RG-RSR20系列路由器配置手册(V2.0)

2、AR120&AR150&AR160&AR200&AR500&AR510&AR1200&AR2200&AR3200&AR3600 配置手册
