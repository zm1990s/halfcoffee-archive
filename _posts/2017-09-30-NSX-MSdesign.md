---
layout: post
title:  "NSX 多站点设计1：几种部署方案"
date:   2017-09-30
categories: NSX
tags: NSX VXLAN design 设计
typora-root-url: ../../halfcoffee
---

* content
{:toc}

# 为什么有多站点

- DR
- 扩建
- 公司合并
- 多个公司在不同的地方，为本地员工提供服务
- 新旧数据中心迁移
- 公司

# 传统的多站点解决方案面临的挑战

- 如果进行了业务迁移，需要更改业务 IP 地址
- 需要重新配置业务所需的L2-L3网络
- 安全策略需要在备份站点手动创建/同步，如果应用更改了IP地址，安全策略也需要更改IP
- 其他相关设备也要跟着做改变，例如LB
- ACL、DNS等也要随之更改

## 传统解决方案

**1、使用裸光纤打通二层网络**





<img src="/pics/multi1.png" width="800">

此方案的问题：

- 延展二层网络会增加二层故障域
- 二层环路会导致两个数据中心网络都受影响
- 广播风暴会影响到两个站点
- 敏捷性、扩展性不好，新的工作负载依然会受 VLAN 4096 的限制，且配置不方便
- 需要使用STP来防止环路，会带来额外的开销，增加网络复杂度
- STP 误配置、重新收敛会导致网络无法访问。
- 因为 STP 的阻塞机制，无法100%利用所有链路，带来链路浪费
- 如果使用其他协议替代 STP，一般这些协议都是厂商私有协议，依赖于硬件设备
- 操作维护困难
- 只能实现L2的扩展
- 每个设备都需要配置，缺少自动化，灵活性



**2、VPLS（Virtual Private LAN Service）**

<img src="/pics/multi2.png" width="800">

此方案的问题：

- 花费高
- VPLS 桥不需要 STP 来防止环路，但是每个站点都需要配置 STP，STP 误配置、收敛也会引起站点的网络中断
- 站点内使用 STP 带来链路浪费
- 站点内也会发生环路，造成网络中断
- 如果使用其他协议替代 STP，一般这些协议都是厂商私有协议，依赖于硬件设备
- 一个典型的 VPLS 部署中，所有运营商的PE（Provider Edge）设备需要学习所有客户网络的MAC地址，造成MAC地址泄露。为了避免此问题，可以使用PBB（Provider backbone bridging），使用MAC-in-MAC方案，但是这会增加网络复杂性。
- 敏捷性、扩展性不好，新的工作负载依然会受 VLAN 4096 的限制，且配置不方便
- 操作维护困难
- 只能实现L2的扩展
- 每个设备都需要配置，缺少自动化，灵活性



**3、基于硬件的 Overlay 网络（例如OTV）**

<img src="/pics/multi3.png" width="800">

此方案存在的问题：

- 花费高
- 基于硬件，一般需要购买额外的 License，也可能要求替换当前使用的设备。
- 协议私有、复杂
- 操作维护困难
- 只能实现L2的扩展
- 每个设备都需要配置，缺少自动化，灵活性



## **使用 NSX 建立双活数据中心**

NSX 提供一系列的逻辑网络、安全功能，可以像服务器虚拟化一样将网络功能从硬件设备中解耦和。NSX 的分布式交换、分布式路由、分布式防火墙等功能可以提供增强分布式网络功能，满足各种二层网络扩展和安全需求。

NSX 能实现以下功能：

- 跨站点逻辑网络一致性
- 跨站点的安全一致性
- 跨站点业务 IP 无需改变
- 可以运行在任意 IP 网络上
- 软件定义网络，不依赖于硬件。有效保护当前物理网络资产
- VXLAN 协议是个标准协议，不同的厂商能够提供第三方集成方案
- 可以和 VMware 其他 SDDC 产品结合，例如 vSphere，vRealize Suite （Automation、Operation Manager、LogInsight）

# NSX Multi-sites 配置模式

