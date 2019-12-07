---
layout: post
title:  "[Series:4/4]nsx-t powercli script to get loadbalancer infos"
date:   2019-12-07
categories: NSX
tags: vmware powercli nsx-t nsxt powershell
typora-root-url: ../../halfcoffee
---



* content
{:toc}
There are some times you want to examine your configuration on virtual services or pools . But NSX-T's UI isn't so friendly , it can't even sort ! 



Well, now you have a option to get all the configuration around pools in your environment. Using powercli ! 



The script itself has some limitations:

- Currectlly can't  display monitor name (I'm using monitor's id instead), this is because I can't figure out how to use API..
- Can't  display persistence profile name either, I can't figure out how to use API..
- Snat's format is a bit mess, I don't know how to disassemble that variable.



The script was tested on **Powershell 5.1、PowerCLI 11.5.0**



## csv example

filename: **pools.csv**

```
name
Pool_test_1
Pool_test_2
Pool_test_3
```




## The code

```powershell
#in order to get the exact pool info as your design, we need a pool name list.
$table= Import-Csv .\pools.csv


# get nsxt services
$nsxlbmonitor =get-nsxtservice -name com.vmware.nsx.loadbalancer.monitors  
$nsxlbpools =get-nsxtservice -name com.vmware.nsx.loadbalancer.pools 
$nsxlbvs =get-nsxtservice -name com.vmware.nsx.loadbalancer.virtual_servers 

#get all virtual servers
$allvs= $nsxlbvs.list().results | select display_name,ip_address,port,ip_protocol,pool_id,persistence_profile_id

#setup a emepty array
$allinfo=@()

ForEach ($poolname in $table){
	$selectedpool=$nsxlbpools.list().results | where display_name -eq $poolname.name

	$info="" | select vs_name,vip,vs_port,poolname,members,port,alg,monitor,snat,persistence_profile_id
	$currentvs=$allvs | where pool_id -eq $selectedpool.id
	$info.vs_name=$currentvs.display_name
	$info.vip=$currentvs.ip_address
	$info.vs_port=$currentvs.port
	$info.poolname=$poolname.name
	$info.members=[string]$selectedpool.members.ip_address
	$info.port=[string]$selectedpool.members.port
	$info.alg=[string]$selectedpool.algorithm
	$info.monitor=[string]$selectedpool.active_monitor_ids
	$info.snat=[string]$selectedpool.snat_translation
	$info.persistence_profile_id=[string]$currentvs.persistence_profile_id
	#display current lb info 
	$info
	$allinfo+=$info

}

#display all info
$allinfo | ft -AutoSize 

#export to csv
$allinfo | Export-Csv  all_LB_info.csv

#export to html
$css  = "table{ Margin: 0px 0px 0px 4px; Border: 1px solid rgb(200, 200, 200); Font-Family: Tahoma; Font-Size: 8pt; Background-Color: rgb(252, 252, 252); }"
$css += "tr:hover td { Background-Color: #6495ED; Color: rgb(255, 255, 255);}"
$css += "tr:nth-child(even) { Background-Color: rgb(242, 242, 242); }"
Set-Content -Value $css -Path all_LB_info.css
$allinfo | ConvertTo-Html -CSSUri "all_LB_info.css" | Set-Content "all_LB_info.html"
```



## the result

HTML format:

![image-20191207210433565](/pics/image-20191207210433565.png)

