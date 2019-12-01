---
layout: post
title:  "[Series:2/4]nsx-t powercli script to batch create logical switches"
date:   2019-12-01
categories: NSX
tags: vmware powercli nsx-t nsxt powershell
typora-root-url: ../../halfcoffee
---



* content
{:toc}
## Notice

There are few things you should know before use:

1. Don‘t change table names 
4. Tested on **Powershell Core 6.2.3 、PowerCLI 11.5.0**

## csv example

filename: **lsw.csv**

```
LS_Name
LS-01
LS-02
LS-03
```




## The code

```powershell

$logSwitchSvc = Get-NsxtService -Name com.vmware.nsx.logical_switches 
#获取逻辑交换机相关服务

$logSwitchSpec = $logSwitchSvc.Help.create.logical_switch.Create()
$logSwitchSpec.admin_state = "UP"
$logSwitchSpec.replication_mode = "MTEP"

$tZoneSvc = Get-NsxtService -Name com.vmware.nsx.transport_zones
#获取传输区域相关服务
$tZones = $tZoneSvc.list()
$logSwitchSpec.transport_zone_id = ($tZones.results | Where-Object transport_type -eq OVERLAY).id
#假设环境中只有一个 Overlay 的传输区域，可以用上述命令来自动获取并设置 Tranport Zone ID


$lswlist = Import-CSV ./lsw.csv 

Foreach ($lswname in $lswlist) 
{
$logSwitchSpec.display_name = $lswname.LS_Name

$logSwitchSpec
$logSwitchSvc.create($logSwitchSpec)
#创建逻辑交换机
}
```



## the result

![WX20191201-171218@2x](/pics/WX20191130-170108@2x.png)



Reference:

[https://www.vmbaggum.nl/2019/03/automate-nsx-t-with-powercli/](https://www.vmbaggum.nl/2019/03/automate-nsx-t-with-powercli/)