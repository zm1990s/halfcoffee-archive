---
layout: post
title:  "OSPF MA网络在不同情况下收敛速度"
date:   2017-01-07
categories: OSPF
tags: ospf 收敛 速度 优化
---
* content
{:toc}
> 致我的网络老师：跟你学的都还给你了。。

> 之所以写这篇文章，是最近做NSX相关设计，测试结果表面NSX在某些情况下收敛速度不容乐观(1分钟以上)，所以就回头又来看看OSPF的设计，做做实验分析原因。



## OSPF 的 Hello、Dead 时间

ospf 有多种网络类型，此处只谈比较常用的MA网络，广播类型。

默认OSPF hello 时间为10s，Dead 时间为40s(4倍的hello时间)。

Hello 时间表示多久发一次Hello包，在邻居建立完成后，每个Hello时间会给邻居发送Hello包来表示自己还是存活的。

Dead 时间表示多久内未收到邻居的Hello包，就将邻居标记为Down，这样ospf的网络会重新收敛，以前从邻居收到的路由信息将会不可用。



## 一个实验











