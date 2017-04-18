---
layout: post
title:  "Veeam 更改备份任务的存储路径"
date:   2016-11-5 15:14:54
categories: veeam
tags: veeam 备份 存储空间
typora-root-url: ../../halfcoffee
---

* content
{:toc}
Veeam 备份是比较灵活的一个备份软件，每个备份文件和备份任务信息都保存在相同的路径，可以方便将任务导入其他backup server，重装veeam也不会影响到之前的备份。

前段时间遇到veeam一个backup repository几乎快满，而里面都是虚拟机的增量文件，按照计划任务还未进行增量文件的合并，如果手动合并会因为存储空间满而任务失败。剩下两种解决办法：**增加空间容量或者迁移**。我选择了后者。

操作方法：

1、找到对应虚拟机的备份文件夹，文件夹中应该至少包含一个.vbm文件和一个.vkb文件，以及.vib增量文件。

<img src="/pics/veeam1.png" width="300">

2、将此目录完整复制到目的存储

3、在veeam的 Backup Infrastructure > Backup Repositories 中添加（如果存储库已存在，直接点击rescan）存储库，然后右键选择Rescan repository。

<img src="/pics/veeam2.png" width="400">

4、更改虚拟机备份任务，在Storage栏中，直接选择新的存储库，点击确定即可。

<img src="/pics/veeam3.png" width="600">

 5、测试运行备份任务，虚拟机增量文件已经备份到新存储。删除原存储的备份文件。

