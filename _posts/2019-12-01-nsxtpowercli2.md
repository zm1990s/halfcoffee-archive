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
#get logical service
$logSwitchSvc = Get-NsxtService -Name com.vmware.nsx.logical_switches 


$logSwitchSpec = $logSwitchSvc.Help.create.logical_switch.Create()
$logSwitchSpec.admin_state = "UP"
$logSwitchSpec.replication_mode = "MTEP"

#get transportzone info
$tZoneSvc = Get-NsxtService -Name com.vmware.nsx.transport_zones
$tZones = $tZoneSvc.list()

#we assume there's is only one overlay transport zone in your environmnet.
#if not, use the following line instead
#$logSwitchSpec.transport_zone_id = ($tZones.results | Where-Object display_name -eq "TransportZoneName"
$logSwitchSpec.transport_zone_id = ($tZones.results | Where-Object transport_type -eq OVERLAY).id


$lswlist = Import-CSV ./lsw.csv 

Foreach ($lswname in $lswlist) 
{
#creating logical switch
$logSwitchSpec.display_name = $lswname.LS_Name
$logSwitchSpec
$logSwitchSvc.create($logSwitchSpec)

}
```



## the result

![WX20191201-171218@2x](/pics/WX20191130-170108@2x.png)



Reference:

[https://www.vmbaggum.nl/2019/03/automate-nsx-t-with-powercli/](https://www.vmbaggum.nl/2019/03/automate-nsx-t-with-powercli/)