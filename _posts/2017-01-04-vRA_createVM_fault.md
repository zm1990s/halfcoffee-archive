---
layout: post
title:  "vRealize Automation 7.0的一个奇怪BUG，虚拟机创建路径问题"
date:   2017-01-04
categories: vRealize
tags: vsphere vmware 7.0 automation vRealize
---
* content
{:toc}

> 摘要：客户环境中使用了VMware PSO团队的Unified Portal产品，Unified Portal是个借用vRealize Automation实现虚拟机的申请创建生命周期管理的云管平台。其自身有多部门管理，用户分级管理和审批流程等功能，UP底层工作流使用VCO（vRealize Orchestrator）

## 问题描述

原vSphere环境中，只有一个集群加入了vRealize Automation的计算资源中，但因为系统扩容，新建了一个集群，为了统一管理，在vRealize automation在<span style="color:blue;">*架构组*</span>下勾选了此新集群（Compute Resources）。

按照Unified Portal的工作流程，要求接下来创建 **预留策略** > **网络配置文件**  > **预留** ，我按照VMware的说法，创建了同名的预留策略和预留（假设名为 res-test ），因为当时网络环境（NSX未安装到新集群）尚未具备，但是在设置中可以选择NSX的端口组进行关联。创建完预留后vRA出现错误警告，然后预留无法再进行编辑（会提示错误），且删除也报错。

后来修复了NSX的问题，此策略依旧无法使用无法删除，无奈新建了一个策略暂时使用

然后问题来了，在旧集群上申请的虚拟机，无一例外都会部署到新的集群中，屡试不爽。



## 解决办法

后来仔细观察发现，新建的虚拟机都会自动关联到那个不能编辑的策略 res-test 中。

1. 在 **Automation>基础架构>受管计算机** 下找到所有属于 res-test 预留的虚拟机，并销毁
2. 修改预留策略 res-test 的名字为其他任意名字（res-test1）
3. 删除 res-test 预留
4. 删除 res-test1 预留策略



> 感觉好奇怪的bug....删除这个策略后vRA又恢复正常工作

