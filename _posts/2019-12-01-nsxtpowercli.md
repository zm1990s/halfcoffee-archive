---
layout: post
title:  "nsx-t powercli script to batch create firewall rules"
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
2. Action must be **ALLOW** or **DROP** or **REJECT**, case sensitive
3. The script **ONLY** support **one source、one destination、one service per rule**
4. Tested on **Powershell Core 6.2.3 、PowerCLI 11.5.0**
5. The script would not create **sourceipset/dstipset/service** if it's name is used (which should mean they already exist!)
6. But for firewall rules, the script will create them each and every time you run! be careful of the duplicated rules. 

## csv example

```csv
rulename,srcipsetname,srcips,dstipsetname,dstips,servicename,protocol,port,action
APP1,APP1-192.168.1.1_10,192.168.1.1-192.168.1.10,any,,TCP-8080,TCP,8080,ALLOW
APP2,any,,APP2-172.16.0.0/24,172.16.0.0/24,HTTP,TCP,80,ALLOW
Deny any,any,,any,,any,,,DROP
```

![WX20191201-171309@2x](../pics/WX20191201-171309@2x.png)



## The code

```powershell
#Author: zm1990s@gmail.com

#import firewall csv
$table=Import-Csv ./fw.csv
[array]::Reverse($table)

#the section you want your rules in 
$fwsectionname = "Default Layer3 Section" 


foreach ($rule in $table)
{

#1. firewall source IPset, if not exist, create it 

$fwruleipsetssvc = Get-NsxtService -Name com.vmware.nsx.ip_sets
if ($fwruleipsetssvc.list().results | where display_name -eq $rule.srcipsetname) {
    Write-Host "Source IPset already created !" -ForegroundColor Yellow
    $fwipseccreated=$fwruleipsetssvc.list().results | where display_name -eq $rule.srcipsetname 
    $srcsetsinfo =""| select target_id,target_type   
    $srcsetsinfo.target_id=$fwipseccreated.id
    $srcsetsinfo.target_type=$fwipseccreated.resource_type
    Write-Host "Source IPset ID is" 
    $srcsetsinfo.target_id
   
}
elseif ($rule.srcipsetname -eq "any") {
    Write-Host "souce is any, ignoring" -ForegroundColor Yellow
}
else {
    $fwruleipsets=$fwruleipsetssvc.Help.create.ip_set.create()

    #1.1. ipset rule name
    $fwruleipsets.display_name=@()
    $fwruleipsets.display_name=$rule.srcipsetname

    #1.2. ipset addresses
    $fwruleipsets.ip_addresses=@()
    $fwruleipsets.ip_addresses+=$rule.srcips
    #for debug: display $fwruleipsets
    #$fwruleipsets

    #1.3 creating ipset
    $fwruleipsetssvc.create($fwruleipsets)

    #1.4 filter created ipsers, get id and type
 
    $fwipseccreated=$fwruleipsetssvc.list().results | where display_name -eq $rule.srcipsetname 
    $srcsetsinfo =""| select target_id,target_type   
    $srcsetsinfo.target_id=$fwipseccreated.id
    $srcsetsinfo.target_type=$fwipseccreated.resource_type
    Write-Host "Source IPset ID is" 
    $srcsetsinfo.target_id
   }

#2. firewall destination IPset, if not exist, create it 

$fwruleipsetssvc = Get-NsxtService -Name com.vmware.nsx.ip_sets
if ($fwruleipsetssvc.list().results | where display_name -eq $rule.dstipsetname) {
    Write-Host "Destination IPset already created !" -ForegroundColor Yellow
    $fwipseccreated=$fwruleipsetssvc.list().results | where display_name -eq $rule.dstipsetname 
    $dstsetsinfo =""| select target_id,target_type   
    $dstsetsinfo.target_id=$fwipseccreated.id
    $dstsetsinfo.target_type=$fwipseccreated.resource_type
    Write-Host "Destination IPset ID is" 
    $dstsetsinfo.target_id
   
}
elseif ($rule.dstipsetname -eq "any") {
    Write-Host "destination is any, ignoring" -ForegroundColor Yellow
}
else {
    $fwruleipsets=$fwruleipsetssvc.Help.create.ip_set.create()

    #1.1. ipset rule name
    $fwruleipsets.display_name=@()
    $fwruleipsets.display_name=$rule.dstipsetname

    #1.2. ipset addresses
    $fwruleipsets.ip_addresses=@()
    $fwruleipsets.ip_addresses+=$rule.dstips
    #for debug: display $fwruleipsets
    $fwruleipsets

    #1.3 creating ipset
    $fwruleipsetssvc.create($fwruleipsets)

    #1.4 filter created ipsers, get id and type
    #$fwipseccreated=$fwruleipsetssvc.list().results | where display_name -eq $fwruleipsets.display_name | select id,resource_type
    $fwipseccreated=$fwruleipsetssvc.list().results | where display_name -eq $rule.dstipsetname 
    $dstsetsinfo =""| select target_id,target_type   
    $dstsetsinfo.target_id=$fwipseccreated.id
    $dstsetsinfo.target_type=$fwipseccreated.resource_type
    Write-Host "Destination IPset ID is" 
    $dstsetsinfo.target_id
  }


#3. creating service

$servicesvc = Get-NsxtService -Name com.vmware.nsx.ns_services

#3.1. Check if service exists
if ( $servicesvc.list().results | where display_name -eq $rule.servicename )
{
    Write-Host "Service already created !" -ForegroundColor Yellow
  
    $fwservicecreated= $servicesvc.list().results | where display_name -eq $rule.servicename 

    $serviceinfo="" | select target_id,target_display_name,target_type
    $serviceinfo.target_id=$fwservicecreated.id
    $serviceinfo.target_display_name=$fwservicecreated.display_name
    $serviceinfo.target_type=$fwservicecreated.resource_type
    Write-Host "Service ID is" 
    $serviceinfo.target_id

}
#3.2. Check if service is any
elseif ( $rule.servicename -eq "any") {
    Write-Host "service is any, ignoring" -ForegroundColor Yellow
} 
#3.3. Check if service does no exists, create it
else 
{   
    $servicesvc = Get-NsxtService -Name com.vmware.nsx.ns_services
    Write-Host "Creating Service!" -ForegroundColor Green
    $servicespec = $servicesvc.Help.create.ns_service.Create()

    $servicespec.display_name = $rule.servicename
    $servicedetailspec = $servicesvc.Help.create.ns_service.nsservice_element.l4_port_set_NS_service.Create()
    $servicedetailspec.destination_ports = New-Object System.Collections.Generic.List[string]
    $servicedetailspec.destination_ports.add($rule.port)
    $servicedetailspec.l4_protocol = $rule.protocol
    $servicedetailspec.resource_type = "L4PortSetNSService"
    $servicespec.nsservice_element = $servicedetailspec
    #$servicespec
    $servicesvc.create($servicespec)

    $fwservicecreated= $servicesvc.list().results | where display_name -eq $rule.servicename 

    $serviceinfo="" | select target_id,target_display_name,target_type
    $serviceinfo.target_id=$fwservicecreated.id
    $serviceinfo.target_display_name=$fwservicecreated.display_name
    $serviceinfo.target_type=$fwservicecreated.resource_type
    Write-Host "Service ID is" 
    $serviceinfo.target_id
}


#4. building firewall rule
$fwrulesvc = Get-NsxtService -Name com.vmware.nsx.firewall.sections.rules
$fwrulespec = $fwrulesvc.Help.create.firewall_rule.Create()

#4.1. getting firewall rule name
$fwrulespec.display_name = $rule.rulename

#4.2. setting firewall rule source
if ($rule.srcipsetname -eq "any") 
{   Write-Host "sources is any, resetting varible!" -ForegroundColor Yellow
    $fwrulespec.sources = @()
}
else{
    $fwrulespec.sources=@($srcsetsinfo)
}

#4.3. setting firewall rule destination
if ($rule.dstipsetname -eq "any") 
{   Write-Host "destination is any, resetting varible!" -ForegroundColor Yellow
    $fwrulespec.destinations = @()
}
else {
    $fwrulespec.destinations=@($dstsetsinfo)
}


#4.4. setting firewall service
if ($rule.servicename -eq "any") 
{
    Write-Host "service is any, resetting varible!" -ForegroundColor Yellow
    $fwrulespec.services = @()
} 
else{$fwrulespec.services = @($serviceinfo)}

#4.5. setting firewall action
$fwrulespec.action = $rule.action

#4.6. turn on logging if you want
#$fwrulespec.logged = $true


#4.7. get section id and current section revision
$fwsectsvc = Get-NsxtService -Name com.vmware.nsx.firewall.sections
$fwsections = $fwsectsvc.list()
$fwsection = $fwsections.results | Where-Object {$_.display_name -eq $fwsectionname}
$fwrulespec.revision = $fwsection.revision

#4.8. display current settings
Write-Host "settings for current rules !" -ForegroundColor Green
$fwrulespec

#4.9. create the firewall rule
$fwrule = $fwrulesvc.create($fwsection.id, $fwrulespec)

#4.10. give it a time to fly
sleep 2

}
```



## the result

![WX20191201-171218@2x](../pics/WX20191201-171218@2x.png)



Reference:

[https://www.vmbaggum.nl/2019/03/automate-nsx-t-with-powercli/](https://www.vmbaggum.nl/2019/03/automate-nsx-t-with-powercli/)