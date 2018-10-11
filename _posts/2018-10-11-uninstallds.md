---
layout: post
title:  "如何卸载趋势无代理防病毒"
date:   2018-10-11
categories: NSX
tags: NSX 6.3 uninstall remove 卸载 deep security
typora-root-url: ../../halfcoffee
---



* content
{:toc}

> 在[前面的文章中](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247484315&idx=1&sn=cb68b7ed3e768a1e389a89bd2a3afc76&chksm=f98273b5cef5faa33db875e113e727ebd365f6c0dcbe62f9ec59db30db507c75cdecc6722e2b&scene=21#wechat_redirect)讲述了怎么才能顺利部署完趋势无代理防病毒，但最近也遇到过一些案例，无代理防病毒部署失败，未查出原因然后直接删除了相关虚拟机，结果是卸载不干净导致再次部署失败。因此，有了本文，一部分内容演示如何正确卸载无代理防病毒，另一部分演示如何修复异常卸载的残局。

卸载，一般是安装的反操作，因此需要先熟悉一下完成的部署流程，这次通过流程图来展示。

下图表示了部署无代理防病毒时一些重要的步骤以及关联关系，其中影响最大的是组件之间的注册，因为一旦注册后会执行很多自动化配置，**如果不按照步骤卸载，这些配置在未来必须手动删除**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHliaFz7QoTTGvWKKx8ribcyoaOM6iatDEUYDylSnjoIff0RAlZhGJJwI9Q/640?wx_fmt=png)

# **正常的卸载流程**

如果要从头至尾彻底地卸载，需要经历大约 7 个步骤：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHoCibGVranApk295Q35Y4IFvoibdicgeCqlDWNTiawibA5GcV0GGyQvUpTug/640?wx_fmt=png)

下面演示如何进行每一步的操作：

**1、取消激活计算机**



打开 DSM，进入“计算机”，选中已被管理的虚拟机，右键选择“停用”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHZjusfTS4G4wRJzsxq1AOpFoL5ucIzFv4aed8OicCX9ZI55ZUxibYIl9A/640?wx_fmt=png)

**2、删除 NSX 安全策略**



打开 vCenter，进入“网络与安全>服务编排>安全策略”，移除之前创建的安全策略

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHcC1DUULJjj2o4EtYZDNauyyKLX5dTqOe8rFGFr7TX2FSXclBLDiaIqw/640?wx_fmt=png)

**3、删除 NSX 安全组（可选）**



打开 vCenter，进入“网络与安全>服务编排>安全组”，移除之前创建的安全组

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHDkOugp0FZ1PX9iaBD30EbicHKWiajQYb0B3zMPnjVDgV09876Ph7ALiaqw/640?wx_fmt=png)

**4、卸载 Deep Security SVM**



打开 vCenter，进入“网络与安全>安装和升级>服务部署”，选中之前部署的“Trend Micro Deep Security”，点击删除按钮，等待卸载完成

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfH4Wwy77I0lNdYAWO4LP4SicV5q0p17WlNfJrrLnBBW7Rh8yiceu6icUYPA/640?wx_fmt=png)

**5、卸载 Guest Introspection**

打开 vCenter，进入“网络与安全>安装和升级>服务部署”，选中之前部署的“Guest Introspection”，点击删除按钮，等待卸载完成

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHuPrQDnxazz94MNias1xJOJss1mnJ13zlNzgmeibswzfxst8Qn2eNGibkQ/640?wx_fmt=png)

**6、DSM 中移除 vCenter Server**

打开 DSM，进入“计算机”，选中左侧 Computers 下的 vCenter，右键选择“Remove VMware vCenter Server”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfH1XiaGuBsVIrDbwaT4ky1SmN3SLZXhILETgZgsYYqKDQagCP3ibvicDQEw/?wx_fmt=png)

如果要完整删除，确保选择第一个选项“从 DSM 移除 VMware vCenter和所有从属计算机/组”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHy78t2p1Sb8bh16s0rQsH3FLFSicqDTlrDNc15aSQFmpsMUQ0dzYYjBA/640?wx_fmt=png)

删除完成后，需要登录 vCenter 检查 Deep Security 已经被卸载

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHguTyvx2UGibNmWlhrzCT0ltmLv44vMgiadK8cY21jiaYcpJj8vXRcA26Q/640?wx_fmt=png)

**7、卸载 NSX（可选）**

如果安装了 NSX 完整的功能（逻辑网络、分布式防火墙等），未来会有专门的文章描述如何卸载。

如果仅部署了 NSX 无代理防病毒相关的组件，在执行完上述操作后，可以直接将 NSX manager 关机，然后在 vCenter 中删除 NSX manager 相关插件。具体操作步骤如下：

登录如下地址 ： https://vcenter-FQDN/mob，登录

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHwvUoyXiadBBpcCHzWPY8P7wyibcwau2BXKaC7q9oEV0asslJf0WKLEaw/640?wx_fmt=png)