NSX 可以提供多种 Multi-site 配置方案，包括使用 L2VPN 进行二层网络扩展，通过单 NSX 扩展逻辑网络（包括双数据中心多集群或者双数据中心延展集群），跨 vCenter NSX 等。

- A-A 模式，使用单一延展集群
- A-A 模式，使用多个 vSphere 集群
- A-A 模式，Cross-VC 的 NSX
- A-A 模式，使用 L2VPN

## 1、A-A 模式，使用单一延展集群

此解决方案中，集群的主机分布在两个数据中心中，伴随着集群的延展，通常也需要存储延展到两个数据中心，常见的解决方案有 EMC 的 VPLEX，或者VMware vSAN。

延展集群不一定需要使用 NSX，但是使用 NSX 可以带来诸如逻辑网络、分布式防火墙、自动化等方便实用的功能。实现二层延展也不需要依赖硬件厂商的解决方案，对数据中心物理网络要求更低。

<img src="/pics/multi4.png" width="800">



此部署模式的特点有：

- 两个站点只能有一个vCenter，计算、Edge、管理集群需要延展到两个数据中心
- 需要使用 VPLEX 或者 VSAN 等存储解决方案实现跨数据中心共享存储访问。
- 在vSphere 6 中，vMotion 的延迟要求是 RTT<150ms。存储(VPLEX)对于数据中心网络延迟要求是 RTT<10ms
- 一般这种模式适用于同城、或者同建筑物，这是受存储同步复制的延迟要求限制。
- 本质上来说是一个 vSphere 集群，所以 vSphere 集群的功能例如 HA 、DRS 都可以使用。
- 这种解决方案可以提供容灾、恢复功能
- 所有管理组件运行在单个数据中心中，包括：vCenter、NSX manager、NSX Controller。可以通过 HA 在另外的数据中心恢复。不一定需要将管理二层网络在两个数据中心间打通，只需要保证故障恢复时备份站点能提供一样的网络服务。
- 站点间需要保证 MTU > =1600
- 这种部署模型下，多个Edge可以是不做egress流量优化的A-A结构，也可以使用 Universal 对象来配置Local Egress优化，但是使用Local Egress优化只支持静态路由。
- 通常也可以做成主数据中心传输数据，备份数据中心热备的A-S结构（上图中即为此结构），通常为了维护方便，避免异步路由，也建议使用这种结构。这种结构下，单个数据中心的单个Edge可以用来提供 LB、FW 等服务，开启Edge HA来保证其高可用性。



## 2、A-A 模式，使用多个 vSphere 集群

在此架构中，vSphere集群不会延展到两个站点，在每个站点都有一个集群，对数据中心之间延迟需求也只是vMotion的RTT<=150ms，NSX Control 平面对于网络的最低需求也是150ms。

<img src="/pics/multi5.png" width="800">



此部署模式的特点有：

- 两个站点公用一个vCenter，计算、Edge、管理集群在每个站点都存在。
- 每个站点有自己的本地存储，数据中心间的延迟要求是 vMotion 的 RTT<150ms。
- 因为两个站点的 vsphere 集群不跨越数据中心了，因此无法使用 vSphere 集群的功能，例如 HA 、DRS。但在站点内部依然能使用这些功能。
- 逻辑网络能够跨越这两个站点，因此 VM 的迁移以及微分段功能不需要物理网络进行 VLAN 打通。
- 站点间需要保证 MTU > =1600
- 上图中，每个站点都部署有一套 ESG，且主站点的两个 ESG 为活动，备份站点的 ESG 为不活动。这种活动/非活动的结构通过设置路由的管理距离来控制的。
- 这种部署模型下，多个 Edge 也可以做成egress流量优化的A-A结构，此时则需要使用 Universal 对象来配置Local Egress优化，但是使用Local Egress优化只支持静态路由。
- 通常为了维护方便，避免异步路由，建议使用活动/非活动(A/S)结构。



## 3、A-A 模式，Cross-VC 的 NSX

从 NSX 6.2 开始引入跨VC NSX以及 Local Egress 优化。

