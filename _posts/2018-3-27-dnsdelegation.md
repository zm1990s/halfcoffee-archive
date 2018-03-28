---
layout: post
title:  "DNS 委派配置"
date:   2018-3-27
categories: NSX
tags: NSX
typora-root-url: ../../halfcoffee
---

* content
{:toc}
## 摘要

> 今天在学习GSLB时，想到一个问题，就是如果企业内部有些服务器对互联网用户提供服务，那么必然牵扯到互联网域名如何对应到本地的DNS服务器。按照F5 GSLB的文档，一般是在互联网域名管理端配置NS记录，将子域名委派(delegation)给GSLB。
>
> 经过测试，有两种办法可以将DNS的解析转移到本地的GSLB上。一种是上面说的配置子域名的NS记录，一种是直接将域名的DNS指定为本地的DNS server，下面详细说说配置过程。

## 修改子域名的NS记录

**环境情况**：

1、在云端自建一个DNS server，环境使用Centos+BIND，可以通过域名**mi.halfcoffee.com**访问到此DNS 服务器。下图为此DNS 服务器上配置的解析配置：

<img src="/pics/dnsde1.jpg" width="700">



2、登陆公网域名管理，添加一个NS记录，将**test.halfcoffee.com**指向mi.halfcoffee.com这个DNS server

<img src="/pics/dnsde2.jpg" width="700">

3、测试，解析test.halfcoffee.com，可以解析到CNAME设定的域名

<img src="/pics/dnsde7.jpg" width="700">

4、文中自建的DNS服务器TTL设置为了最小，所以如果DNS记录有变动，通过nslookup查询可以立即解析到新的地方去。例如刚刚将CNAME改了下：

<img src="/pics/dnsde6.jpg" width="900">

---



## 修改域名的DNS

1、在云端自建一个DNS server，环境使用Centos+BIND，可以通过域名**mi.halfcoffee.com**访问到此DNS 服务器。下图为此DNS 服务器上配置的解析配置：

<img src="/pics/dnsde1.jpg" width="700">



2、登陆公网域名管理，修改域名的DNS服务器，将其修改为刚才搭建的域名**mi.halfcoffee.com**（图中要求必须指定两个DNS server，其实两个域名指向同一个IP地址）

<img src="/pics/dnsde3.jpg" width="800">



3、使用nslookup测试解析

```
#nslookup
> server 114.114.114.114 //指定当前测试机的DNS服务器
> set debug //开启debug
> test.halfcoffee.com
服务器:  public1.114dns.com
Address:  114.114.114.114

------------
Got answer:
    HEADER:
        opcode = QUERY, id = 11, rcode = NOERROR
        header flags:  response, want recursion, recursion avail.
        questions = 1,  answers = 1,  authority records = 1,  additional = 0

    QUESTIONS:
        test.halfcoffee.com, type = A, class = IN
    ANSWERS:
    ->  test.halfcoffee.com
        canonical name = halfcoffee.com
        ttl = 32 (32 secs)
    AUTHORITY RECORDS:
    ->  halfcoffee.com
        ttl = 32 (32 secs)
        primary name server = mi2.halfcoffee.com
        responsible mail addr = root.halfcoffee.com
        serial  = 1
        refresh = 1 (1 sec)
        retry   = 1 (1 sec)
        expire  = 1 (1 sec)
        default TTL = 1 (1 sec)

------------
非权威应答:
------------
Got answer:
    HEADER:
        opcode = QUERY, id = 12, rcode = NOERROR
        header flags:  response, want recursion, recursion avail.
        questions = 1,  answers = 3,  authority records = 0,  additional = 0

    QUESTIONS:
        test.halfcoffee.com, type = AAAA, class = IN
    ANSWERS:
    ->  test.halfcoffee.com
        canonical name = zm1990s.github.io
        ttl = 1 (1 sec)
    ->  zm1990s.github.io
        canonical name = sni.github.map.fastly.net
        ttl = 3267 (54 mins 27 secs)
    ->  sni.github.map.fastly.net
        AAAA IPv6 address = 2a04:4e42:11::403
        ttl = 30 (30 secs)

------------
名称:    sni.github.map.fastly.net
Address:  2a04:4e42:11::403
Aliases:  test.halfcoffee.com
          zm1990s.github.io
```


<img src="/pics/dnsde4.jpg" width="700">

4、测试访问，可以正常访问到网站

<img src="/pics/dnsde5.jpg" width="900">

