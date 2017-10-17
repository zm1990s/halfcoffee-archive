---
layout: post
title:  "使用 NSX+SRM 搭建灾备环境"
date:   2017-10-16
categories: NSX
tags: NSX VXLAN SRM DR 灾备
typora-root-url: ../../halfcoffee
---

* content
{:toc}
# 传统DR面临的挑战

> 当前的灾备系统依然比较复杂，因为灾备包含存储、计算和网络资源，当前的解决方案例如SRM可以完成计算以及存储的恢复，但是网络受限于物理架构，比较难通过SRM实现自动恢复。网络方面具体的问题如下

- 在备份站点和主站点建设等同的二层和三层网络架构
- 在两个数据中心使用一致的安全策略
- 在创建安全策略时依赖于IP地址
- 如果进行IP remaping，则依赖的策略都需要进行修改
- 如果不适用 IP re-mapping，则要保证两个数据中心二层网络打通，需要OTV、VPLS等技术。这样会扩大二层广播域。

# DR 场景

- 恢复部分应用
  - 恢复一个应用的部分虚拟机，恢复之后要保证其网络正常。
- 恢复整个应用
  - 完整恢复一套应用
- 整个站点恢复
  - 整个站点，包含 NSX 组件都需要恢复

# 资源需求

## ESXI 和 vCenter

为了确保灾难发生后两个数据中心都可管理，需要分离管理平面，即两个数据中心都有自己的vCenter管理本地的虚拟机。

## SRM

SRM依然可以沿用其最佳实践，在使用NSX后，SRM不在需要PortGroup的Mapping，只需要通过NSX创建全局逻辑网络。

## Storage Replication

在使用SRM时，Storage Replication是必须用到的，可以使用Array-Based Replication或者vSphere Replication。

如果使用 Array-Based Replication，则需要在SRM上安装SRA（Storage Replication Adapter），让SRM可以与存储阵列通信。

## NSX

两个数据中心分别安装自己的NSX manager，关联到本数据中心的vCenter，这时候需要使用Cross-VC NSX解决方案，使用 Universal 逻辑网络。

使用 Cross-VC NSX 时网络有以下特点：

- Cross-VC 可以将二层网络、三层网络以及安全延展到两个数据中心
- Cross-VC 支持 Universal 对象，如：Universal  Logical Switch、Universal Distributed Router、Universal DFW等。这些对象在 Primary NSX manager 被创建，可以同步到其他 NSX manager。
- Cross-VC 部署时有一个 Primary NSX manager，其他都是 Secondary NSX Manager。
- Cross-VC 解决方案只能有一个 Universal Controller Cluster，同时控制 Universal 网络和 Local NSX逻辑网络。
- Universal 逻辑网络只能在 Primary NSX manager上进行创建，但是在其他 NSX manager 上可以查看这些配置。
- Cross-VC NSX 支持 Universal DFW 规则，任何在 Universal Section 创建的防火墙规则都可以同步到所有 NSX manager。
- Universal DFW 只支持使用 IP Set、MAC Set、Security Groups（包含 IP Set和 MAC Set）来做防火墙策略
- Universal DLR 在进行 Egress 优化时，需要在两个数据中心分别部署一台 UDLR VM，和本地的 ESG 建立邻居关系。
- 如果 Primary NSX Manager 故障， NSX 网络可以被恢复，配置信息不会丢失



## NSX 6.2 及 Egress 流量优化

为了避免部分流量在数据中心间的传播，需要进行出流量优化，让本地数据中心产生的流量从本地的出口路由器出去，NSX 为了实现此功能引入了 Locale-ID 的概念，允许流量从关联到的 Locale-ID 的站点出。其特性主要功能如下：

- 默认，Locale-ID 被设置成 NSX Manager UUID，Locale ID 只有在启用 Local Egress 功能时才有效，平时无效。对于 DR 环境，必须做 Local  Egress。
- 启用 Local Egress 优化时，Locale-ID 可以基于主机、集群、DLR实例来进行配置。对于 DR 环境，我们通常针对整个集群进行 Locale-ID 配置。
- 对于每个 Universal DLR，需要部署两台虚拟机，一个在主站点一个在备份站点。每个 CVM 和其本地的 ESG 进行邻居关系建立。
- 配有相同 Locale-ID 的主机才能接收到对应 Locale-ID 的 CVM 的路由

