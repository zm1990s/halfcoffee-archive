---
layout: post
title:  "Lenovo system X3850 X6 固件(M5210 RAID卡)升级及开启JBOD（直通）模式 "
date:   2016-07-13
categories: VSAN
tags:vsphere vmware esxcli X3850 systemX IBM Lenovo
typora-root-url: ../../halfcoffee
---
* content
{:toc}
> 写在前面：因为需要做VMware VSAN，要求服务器RAID卡最好支持直通模式。X3850 X6带的RAID卡是M5210，硬件说z第一次做IBM服务器的升级，升级原因是M5210 RAID卡虽然支持JBOD模式(硬盘pass-through)，但是开启对应选项时报错，国外有人用X3650服务器，M5210的RAID卡遇到了同样错误，最终通过升级RAID卡固件，并移除RAID卡缓存后实现了JBOD(厂家也告知要移除缓存才支持JBOD，但是我移除后RAID卡会进入safe-mode模式)。



本文所使用工具均在(固件包除外)：[http://pan.baidu.com/s/1dFfqpvb](http://pan.baidu.com/s/1dFfqpvb)

**正题：**按照400的说法，需要使用BoMC工具进行升级，可在IBM官网下载到：

[http://www-947.ibm.com/support/entry/portal/docdisplay?lndocid=TOOL-BOMC](http://www-947.ibm.com/support/entry/portal/docdisplay?lndocid=TOOL-BOMC)

或者使用400给的百度链接：bomc:[http://pan.baidu.com/s/1c2kHVqS](http://pan.baidu.com/s/1c2kHVqS)



BOMC全称为Bootable Media Creator。他是用来创建IBM服务器微码升级盘的工具。

可以利用他来创建升级微码的启动光盘、升级微码的启动USB盘、以及可启动的PXE文件。可以利用此工具，将所有的当前最新的微码进行下载。并在需要时使用。

注：

+ 此工具不能下载设备驱动程序(需要自行到IBM官网搜索，方法下面介绍)。
+ 此工具必须在有administrator权限的用户下运行。
+ 部分防火墙需要将此工具手动加到可访问程序列表中（软件需要下载制作bootios需要的工具，我的百度盘中已上传相关文件）。

**一、IBM 上搜索需要升级的固件**

[https://www-947.ibm.com/support/entry/myportal/fixcentral_redirect](https://www-947.ibm.com/support/entry/myportal/fixcentral_redirect)

点击**Downloads > Fixes，updates & drivers**

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img0.ph.126.net/rRQF-AvEYQbJHiT2mifd6Q==/6631758259001646336.png)

可以输入产品型号或者Machine Type(输入产品型号会让选择machine type 6241) ![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img2.ph.126.net/ObToly3Tclt98GGvIYlIJQ==/6631616422001659100.png)

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img2.ph.126.net/5BxT6VS8kNKKf-PWXOYyPA==/6631514167420275500.png)

点击Continue后，找到需要下载的固件，本次需要使用第四个固件，M5200 RAID卡固件。

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img1.ph.126.net/fTWOdT_14g0_NsaJmnfk6A==/6631623019071421747.png)

下载完成后，会有四个文件(下图lnvgy-fw_sraidmr_5200)，保存在一个目录中，同时将百度云盘中其他BoMC需要的文件如uxspi、boot-tools也放到此文件夹下。![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img0.ph.126.net/JdGmDPWB83kubE79Da7u8w==/6631830826769087541.png)

**二、使用BoMC制作升级启动镜像**

找到刚才下载的bomc软件，运行

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img1.ph.126.net/ls2B07NEkr1BYgyduIn5yA==/6631593332257479073.png)

此次用做更新服务器固件，选择updates，如果需要自动运行更新任务，可以勾选下面的Enable Task AutoRun；为节省资源可以创建只有CLI的界面。

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img2.ph.126.net/QLqNrz0iR03dQMgUlKxjNg==/6631673596606304795.png)

资源可以是从IBM网站下载，但是这个软件的服务器列表有限，通过这种方式下载不到X3850 X6（6241）的固件，所以我们手动去IBM网站提前下载好了更新包。

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img2.ph.126.net/ciwE1smJ5UDOnpEOmo3fig==/6631717577071413665.png)

选择第一章说的文件夹。

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img1.ph.126.net/zKoXDYduZFOrmENnyHXpCw==/6631769254117920412.png)

可以选择创建光盘 ISO （此ISO只能通过虚拟光驱或光盘使用，不可用于制作USB启动）或者制作U盘启动盘。

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img0.ph.126.net/a-pJBpNpouYFFshKKAZT8Q==/6631718676583041441.png)![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img0.ph.126.net/Eh25RsfzK5iqO-j_HsWLSA==/6631783547769083019.png)![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img2.ph.126.net/CvgQgOXAXqENSo12cWX9rg==/6631586735187712433.png)

创建完成。

**三、使用KVM为服务器加载光驱，并从光驱启动进行安装**

系统提示要不要开始，输入 Y 按回车。

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img2.ph.126.net/XCx-r2yASt2sQ3kg9z17dg==/6631576839583058249.png)

系统会检测系统已安装固件以及新固件版本，默认选择了所有可更新固件，点击Apply All selected开始升级。

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img2.ph.126.net/6t7vnc7ac5cH5qyTcAKlEA==/6631623019071421754.png)

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img2.ph.126.net/FUmVcpj4JeANpEoVX-F7aA==/6631817632629544181.jpg)

已经安装完成两个固件，点Quit退出。

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img2.ph.126.net/zyWaqHpc_G8PsF8d6i34EQ==/6631642810280726606.png)

再输入 Q 退出程序。系统会进行重引导

![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img0.ph.126.net/MyWAOc9xZpu8W6WbLNPp0g==/6631670298071424252.png)

**四、拔掉RAID卡上的缓存，开启JBOD( RAID卡降级为iMR模式 )**

拔掉后启动系统，RAID卡缓存丢失后会报错，提示输入 D 将RAID卡降级为 iMR 模式，根据M5210文档， iMR模式下才能用JBOD。
![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img0.ph.126.net/qbgKzvdyIYA02n90z5q1pA==/6631720875606296991.png)保存配置，进入RAID 卡配置，可以看到固件版本已经更新 ![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img2.ph.126.net/3m6EZSBH0PP4k_7qxH0xsQ==/6631738467792344318.png)在 Raid 卡 controller properties 中可以看到 JBOD 模式已经自动开启。![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img0.ph.126.net/DvfZtbq2hUq7Esx29iQsRg==/6631504271815628907.png)在 Drive management 中可以看到所有硬盘已经是 JBOD 模式。![Lenovo system X3850 X6 固件(RAID卡)升级及开启JBOD（直通）模式 - zm - Art - Life](http://img2.ph.126.net/wl4GlprZvcg-M3wZBVoZ1w==/6631511968397019938.png)
**五、写在后面**好了现在可以做VSAN了，需要注意的是，一定要更新 ESXi 的RAID卡驱动，去IBM下载好对应驱动，用此驱动替换VMware默认的 lsi-mr3 驱动。