点击“Content”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHgNes5GCFjqVmN70JfuyT1UqQYaHiciaX4OOdOayNAM5Xb26g8IkSJ0AA/640?wx_fmt=png)

在 Content 中找到 ExtensionManager，点击

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHNqQFtTWzXeqah0sV0aSBj8WGkq5mfQzV4sV0CpwAVTLfIZT2V1pNnw/640?wx_fmt=png)

在 ExtensionList 中，确保可以看到 com.vmware.vShieldManager

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHiaibw1vsTQ4shKy3gdGBQHcQSiamgxMJia6qcGA31MASZa3PKuxC76MxWA/640?wx_fmt=png)

点击下方的 UnregisterExtension

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHhM1RBVeab2yZKgdicRX1hynicOwx9LPlDwPpXCe6aVGJicpGaGnDsmw1A/640?wx_fmt=png)

在 VALUE 栏中输入 “com.vmware.vShieldManager”，点击 Invoke Method

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHr6nViakZJkadwllXGYhnwViboovQ1xPG2CRRUtemwiaWHVNvLlORvFk3A/640?wx_fmt=png)

如果返回的结果为 void，代表命令被正常执行

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHxRDrHkpicaJBTKERC7mANHVEdxORS4sDyI5HvUgchy6pLONky6B8Axw/640?wx_fmt=png)

重新登陆 vCenter，可以看到主页中已经没有“网络与安全”的图标

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHjLxvADeR56JuaBOicEgiavRFu0fEibrdTCVWeEF2hicA4EtoH3BmUWlnYw/640?wx_fmt=png)

------

# **异常卸载后怎么办？**

****

## **情况1，单方删除了 DSM 等组件**

****

如正文一开始说的，非正常卸载需要手动执行一些操作，大致步骤如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHrHYXAicpiciciaqDAk4VOxMplmnFPqOkoD89WwT10ic3n6QjQ21Vpe1V52Q/640?wx_fmt=png)

步骤1~4参考前面的内容

**5、禁用“服务定义”中的 Deep Security Service Manager**

打开 vCenter，进入“服务定义>服务管理器”，选中 Deep Security Service Manager，点击编辑按钮

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHZSM6VAV4M7KP3sjep53kgovGKHaBSXiaAepg7szVOAUZYEVaKvKoQPg/640?wx_fmt=png)

取消勾选“Operational State”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHVIcH700fUMroqeF7UbRGia9TJcOiaJzeXeib495SefoEcUYib4Mib8233fg/640?wx_fmt=png)

**6、删除“服务定义”中的 Deep Security**

打开 vCenter，进入“服务定义>服务”，选中 Trend Micro Deep Security，点击编辑按钮

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHu8znMMQm8PZOSJCI0wRXKr0jxBcMVHgl5KCPef3oVKllWDFa26me9Q/640?wx_fmt=png)

点击左侧的 Service Instances>Trend Micro Deep Security，点击右侧 Related Objects，选中 Default（EBT），点击删除按钮

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHvk4zzyZgg22HuTGZMS4dtYQQKcr1jrH0zBiaxaHy5ibAckeYoEsib1IQQ/640?wx_fmt=png)

点击左侧的 Service Instances，点击右侧 Related Objects，选中 Trend Micro Deep Security-Globalinstance，点击删除按钮

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHeATgicU2icsbHqTUDrp4I5icwgShiaicCRwQUUuGOK6MQSywtUo99W4NpXA/640?wx_fmt=png)

点击左侧的 Trend Micro Deep Security，点击右侧窗口的删除按钮

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHN0l5jiazGWvLtgrSXiabxqwxyWn075Xhr1iafcsFJpbQThpdXVuUaJEpg/640?wx_fmt=png)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfH6XVA5UQI992HictfftYcSxQ4ico48SuASZHDdxFgiabmc6gF6hdXwICwg/640?wx_fmt=png)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHguTyvx2UGibNmWlhrzCT0ltmLv44vMgiadK8cY21jiaYcpJj8vXRcA26Q/640?wx_fmt=png)



**按照以上步骤卸载完成后，便可以重新安装 DSM，进行 vCenter 与 NSX 的注册等等。**



------

## **情况2，单方删除了 NSX Manager**

****

如果单方删除了 NSX Manager，问题会比较复杂，因为通过 NSX Manager 部署的 Guest Introspection 会在主机上有很多配置，这些配置必须在 NSX Manager 可用的情况下才能自动删除，这些配置包括虚拟交换机的配置、主机上的 EPSec vib 及配置等。

有种简便的办法，就是将 Guest Introspection 手动删除，然后让主机进入维护模式，然后移出集群，这样关于集群的相关配置都会自动被删除。

1、将虚拟机关机或迁移到其他主机

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHNhFyLjQhMgTiavBHkT7f1NibmRcQHg1ST83vxEO4nS3sIsxs4bcas49Q/640?wx_fmt=png)

