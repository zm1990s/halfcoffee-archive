---
layout: post
title:  "HowTO：如何配置 DHCP 中继"
date:   2017-10-16
categories: NSX
tags: NSX VXLAN DHCP RELAY
typora-root-url: ../../halfcoffee
---

* content
{:toc}
> 摘要：DHCP



## 配置步骤

- 在 NSX 中创建新网络（LSW）
- 在新创建的网络中启用 DHCP Relay agent
- 在 DHCP Server 中创建关于此网络的 DHCP 池



DHCP 全局配置

​		
​		
​	
​	
​		
​			
​				
​					

						1. Click Manage.
						2. Click DHCP.
						3. Click Relay.
						4. Click Edit.

​		
​		
​	
​	
​		
​			
​				
​			
​		
​	

IP Sets

IP Sets are con gured from the NSX Manager Global Con guration and allow us tospecify a subset of DHCP servers by creating a named grouping.