---
layout: post
title:  "nsx-t powercli script to batch create router ports"
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
ls_name,routeport_name,routeport_ip,routeport_prefix
LS-01,LS-01,192.168.101.1,24
LS-02,LS-02,192.168.102.1,24
LS-03,LS-03,192.168.103.1,24
```

![WX20191201-171309@2x](/pics/WX20191201-201932@2x.png)



## The code

```powershell
$table=Import-Csv ./lsw.csv

#1. 设置并获取 T1 路由器的 ID
$t1routename="k8s-cluster1-pear"
$t1routesvc = Get-NsxtService -Name com.vmware.nsx.logical_routers
$t1routeid=$t1routesvc.list().results | where display_name -eq "$t1routename"

foreach ($routeport in $table ){

#2. 获取逻辑交换机 ID 
$logSwitchSvc = Get-NsxtService -Name com.vmware.nsx.logical_switches 
$logSwitchid = $logSwitchSvc.list().results | where display_name -eq $routeport.ls_name

#3. 创建逻辑端口并关联逻辑交换机
$logicalportsvc = Get-NsxtService com.vmware.nsx.logical_ports
$logicalportspec= $logicalportsvc.help.create.logical_port.create()
$logicalportspec.logical_switch_id=$logSwitchid.id
$logicalportspec.display_name=$routeport.ls_name+"-l3port"
$logicalportspec.admin_state="UP"
$logicalportspec
$logicalportsvc.create($logicalportspec)

#3.1 获取创建出的逻辑端口的 ID
$logicalportid=$logicalportsvc.list().results | where display_name -eq $logicalportspec.display_name

#4. 在T1上创建三层接口
$routeportsvc = Get-NsxtService -Name com.vmware.nsx.logical_router_ports
$routeportspec = $routeportsvc.help.create.logical_router_port.logical_router_down_link_port.create()

#4.1 设置三层接口名称
$routeportspec.display_name=$routeport.routeport_name

#4.2 配置三层路由器接口参数
$routeportspec.linked_logical_switch_port_id=$routeportsvc.help.create.logical_router_port.logical_router_down_link_port.linked_logical_switch_port_id.Create()
$routeportspec.linked_logical_switch_port_id.target_id=$logicalportid.id
$routeportspec.linked_logical_switch_port_id.target_type="LogicalPort"

#4.3 配置ip及掩码参数
$ipinfo="" |select  ip_addresses,prefix_length
$ipinfo.ip_addresses=New-Object System.Collections.Generic.List[string]
$ipinfo.ip_addresses=@($routeport.routeport_ip)
$ipinfo.prefix_length=$routeport.routeport_prefix

$routeportspec.subnets = $routeportsvc.help.create.logical_router_port.logical_router_down_link_port.subnets.create()
$routeportspec.subnets=@($ipinfo)

#4.4 配置T1 ID参数
$routeportspec.logical_router_id=$t1routeid.id

#4.5 完成创建
#$routeportspec
$routeportsvc.create($routeportspec)
}
```



## the result

![WX20191201-171218@2x](/pics/WX20191201-202029@2x.png)



Reference:

[https://www.vmbaggum.nl/2019/03/automate-nsx-t-with-powercli/](https://www.vmbaggum.nl/2019/03/automate-nsx-t-with-powercli/)