---
layout: post
title:  "ESXi 下 intel SSD 固件及驱动升级"
date:   2016-08-29
categories: VSAN
tags: vsphere vmware esxcli intel SSD p3700
typora-root-url: ../../halfcoffee
---

* content
{:toc}




## **驱动升级**

VSAN 兼容列表内，Intel 的P3600和P3700有两款固件：8DV10171 和 8DV10131，两个推荐的驱动不一样，具体信息如下：

总结来说，固件为**8DV10171**的推荐VMware自己的 **nvme-1.2.0.27-4vmw** 驱动，我在网上找到最新版本为：

<span style="line-height: 28px; color: rgb(0, 0, 255);">nvme-1.2.0.27-4vmw.550.0.0.1331820.x86_64.vib

以及intel自己的intel-nvme-1.0e.2.0 驱动，我找到最新版本为：

<span style="line-height: 28px; color: rgb(0, 0, 255);"> intel-nvme-1.0e.2.0-1OEM.550.0.0.1391871.x86_64.vib

****

固件为**8DV10171**的推荐intel自己的 intel-nvme-1.0e.1.1 驱动。 

<img src="/pics/updateintelssd1.png" width="700">



 <img src="/pics/updateintelssd2.png" width="700">

如果使用Intel自己的驱动，建议安装intel的DC TOOL，具体安装方法见下文。

驱动安装方法：

1、主机进入维护模式

2、将驱动通过 scp 上传到 tmp 目录或者直接通过 vsphere client 放到某个 datastore 中，下文是使用scp传到了tmp目录。

通过命令安装：

`esxcli software  vib install -v /tmp/`

`intel-nvme-1.0e.2.0-1OEM.550.0.0.1391871.x86_64.vib`

安装完成后重启主机。使用**esxcfg-scsidevs -a **命令可以看到系统使用了intel-nvme驱动

<img src="/pics/updateintelssd3.png" width="800">

`esxcli system module set --enabled=false —module=nvme`

 

## **固件升级**

Intel 提供两种固件升级方案，1是下载[ FirmwareUpdateTool](https://downloadcenter.intel.com/zh-cn/download/18363/Intel-?product=35125)，这是个 OS independent ISO镜像。直接挂在服务器或者PC上，用这个光盘引导升级。但是因为某种原因，此次用的X3850 X6服务器不能引导这个镜像，因此需要用另一种升级方法，直接在操作系统上使用DataCenterTool升级。

下载地址：[https://downloadcenter.intel.com/zh-cn/download/26221/Intel-SSD-?product=35125](https://downloadcenter.intel.com/zh-cn/download/26221/Intel-SSD-?product=35125)

DataCenterTool 默认搭载支持SSD的最新固件，具体固件版本请参见发行说明。**必须先正确安装intel-nvme驱动才能使用DataCenterTool，否则此工具识别不到 intel SSD。**

将下载好的工具(.vib文件)上传到ESXi，使用命令(`esxcli software vib install -v 安装包路径` )安装：

<img src="/pics/updateintelssd4.png" width="700">

安装报错，提示系统不能安装社区签名的vib，需要更改系统的软件安装属性：

使用 `esxcli software acceptance set--level=CommunitySupported ` 修改

<img src="/pics/updateintelssd5.png" width="700">

 

因为没有环境变量，所以直接运行isdct会失败，需要指定完整路径运行。

使用 `/opt/intel/isdct/isdct show -intelssd ` 可以查看当前系统安装的intel SSD详细信息，其中有个Index需要注意。

使用` /opt/intel/isdct/isdct load -intelssd 0` 对硬盘固件进行升级 

<img src="/pics/updateintelssd6.png" width="800">





