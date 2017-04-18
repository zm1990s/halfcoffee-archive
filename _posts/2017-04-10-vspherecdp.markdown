---
layout: post
title:  "vSphere 下交换机报00:00:00:00:00:00全零MAC Flapping"
date:   2017-04-10 15:14:54
categories: vSphere
tags: vSphere Cisco CDP 邻居发现协议 mac flapping
typora-root-url: ../../halfcoffee
---

* content
{:toc}
> 摘要：在客户现场遇到一个奇怪的问题，华为交换机频繁地报告 MAC 地址抖动，通过交换机日志发现一个全0的MAC地址在两个端口上抖动，而这两个端口分别接同一个集群的两台vSphere主机，网卡为标准虚拟机交换机的uplink。

## 问题分析

全为 0 的 MAC 在正常的理解中是不合规的 MAC 地址，但是在交换机上查看mac地址表确实有此mac。但是关联在VLAN 1 下。

``` dis mac-address table
<Huawei>dis mac address-table
        Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
 1    0000.0000.0000    dynamic      G1/1/0/47
```



登陆 vCenter 查看vSwitch的配置，确定所有的端口组都有 VLAN 标记，因此确定这个全 0 的 mac 地址是由vSphere发出的，猜测或许是CDP协议，但是不能确定。

为了确认此地址的来源，在交换机上做了端口镜像，用 Wireshark 抓包分析，过滤全 0 的 mac，结果如下图：

![cdp](/pics/cdp.png)

## 解决办法

确定是 vSphere 发出了mac 地址为全 0 的CDP包，那么解决办法就是关闭cdp。

通过esxcli 命令可以查看或者修改cdp的工作状态。

查看 vSwitch1 的cdp状态，结果显示为both，表示发送并接收cdp包：

``` 
#esxcfg-vswitch -b vSwitch1
both 
```


将cdp模式修改为只接收或者关闭cdp：

` esxcfg-vswitch -B listen vSwitch1`

`esxcfg-vswitch -B down vSwitch1` 



**但是**，实际测试时发现cdp状态会自动从listen变为both，没办法只能去关闭cdp进程：

` /etc/init.d/cdp stop` 停止cdp进程

`chkconfig --del cdp`  停止开机启动