# 预先配置

在开始搭建 DR 环境时，需要预先搭建好 NSX 环境，具体步骤如下：

- 两个独立的站点，之间通过三层网络连接起来
- 每个站点部署下列东西：一套或多套vCenter；ESXi 主机，请注意版本号；SRM，SRM需要安装最佳实践搭建
- 主数据中心搭建 Primary NSX Manager，和主数据中心的 vCenter 做对接。
- 其他数据中心搭建 Secondary NSX Manager
- 在主机数据中心部署三台 Universal Controller
- 主备数据中心使用相同的 Universal Transport Zone，使用相同的 Universal Logical Switch
- 每个站点有自己的 ESG 打通南北通信 - 这是为了做 Local Egress。
- 每个站点有自己的 DLR CVM 和本地的 ESG 做路由邻居。

# 细节配置

## 控制 Ingress 和 Egress 流量

有两种办法来控制流量

1、使用 Locale-ID 

> 此设计中，在主备数据中心均有自己的 UDLR，两个 UDLR 分别和自己本地的 ESG 建立邻居关系，这样的架构可以让两个数据中心的路由学习分开，进而控制本地流量。但是 Locale-ID 只能控制 Egress 流量，一般可以使用 deny prefix list 来控制路由学习，让备份数据中心的 ESG 不通过备份站点的 UDLR 学习到其关联的业务网络的路由。

- 需要创建两个 Transit Logical switch，两个分别连接本地的 ESG 和本地的 UDLR。
- 每个站点的为 UDLR 创建一个 Control VM，UDLR 与本地的 ESG 建立邻居关系，一旦灾难发生，备份站点的 UDLR CVM 已经和 ESG 建立邻居关系可以正常使用
- Egress 流量：开启 Local Egress，将主备数据中心所有主机的 Locale-ID 设置为主数据中心的 NSX manager UUID
- Ingress 流量：使用路由前缀列表（prefix list），将业务网络只通过给主数据中心的 ESG，不通告给备数据中心的 ESG

<img src="/pics/NSX-DR1.png" width="800">

2、使用动态路由调整流量路径

>  在此设计中，UDLR 存在于两个数据中心，但是只部署一个 UDLR Control VM。只给 UDLR 创建一个 Uplink，和两个站点的 ESG 做邻居。这时候所有主机都能从两个 ESG 收到路由，为了优化 Egress，可以修改动态路由的 Cost。对与 OSPF，Cost 越小路由越优，对于 BGP，Weight 越大路由越优。本例中需要修改 ESG 到 UDLR (BGP:以及外部路由器连接 ESG) 的 Cost 值。

- 只创建一个 Universal Logical Switch 来作为 UDLR 的 Uplink，同时连接两个数据中心的 ESG
- 只在主数据中心部署一个 UDLR CVM。在发生灾难后，CVM 需要重新在备份站点进行部署，重新建立路由关系
- 不需要使用 Local Egress
- 通过动态路由 Cost 调整选路
- Egress 流量：Controller 只会将最佳路由宣告给ESXi 主机，所以此例中，所以流量会从主数据中心出去
- Ingress 流量：调整动态路由 Cost 后，虽然两个数据中心的物理路由器都能收到去往 NSX 逻辑网络的路由，但是只会优先从主数据中心的 ESG 进去，因为其路径最优。

<img src="/pics/NSX-DR2.png" width="800">

**3、综合 1 和 2 的设计，两个 UDLR，开启 Local Egress 。**

Egress：使用 Locale-ID 控制从主数据中心出去。

Ingress ：调整 ESG （使用 BGP 时还需要调整物理路由器 Weight）的 Cost，让所有流量优先从主数据中心进入。

## 计划迁移/部分应用恢复

>  大部分企业要求 DR 设计能够满足部分应用恢复或者计划迁移的要求。在这时候，做完恢复后要求恢复的虚拟机不仅能够正常运行，他们还需要与运行在主数据中心的部分虚拟机进行正常通讯。

以下是部分恢复时 SRM 配置步骤：

- 执行部分 failover plan，在此案例中，只恢复 web 和 APP VM，不恢复 DB VM

<img src="/pics/NSX-DR3.png" width="800">

- 对于 NSX 而言，不需要做任何变更，虚拟机可以随意在两个数据中心进行迁移
- ​

# 参考资料

【Disaster Recovery with NSX and SRM】

