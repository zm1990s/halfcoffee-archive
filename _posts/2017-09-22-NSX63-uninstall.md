---
layout: post
title:  "如何卸载 NSX"
date:   2017-09-22
categories: NSX
tags: NSX 6.3.3 uninstall remove 卸载
typora-root-url: ../../halfcoffee
---

* content
{:toc}
> 摘要：NSX 目前不支持直接降级操作，如果要降低NSX版本，只能通过卸载再重装 NSX 的方式。



# 一、截图备份所有配置

包括主机准备（IP地址池）、Controller cluster IP地址池、分段ID配置、LSW配置、DLR、ESG配置等。

 

# 二、卸载逻辑路由器及ESG

选中对应的Edge设备，点击图中删除按钮删除。删除前请确保所有虚拟机未使用NSX逻辑网络

<img src="/pics/nsx63un-1.jpeg" width="800">



<img src="/pics/nsx63un-2.jpeg" width="400"> 

<img src="/pics/nsx63un-3.jpeg" width="800">

 

# 三、删除逻辑交换机

检查所有虚拟机未使用LSW，如果有使用，请选择将其从LSW中移除

<img src="/pics/nsx63un-4.jpeg" width="800">

<img src="/pics/nsx63un-5.jpeg" width="800">

<img src="/pics/nsx63un-6.jpeg" width="800">

 

依次删除所有LSW

<img src="/pics/nsx63un-7.jpeg" width="800">



 

# 四、删除传输区域

首先将集群从传输区域中移除，一般当传输区域内剩余一个集群时，不允许将其从传输区域中删除，此时只需要强制删除该传输区域即可。

<img src="/pics/nsx63un-8.jpeg" width="800">



 

# 五、取消主机VXLAN配置

在逻辑网络准备>VXLAN传输中取消配置所有集群，此操作会删除所有主机的VTEPVMkernel配置，取消完成后请务必先检查主机上VTEP网卡已被删除，如未成功自动删除，请手动删除。

<img src="/pics/nsx63un-9.jpeg" width="800">

<img src="/pics/nsx63un-10.jpeg" width="800">

 

# 六、卸载NSX组件

在**主机准备**中选择卸载NSX

<img src="/pics/nsx63un-11.jpeg" width="800">



 

此时vCenter任务栏会提示正在卸载代理

<img src="/pics/nsx63un-12.jpeg" width="800">

 

当任务进行完后，选中主机查看其事件，发现错误，提示主机需要进入维护模式才能完成卸载。

依次将所有主机进入维护模式，完成NSX VIB卸载：

<img src="/pics/nsx63un-13.jpeg" width="800">

<img src="/pics/nsx63un-14.jpeg" width="500">

 

 

<img src="/pics/nsx63un-15.jpeg" width="800">

  

在卸载完成后退出维护模式

<img src="/pics/nsx63un-16.jpeg" width="700">

 

在事件中看到VCNS已经被成功卸载

<img src="/pics/nsx63un-17.jpeg" width="800">

如果NSX同VSAN一同安装了，请在主机退出维护模式后检查VSAN健康状况，所有VSAN主机组件完成同步后再对下台主机进行维护操作。

<img src="/pics/nsx63un-18.jpeg" width="800">

<img src="/pics/nsx63un-19.jpeg" width="800">

卸载完成后，在主机准备栏可以看到主机安装状态为空

<img src="/pics/nsx63un-20.jpeg" width="800">



依次将所有主机进入维护模式，等待所有主机卸载完成。

<img src="/pics/nsx63un-21.jpeg" width="800">

 

# 七、删除Controllers

删除Controller，前两个可以使用删除，最后一个Controller使用强制删除。

<img src="/pics/nsx63un-22.jpeg" width="800">

<img src="/pics/nsx63un-23.jpeg" width="800">



<img src="/pics/nsx63un-24.jpeg" width="800">

 

# 八、取消NSX注册

关闭NSXmanager电源

<img src="/pics/nsx63un-25.jpeg" width="800">

 

登陆vCentermob ，地址为 [https://vCenter-IP/mob](https://vCenter-IP/mob)

选择Content：

<img src="/pics/nsx63un-26.jpeg" width="800">



找到 ExtensionManager

<img src="/pics/nsx63un-27.jpeg" width="800">

选择UnregisterExtension：

<img src="/pics/nsx63un-28.jpeg" width="600">

输入“com.vmware.vShieldManager”，点击 Invoke Method 。

<img src="/pics/nsx63un-29.jpeg" width="500">

待结果返回void表示卸载完成。

<img src="/pics/nsx63un-30.jpeg" width="500">

重新登陆vCenter Web Client 可以看到NSXNetwork&Security图标已经消失。

<img src="/pics/nsx63un-31.jpeg" width="800">

 

至此NSX卸载完成。

## 参考资料

[NSX 6.3安装手册>卸载]([https://docs.vmware.com/en/VMware-NSX-for-vSphere/6.3/com.vmware.nsx.install.doc/GUID-33D4FCB4-A8C8-4307-B638-AA6BBA0AD2A5.html](https://docs.vmware.com/en/VMware-NSX-for-vSphere/6.3/com.vmware.nsx.install.doc/GUID-33D4FCB4-A8C8-4307-B638-AA6BBA0AD2A5.html))