2、删除 Guest Introspection 虚拟机

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHItSfUSOHxxIz1GLEt4IJEapvPJR5DsTFwrNauicxAOkB1ma8LpQtv9g/640?wx_fmt=png)

3、将主机进入维护模式

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfH8D15AgOicFibVwGINzvavxYNQKaL80DfpMQzJN6Gp5KftDMAh7Q0m2zg/640?wx_fmt=png)

4、将主机移出集群

系统会自动进行 agent 卸载

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfH4Q4Qibpib2icjr1xmFY3KImKiavGJObFs3RtnDWaZp4nVudLWTACZZZ7nw/640?wx_fmt=png)

5、手动删除 vmservice-vswitch

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHrDwLw6dDMznZAIReMcbObiaWFBVj9mxtG1doia74ypbLIEYnNujtjttg/640?wx_fmt=png)

**在 DSM 端，也需要手动移除 NSX Manager 和 vCenter：**

打开 DSM，进入“计算机”，选中左侧 Computers 下的 vCenter，右键选择“Properties”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHBASwOh2rHboEqsyRyI2q0dRKiaMibbZTBTkquQeMicHDLJPYJwSkL6l8A/640?wx_fmt=png)

选择 NSX Manager，点击“Remove NSX Manager”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHNiaMXqtwu4vRPPqcm7E6O2FppmqIzfnDSIgyq2vc9etrMLiazFgcba0A/640?wx_fmt=png)

然后再移除 VMware vCenter Server

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHK3eb44AzvPHUFqjX9zAFtibG9opCQylyJfJv0uaZIFQXCicR0TbiaqjpQ/640?wx_fmt=png)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHy78t2p1Sb8bh16s0rQsH3FLFSicqDTlrDNc15aSQFmpsMUQ0dzYYjBA/640?wx_fmt=png)

------

# **其他一些相关问题**

（卸载重装并不能从根本上解决问题，有时候就是因为误配置导致安装/测试失败）：

## **1、部署 Guest Introspection/Deep Security SVM 异常怎么办？**

这种问题可能由多种原因造成，常见解决办法如下：

- 检查 GI 和 DS SVM 地址池配置是否正常，可用地址是否足够。

对于 NSX 6.3 以前的版本，地址池位置在：“vCenter>网络与安全>左下角 NSX Manager>选中合适的 NSX manager>Manage>Grouping Objects>IP Pools”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfH0t7GicicWhOUg88GLNZsearibA2dLzFKgNrr5QNMrEEEuR646H5ZL2hgw/640?wx_fmt=png)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHwSiciaSicG6EsRpXNoJPSeVOhMxBULxVK1rOy4sMCvmHvHyIcSFiaaiaFrA/640?wx_fmt=png)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHAazUXs2ic9k4iciaLpnia1Z3x6fp9SmOAz6FMswOSzSOVibELsN0vDSIe8w/640?wx_fmt=png)

对于 NSX 6.4以后的环境，地址池位置在：“vCenter>网络与安全>Groups and Tags>IP Pools”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHW7DXR9CPX9xLN7l1oTicj47ibV1TI0hSa1kOvHccf4icfdL7wNtkaL8Ag/640?wx_fmt=png)

- 检查两种虚拟机所在的磁盘、网络端口组是否正确

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHgIUicnCGkmFZWd7J5qfHkj4xQckbT3Ys1hibxXVJYJ65qAGYtqBDSAkg/640?wx_fmt=png)

- 重启大法

  一般如果系统检测到运行异常，可以使用“Resolve”按钮重启虚拟机，或者手动重启虚拟机



![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHBmH2IOuGxgrKzTgGOKal8KriciaauFNm5Ak7Ny4ZibN4PAL7Ats32DkJQ/640?wx_fmt=png)

- 重装大法

在 NSX 服务部署页面删除掉 Guest Introspection 和趋势 SVM，再重新安装

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHUFK7y7bT6EJhfJKWOyeaCUeeGhEicWMmC4JrVPFib6lOSwkQNR6ibGicxg/640?wx_fmt=png)

## **2、执行防病毒测试失败怎么办？**

笔者在实验环境中，唯一遇到的问题就是 DSM 未配置 Relay Server（或 Relay Server 未正常工作/未连接到互联网），一旦修正此配置，测试立马通过。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePZwq940pn9EIicWFMpbqXfHz0PwrWxoPO0cVkAZvSoOFE7mUPciczhdCmMUzvDJCnpWlbpLpGZakcw/640?wx_fmt=png)



------

# 本文参考资料：

https://help.deepsecurity.trendmicro.com/10/0/zh-cn/Get-Started/Install/ig-uninstall-nsx.html

https://help.deepsecurity.trendmicro.com/10/0/Get-Started/Install/ig-uninstall-nsx.html

https://docs.vmware.com/en/VMware-NSX-for-vSphere/6.4/com.vmware.nsx.install.doc/GUID-33D4FCB4-A8C8-4307-B638-AA6BBA0AD2A5.html