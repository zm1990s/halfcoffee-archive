---
layout: post
title:  "NSX Edge与物理交换机OSPF认证配置"
date:   2017-03-14
categories: NSX
tags: vsphere vmware NSX cisco auth ospf
typora-root-url: ../../halfcoffee
---

* content
{:toc}


> 摘要：用户为 OSPF 启用了区域认证，在以前只有交换机参与路由的时候，OSPF 邻居都能正常建立，但是加入 NSX Edge 之后，发现 NSX 和交换机的认证配置方法略微“不一样”，这篇文章会介绍下OSPFv2的认证方法并解决上述问题。

## OSPFv2 的认证模式

- 空，即不认证
- 明文，在OSPF报文中明文加入认证密钥，但是**以明文形式**发送，通过抓包软件能看到
- MD5，将整个OSPF报文 (包含我们设置的认证秘钥)进行hash校验(除了auth date这个字段)，校验值作为认证字段(auth data)写入报文中



对于 Cisco 的设备，认证的配置有**两种认证配置模式**：

- OSPF 区域认证，一般在ospf进程内进程内开启，在接口下配置认证密码
- OSPF 接口认证，在参与OSPF邻居的接口下开启，并在接口下配置认证密码





那综合认证模式和配置模式，思科设备就会有以下几种配置组合：

1、OSPF 进程内开启区域明文认证， 这样所有在此区域内的所有接口**需要**配置**明文认证密钥**，**如果不配置密钥，则使用空密钥**进行报文的校验。配置示例如下：

```
router ospf 110
 router-id 1.1.1.1
 area 0 authentication
R1(config)# interface f0/0
R1(config-if)# ip ospf authentication-key cisco
```

<img src="/pics/ospf-auth1.png" width="400">



2、OSPF 进程内开启区域密文认证， 这样所有在此区域内的所有接口**需要**配置**密文认证密钥**，**如果不配置密钥，则使用空密钥**进行报文的校验。

```
router ospf 110
 router-id 1.1.1.1
 area 0 authentication message-digest
R1(config)# interface f0/0
R1(config-if)# no ip ospf authentication-key
R1(config-if)# ip ospf message-digest-key 1 md5 cisco
```

<img src="/pics/ospf-auth2.png" width="400">

在这里可以看到key的id是从1开始的，0则是预留的空密钥。



3、在对应的 OSPF 接口下开启明文认证，并设置明文认证密钥。

```
router ospf 110
 router-id 1.1.1.1
 no area 0 authentication
R1(config)# interface f0/0
R1(config-if)# ip ospf authentication
R1(config-if)# ip ospf authentication-key cisco

```



4、在对应的 OSPF 接口下开启密文认证，并设置密文文认证密钥

```
router ospf 110
 router-id 1.1.1.1
 no area 0 authentication message-digist
R1(config)# interface f0/0
R1(config-if)# no ip ospf authentication-key
R1(config-if)# ip ospf authentication message-digest
R1(config-if)# ip ospf message-digest-key 1 md5 cisco
```



5、如果同时开启了OSPF区域认证和接口认证，则优先使用接口的配置


例如，
在OSPF进行下设置 Aera 0 使用密文认证，而在接口下配置了 `ip ospf authentication` 命令，则最终结果是使用明文认证。


<img src="/pics/ospf-auth3.png" width="400">

## NSX 认证对接

如果理解了上述理论，那么再回到NSX。

NSX 只支持三种认证模式和一种配置办法，即**空认证、明文认证和密文认证，且只能全局在区域下配置密码，所有接口生效。**

重点是，NSX 必须要求配置认证密钥，1~15个字符，而Cisco可以不设置认证密钥，明文密钥长度最长为8个字符(如果配置时超出则取前8个字符)，密文无限制。

熟悉以上原理后，NSX 和 Cisco 的 OSPF 认证对接就不是问题，推荐 **NSX 使用 md5 认证并配置密钥，物理设备在OSPF区域下开启密文认证，在接口下设置1~15字符的密钥。**

---

笔者做过锐捷、华为和Cisco交换机的OSPF对接，这两个厂商设备ospf认证原理同Cisco一致。H3C略有不同，具体可参考 [此文章](http://archive.halfcoffee.com/2016/12/10/h3c_huawei_cisco_ospf_auth/)。

最后，Cisco也支持调用key chain来进行密文认证。需要在key chain中定义cryptographic-algorithm。估计此种方式在多厂商设备间兼容性更差了。



## 参考文章

1、 [OSPFv2 Authentication Confusion](http://packetlife.net/blog/2010/jun/1/ospfv2-authentication-confusion/)

2、 [OSPF header checksum](https://learningnetwork.cisco.com/message/393900#393900)

3、 [Configuration Examples for OSPFv2 Cryptographic Authentication](http://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_ospf/configuration/xe-3s/iro-xe-3s-book/iro-ospfv2-crypto-authen-xe.html#reference_1E9C132BB5EE4550AA841075A95EBB90)