跨 VC 的 NSX 允许逻辑网络及安全跨越多个 vCenter 域或者数据中心，在此解决方案中 vSphere 集群不跨越数据中心。

对数据中心之间延迟需求也只是vMotion的RTT<=150ms。

<img src="/pics/multi6.jpg" width="800">

此部署模式的特点有：

- 两个站点分别有自己的 vCenter 和 NSX manager，计算、Edge、管理集群在每个站点都存在。
- 每个站点有自己的本地存储，数据中心间的延迟要求是 vMotion 的 RTT<150ms。
- 因为两个站点的 vsphere 集群不跨越数据中心了，因此无法使用 vSphere 集群的功能，例如 HA 、DRS。但在站点内部依然能使用这些功能。
- 为了实现在线迁移 （vMotion），需要使用增强 vMotion 功能，即需要两个 vCenter 使用同一个 PSC。
- 逻辑网络能够跨越这两个站点，因此 VM 的迁移以及微分段功能不需要物理网络进行 VLAN 打通。
- 站点间需要保证 MTU > =1600
- 上图中，每个站点都部署有一套 ESG，且主站点的两个 ESG 为活动，备份站点的 ESG 为不活动。这种活动/非活动的结构通过设置路由的管理距离来控制的。
- 这种部署模型下，多个 Edge 也可以做成egress流量优化的A-A结构，此时则需要使用 Universal 对象来配置Local Egress优化，但是使用Local Egress优化只支持静态路由。
- 通常为了维护方便，避免异步路由，建议使用活动/非活动(A/S)结构。

## 4、A-A 模式，使用 L2VPN

NSX Edge 提供 L2VPN 功能，可以很方便地将一个数据中心的二层网络延展到其他数据中心。通常 L2VPN 用于以下两个场合：

1、业务迁移

2、和公有云的集成

<img src="/pics/multi7.png" width="800">



在下图中，通过 NSX L2VPN 将两个站点的二层网络打通。Site 2 上运行传统的 VLAN ，其上运行的所有业务需要迁移到 Site 1，SIte 1中的 DLR 是两个数据中心业务网段的网关，通过 Edge 将 VXLAN 网络和 VLAN 网络打通。

<img src="/pics/multi8.png" width="800">



下图是 NSX L2VPN 支持的一些网络组合：

1、VLAN 到 VLAN 的桥接

2、VXLAN 到 VXLAN 的桥接

3、VLAN 到 VXLAN 的桥接

<img src="/pics/multi9.jpg" width="800">

Tunnel ID 需要同时在 L2VPN Server 和 Client 端进行配置。所有的数据通过 TLS 加密传输。

在上图中，使用 L2VPN 时需要注意以下事：

- NSX 只在站点 1 进行部署，站点 2 可以不部署 NSX，只部署 NSX L2VPN standalone client。但是 standalone client 只支持二层网络桥接，不支持作为 Edge 网关使用
- NSX L2VPN server 使用 HA 模式部署。如果 Client 端也部署了 NSX，L2VPN client 端也支持部署为 HA。
- 只需要在 NSX Server 端使用 NSX license，客户端不需要。
- 站点之间无延迟要求，但是高延迟会影响应用性能。
- 因为无延迟要求以及双活存储，所以理论上数据中心之间无距离限制
- 因为一般这种环境下管理都是分开的，迁移一般使用冷迁移
- 在上述部署场景中，DLR 作为两个数据中心的网关；网关也可以是 Edge 或者外部物理路由器
- 在上述部署场景中，两个站点的所有数据都通过站点1 的 ESG 出往数据中心。NSX L2VPN 也支持出流量优化，此时两个站点会有同样的网关 IP，使用 ARP 过滤网关 IP 来防止 MAC 地址冲突
- L2VPN 默认在 MTU=1500 时即可工作，最大支持到 9000

**NSX L2VPN 拓扑：**

<img src="/pics/multi10-1.jpg" width="500">

<img src="/pics/multi10-2.jpg" width="500">



# 各种配置模式比较

<img src="/pics/multi11.png" width="900">

# 参考资料

【Multi-site Options and Cross-VC NSX Design Guide】

