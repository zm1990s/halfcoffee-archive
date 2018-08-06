---
layout: post
title:  "NSX 从入门到精通-安全篇"
date:   2018-5-20
categories: NSX
tags: NSX
typora-root-url: ../../halfcoffee
---

* content
{:toc}
---

**“** 

有一天，同事突然说了一句话"你写的文章很好，但我看到5就看不下去了"。这时，我才意识到，并不是每个人都适合阅读系列的所有文章，我欠大家一个大纲，需要告诉大家哪些人应该看哪些内容。

**”**

****

截止目前，NSX 从入门到精通系列已经完成12篇。

写这个系列的原因比较简单：1、工作中会接触很多 NSX 项目；2、生态圈缺少对 NSX 的“正确”认识；3、相比 VMware 其他热门产品，NSX 非官方资料可谓少之又少。

似乎没有一个特别好的路径去了解 NSX 为什么好，好在哪里，除了熟知的安全外还能做什么，以及，最重要的，**对未来和市场的改变**。

与其等着一个称心的资料出现，不如自己去打造一份。那就立志长远，从零做起。

NSX 从入门到精通也是个很随意起的名字，一方面，这个名字代表了连续，可以将一个话题拆分成很多小模块去写，足够细，也就可以达到“精通”的地步；另一方面，是想直接通过标题就传达最终目标：这个系列=你需要的所有 NSX 资源。

NSX 从入门到精通系列面向对象是所有对 NSX 感兴趣的人，而针对不同人，可以读的内容并不一样。

**大致来说，NSX 从入门到精通会包含四部分内容：**

1、产品相关的背景介绍，功能简介（1~4，20~21）

2、产品相关的技术实现，操作（6~8）

3、产品相关的使用场景（5、9、10、11、网络部分待定）

4、最佳实践、排错（预计25+）

**而从模块来说，暂分两大块：**

1、NSX 安全：NSX 自身的安全、NSX 第三方安全解决方案（4~19）

2、NSX 虚拟网络：软件实现的全功能虚拟网络（20-待定）

对于销售人员，只需要了解产品的背景、功能、使用场景等；

对于售前人员，需要了解产品的背景、功能、使用场景、最佳实践等；

对于售后人员，最关注的可能是技术实现、操作、最佳实践、排错等内容。

因此哪些人该读那些内容，应该挺清晰吧！

------

**除了学习，这些文章还能做什么？**

经常有人会要一些产品的**设计方案**或者**实施方案**，这个系列便可以是不错的素材，直接复制粘贴，选用不同的标题就能代表不同的内容。

举个栗子：第5篇讲了 NSX 的安全转型，而转型前的架构可以=现有架构，转型后便是 NSX 的架构和价值；

再举个栗子：实施方案，包含设计和一些操作，6~8便包含了这部分内容。

这个系列，也可以作为 PPT 素材，一个 PPT 的产生=思路+素材，这个系列本身以及每个文章都会有自己的思路，文字和图片等于素材，距离一个 PPT 的诞生只剩下组织编排。

最后，还有一个潜在的价值，用于宣传。

实际会有很多 NSX 最终用户会在后台咨询一些 NSX 相关的问题，我相信产品再好，如果没有人知道能干什么，怎么用，那也会慢慢走出市场视线，所以，非常鼓励各位能帮助转发你觉得有价值的内容，分享给我们“潜在”的用户。

------

公众号后台的留言功能是开放的，也欢迎大家留言，可以咨询一些产品问题，也可以说出你的需求，我会寻找资源以文章的形式发布出来。

---





觉得有必要来一个从入门到精通，从第一次接触NSX到现在已经有三年时间，还记得我第一次自觉学习NSX的时候，竟然翻到了一年前听NSX讲堂的笔记，那些内容足够深足够细，然而我都不记得了，原因就是在于没有体系的从基础来了解这个产品。不了解一个产品的背景，单纯了解产品的卖点和知识点是做不好产品的。因此，第一篇，简单介绍下NSX。

首先，假定我的听众有两种类型，一种是做过服务器虚拟化的系统工程师，知道vSphere是什么；另一种，是网络攻城师，知道路由交换防火墙这些东西。如果是第一种，接着看就行，如果是第二种，可以补充看看网工篇，里面会与虚拟化网络的一些介绍

For 系统工程师：

如果你自己装过vSphere，应该了解过vSphere的虚拟交换机，也可能接触过物理交换机。在虚拟化中，无论是虚拟交换机还是物理交换机，其作用几乎只是做了“通道”，此通道从虚拟机的虚拟网卡-->vSphere主机的虚拟交换机-->主机的物理网卡-->物理交换机-->核心交换机-->客户的其他网络，vSphere vSwitch配置非常简单，两三步就可以创建好，对应的物理网络要求也很简单，一个服务器多个网卡统统接到交换机，所有端口配置为Trunk即可。

我们再看看真实世界的物理交换机，有哪些常见功能：

1、交换：同网段数据包的转发

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNaBvZ7es5KHMFH2e3ic15OzpM1vdmNYesuue1xaC1Jw90fxicOKkfMuKzJbZzHbStYCSQ50KKoTzeQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

2、生成树： 受限于以太网的设计“缺陷”。如果两个交换机A和B，中间接两条链路，配置一模一样，A上接了一个客户端C，C发了一个广播包给A，A会将此报文发给B，此时B的操作是在其**所有其他端口**上再转发此报文，也就是数据包又会发回给A。这个过程叫环路。生成树是来避免交换环路的，其工作原理是屏蔽掉多余的链路，让一个报文只有一条路可以走。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNaBvZ7es5KHMFH2e3ic15OzUgvLyWwXCibeF93G6VtDECtmz3a3tiblloplvMXstB7W1mPtic104EUZQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

3、端口聚合：把两个物理接口组成一个逻辑的接口去用，逻辑接口带宽是物理接口速率之和。这样在某些时候可以避免使用生成树。

端口聚合又分静态聚合和动态聚合(LACP)，每种聚合都有流量负载的算法，每个厂商对静态聚合和动态聚合的支持不一定统一。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNaBvZ7es5KHMFH2e3ic15OzJZWz3GCV9gzN2VEiaGRREYmnFef9c4zLU4oU3oKKgLFXToQWIRqEJfA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

4、VLAN：VLAN相当于把一个物理交换机拆成了多个交换机使用。如果没有VLAN，假如A和B两个团队共用一个交换机，那么A发的广播包B可以收到，B发的A也可以收到，这样很不安全。VLAN就可以将A与B简单隔离起来。VLAN是需要每个设备都配置的。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNaBvZ7es5KHMFH2e3ic15OzL2gbyrwl2CODFoPYEdYUyOoCk0eJu8kEKm91SU3Gl7ufgzKK2RHQ1Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

5、设备堆叠：为了简化管理，可以将多个交换机逻辑堆叠在一起，相当于将两个独立的大脑合并成为一个大脑，这样管理方便，两台交换机之间也不用担心环路问题。比较重要的是，做了堆叠，两两设备互联时架构可以变得很简单。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNaBvZ7es5KHMFH2e3ic15Oz0UF9bjt8DxmG9Wg4Eibv2ibyiaK1SjMZYhN6jD1YITu14x2YMWL9ChRmQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

以上是交换机必备的基础技能，有没有觉得已经挺复杂了，网络工程师很多时候面对的就是这些，这还不算安全。

6、MAC 地址绑定（Port Security）：目的是将物理端口和所接终端的MAC地址绑定起来，防止别人私接设备。但是破解此问题很简单，使用工具改下私接PC的MAC就可以正常入网了。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNaBvZ7es5KHMFH2e3ic15OzlyT51BicULVpBOyuwuw5JicwLpV2QwQ8LWXK66WEjNtic2XfQapb0o6NA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

7、SpoofGuard ：利用 DHCP + ARP inspection等手段，实现 IP 地址到 MAC 地址的绑定。主要是防止用户私改IP。

8、ACL：访问控制策略，主要用于控制接口上能通讯的报文，17年勒索软件爆发，很多企业都在交换机所有接用户的接口上加了ACL来封堵某端口。

9、Private VLAN(或者Protected Port)：可以让同VLAN的多个终端之间不能通，但是到外部网络通，做到横向隔离

其他高级功能：

10、Storm Control：用于限制接口上的广播包，组播包，通常可以用来防止恶意的广播报文攻击。

11、SPAN：将一个端口的流量镜像到流量监控设备，用于分析流量

------

以上大概是所有物理交换机常见的功能，实际在虚拟化中，vSwitch 能实现的有：1、3、4、11

不需要的有：2、5、6

需要但没有的：7、8、9、10

------

实际上，7、8、9、10都是与安全相关的需求，在有了虚拟化之后，一台服务器的一个接口会承载很多虚拟机，因此外部交换机的安全特性不能用于虚拟化，这块一直是空白！！

现在NSX就可以解决以上提到的遗留问题，而且是以一种优雅的方式：

对于8和9，NSX的微分段功能就能做到任意虚拟机到任意位置的访问控制。

策略配置就像下图一样简单：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNaBvZ7es5KHMFH2e3ic15OzficueD9aMYQDQlhWHsFyO6o5pSicZ1eQx3pkgp3jBWBe1f8YV37GpjjA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

而如果要用交换机做这样的配置，需要在每个接口下写至少这么多命令：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNaBvZ7es5KHMFH2e3ic15Ozl18rXwZUlQaDOtnic1o6I2fIoUpEibcnNhrIuqX08ibkon9C699b93H9A/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

对于7，NSX 自带Spoofguard功能，不需要借助任何其他功能，直接开启绑定就行。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNaBvZ7es5KHMFH2e3ic15OzvI7uNxDyPw5qom5zWcTZr70CbO6hOBAoMHJFhaARFibtLxzSd4vRQMA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

对于10，NSX什么都不用配，装上就可以做到广播抑制。

基本上，NSX的基本安全功能就是如此简单，在这个基础上，可以延展出很多场景，和其他很多产品结合，这个我们以后再说。



接上一篇  [NSX从入门到精通(1)：NSX介绍-系统工程师篇](https://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483697&idx=1&sn=9f078cb1d1448e3135a370e92a8d6bac&chksm=f982711fcef5f80958d58e77436b82d6876a0092b45faa4e29f68355e54e19272f0a2901ef19&scene=21#wechat_redirect)，此章节主要补充介绍下vSphere，通过上一篇大家可能发现，NSX更偏向于一个网络产品，未来如果要做复杂的环境，少不了网络工程师的参与。这里小提一下，其实现在行业的发展趋势是知识变得越来越混杂，如果做数据中心的业务，关于数据中心的四大件：计算、网络、存储、应用都需要了解，不一定精通，但四者之间如何共存如何设计，还是要清楚的。

For 网络工程师：

网络工程师，应该清楚Access口和Trunk口的区别，简单说Access口接终端、Trunk口接交换机，面对于服务器，很多人第一直觉是配成Access口，但这在虚拟化的世界里，是不推荐的配置，下面从这个角度谈谈虚拟化网络。

首先，什么是虚拟化？通常我们提到的虚拟化是指计算虚拟化，计算指的就是计算机，这个计算机可以是我们的笔记本，或者机房的X86服务器。

在虚拟化之前，一个计算机上面只能运行一个操作系统，一个操作系统上运行很多应用，这是大部分办公计算机的场景，在商业服务器场景中，这样的配置有以下一些问题：

1、大家都知道电脑软件装多了软件，电脑可能会变卡，问题就是运行的程序太多，**资源争夺**。

2、很多应用程序共享一个操作系统，应用之间**没有任何隔离手段**。举个简单的例子，你在电脑里装了网银客户端，我装了一个网络抓包工具，那这时候我可以通过我的抓包工具看到你的任何报文。

3、操作系统本身会存在漏洞，其他应用也会有漏洞，抛开应用间隔离不说，**一旦操作系统中毒，所有应用不可幸免**。

4、解决1、2、3很简单，每个应用用一个操作系统，每个操作系统用一个硬件。但对于企业，**硬件扩容远跟不上软件的扩展要求**，**成本、空间上也不允许**每个应用用一套独立的硬件。

5、即使解决了1、2、3、4，如果硬件坏了怎么办？硬件故障上面运行的应用必然会宕机，解决办法就是将应用直接开发成集群形式，而现实是**开发集群式的应用成本会很高很高**。

为了解决以上5个问题，有了计算虚拟化，简单来说就是在服务器上安装“瘦操作系统”，此操作系统只用于资源的调度分配，这个操作系统有自己的管理界面，在这个界面里，你可以创建虚拟机(虚拟机和物理机可以做对应，物理机是真实能看到的计算机，虚拟机则是被模拟出来的，真实看不到的计算机)，给这个虚拟机安装操作系统，然后再操作系统里装软件。

有了计算虚拟化：

1、提升CPU、内存的利用率，**降低硬件投资**。如果硬件上直接装操作系统再装应用，拿Web服务器为例，很可能CPU、内存占用不超过10%，90%的资源处于空闲；在虚拟化之后，每个硬件上可以运行8个操作系统，CPU、内存占用达到80%，只有20%资源剩余。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrNjZKDB4ha5nbQYNJt8QqbaCRypn1VWCico42zoUvfsBmV0CFlviaCvHQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

2、硬件利用率高了，空间占用就小了，对机房制冷、耗电也会减小，**降低运营成本**。

3、做到**虚拟机和虚拟机之间的隔离**。在传统的计算虚拟化之上，可以做到虚拟机存储、CPU和内存的隔离，网络还是个空缺。

4、按照业务重要性分配不同的优先级，**保证重要业务稳定运行**。

5、**动态按需分配资源**。比如一个业务刚上线需要的资源很小，但一段时间后用户剧增，处理性能跟不上，那直接通过虚拟化增加内存和CPU即可。

6、在硬件和虚拟机之间多了一层后，虚拟机的操作系统可以无需关心底层硬件的兼容性(比如 Windows XP 很难在新电脑上装成功，但有了虚拟化随便装)，因为虚拟化有一个很重要的作用就是将硬件做了**硬件标准化。**

7、因为上述的标准化，虚拟机理论上就可以在任意硬件上运行，虚拟化还提供一个叫迁移的功能，可以**在线将一台虚拟机从主机A迁移到主机B，业务不会中断**(VMware 称之为 vMotion)。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4Qr5nViaHAMgRT1mYuY0DUDkNL7BbdwUpzWElehXMn0IvJm2hgianR5k1tw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

8、虚拟化之后，多台硬件设备可以组成一个集群，有了集群，**虚拟机可以动态在多个主机之间迁移，保证集群中硬件的使用率相对均衡**(VMware 称之为 DRS)。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrpfxVJx3SDwDsiaCNNTYwT75Py6fiadbaBU1AtPpwNiaouKicDMmBSv9c4w/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

9、有了集群，**虚拟机能够在其所属主机故障后，在集群其他主机上自动开机**，此时业务有中断，但是中断时间在分钟级别(VMware 称之为 HA)。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QricNEngjdhNCuvlfpYbTMZ6s9aM8GubQaqZPKXp29u8N7S5UWnXABUng/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

10、统一管理，一个管理界面，可以管理成百上千个业务。

11、如果架构合理，除非硬件坏了需要更换，否则不用进机房，在电脑前就能完成任何故障处理。

一开始提到传统架构的5个问题，然而虚拟化带来了十几项易用的功能，这就是虚拟化可以风云十几年的原因，因为很好用！

细心的你可能注意到了第三项，**网络的隔离是个空缺**。 

通常，交换机和物理服务器对接，端口只需要两个配置：

1、设为Access口，将其放在某 VLAN

2、如果一个网口带宽不够，或者为了保证冗余，两个接口做一个端口聚合

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrV9C9lFg6woaGu4ic7YRPu4JodQCHTAqscQgNRVHBzVmC7sTQhE5kf3A/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在虚拟化中，这样配是不行的，因为一台主机上可能会同时存在VLAN 10、11、12的虚拟机，物理交换机对应接口只能配置为Trunk，放行VLAN10、11、12（为了简便以及未来扩展，最好不要在Trunk去限制VLAN）：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrMKTLE2UL9vuuZ4m6qHmJoUue2icT1Q1fBYia1NwTibMuIgqpx2NCuMVGw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在这样的结构里，虚拟机发的包必须有VLAN标签才能正常让交换机转发，这个VLAN标签就由虚拟化里的**虚拟交换机**来打了。

普通交换机有接终端的接口(一般叫Edge边缘接口)，有接其他交换机的(一般叫互联口)。

类似的，虚拟交换机有接虚拟机的接口(Internel，接虚拟机的虚拟网卡)，有接物理交换机的接口(Uplink，关联到物理服务器的物理网卡，再连接到物理交换机)。配置VLAN方式略有不同，交换机配置VLAN是直接在接口上敲VLAN xxx；**虚拟交换机是先创建一个端口组，给这个端口组关联VLAN，然后再将虚拟机关联给这个端口组**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrdIg1QddQ5z2t1ibDAiax36UZKiacnLAso4g32qQqI4Oxic6dYqOL0JlfWw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

这里着重提一下，**虚拟交换机是不能和其他虚拟交换机直连的，或者理解为虚拟交换机就是虚拟机到物理交换机的一个桥，只做通道用**。不能相互连，也就意味着没有传统交换里的二层环路，也就不需要生成树协议。

如上图所示，服务器有两个网卡，没做任何特殊的配置，多个虚拟机的流量自动从两个网卡传输，简言之，**虚拟化里，没有端口聚合也可以实现负载均衡**(暂且记住这点，不要每个和服务器对接的环境都去做端口聚合，未来会展开讲这部分设计)。

说了这么多，其实想表达，在虚拟化这么长时间里，计算虚拟化能有那么多的特性和优点，而网络层面却只做了个“桥”，严重依赖于物理网络设备提供的特性，没有安全功能。NSX的推出，彻底改变了这一点，原来的网络是一个桥，NSX之后的网络融合了安全、虚拟二层、三层网络，兼容现有网络，支持各种标准网络协议。

以前虚拟机需要网络功能，在硬件交换机、路由器、防火墙、负载均衡、IPS、WAF 挨个去配置，去串接，架构无比复杂，有时候还达不到想要的效果（颗粒度不够细）。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrPBElCdNGX0kqiaoMpXVavOniaK1JqY32RxLkFhDbXOhOqAl0Z22b1Aicg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

NSX之后，自身提供基础功能(路由交换安全)，提供接口引入高级服务(NGFW、IPS、防病毒)，重点是，以上产品可以无缝集成起来，实现网络和安全的联动。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrwDwPb8neTvKkl3mF6zNXBjGibkuF5vrukvkZpcoK5DoTkJnNtPW2ccQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4Qrrfv0udHxSGIAAVTib4cKjd8l8WWKGtURB7mqFwPXhrxvDz9nYxnpnjQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

那既然网络都虚拟化了，还要硬件网络做什么？

1、**NSX 仅用于数据中心**，传统园区网络的一大部分是不去管的。

2、**NSX 需要硬件交换机来做传输通道**，只是将其功能弱化，而不是替代。

对于技术人员来说：**NSX 也需要网络架构师来设计网络呀，发挥才能的时候到了。**

正如计算虚拟化对硬件服务器的改造一样，NSX是对数据中心网络的改造。计算虚拟化解决了5个问题，还增加了几十上百个优化，NSX在这基础上，又带来超级多的便利。





首先，强烈建议大家先阅读[ NSX介绍-系统工程师篇](https://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483697&idx=1&sn=9f078cb1d1448e3135a370e92a8d6bac&chksm=f982711fcef5f80958d58e77436b82d6876a0092b45faa4e29f68355e54e19272f0a2901ef19&scene=21#wechat_redirect) 和 [NSX介绍-网络工程师篇](https://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483712&idx=1&sn=65b665c274fbcf16fb389c4b198b6410&chksm=f982716ecef5f878492752148fba45c5e68cd60d90c8185999494449f6a2be9c5bfc66a0d71e&scene=21#wechat_redirect)。在前两篇中，我从两个视角介绍了网络和虚拟化，这篇来说说现代化数据中心如何建立，以及未来会是什么样子。

以一个案例开始：

听众中一定有人听过“割接”一词，最早这一词用在电信机房，要做新老设备更换时，为保证切换切换最短，会让新旧设备一起并行工作，然后在某个风清月圆的夜晚，大家拿起剪刀一起剪掉旧设备的线，割接完成后，系统或业务通过手动或自动切换到新设备上，再拆掉旧设备，这整个过程就是割接。

到现在，割接过程和上面大同小异，但有些人可以做到很优雅的割接，有些人只能半夜12点准时开始，早上8点结束。两者最重要差别就是“**割接的操作时间不一样**”，这个背后的因素可能是**架构复杂，或者设备众多，或者连线过多，或者切换方案不平滑**。

有些听众此时会微微一笑：因为我是卖服务器的，服务器到场，预装好了虚拟化管理程序，IP地址规划在项目一期已经做好，配好IP，接入网络，加入集群，测试基本功能，撤人。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrBz8JibsfiaMUMMTL75W2gicbsZ1ymxY7dBTswJS3Ct3y2VmV7T81xxkeg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

有些听众回忆起了痛苦的过去：提前很多天确定割接方案，利用旧设备配置编写脚本，配置新设备，测试功能；割接当天下午到客户现场，割接前准备，备份旧设备配置，到点后割接，拔线，装新设备，插线，测试连通性，测试业务。中途无故障两三个小时，有故障排查，排查不出来回退，轻松到第二天8点。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrkNcUxW9t8rZQ2CnFFlJDx7YjANYxicGGtK40Z0qJohjUibXx3LFick23Q/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1)

区别两张图片就可以表现出来：**网络的割接远比服务器的复杂，接线更多，且很多时候是原位替换，很难做到平滑。**

再往细里说：服务器借助于服务器虚拟化，单台主机可以在线进入维护模式，直接进行设备更换；可以在线添加服务器，扩容。交换机一方面做不到更换前后设备配置一致，一方面做不到在线下电。

解决问题的关键就是“**标准化**”。

VMware在13年提出SDDC(软件定义数据中心)的概念，旨在通过一层软件来做到数据中心各种资源的标准化。在软件层之上，是最终业务系统(VM)，在软件层之下，是各式各样冰冷的硬件设备。标准化后，上层业务可以不用关心底层硬件，在**任意地方运行**。

2017年，SDDC的概念发展为现代化数据中心：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4Qr819nJsjYNibib9pNib2me1f89ISeXGraUasPgUag9KcdOW7v7H9pphmQQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

这样的图景不是天方夜谭，Google早已步入这样的阶段。

标准化包含的不是几个简单的产品，将一些硬件做个标准化，还包含架构的标准化，服务的标准化，操作流程的标准化。

硬件的标准化：

1、**VMware vSphere**：服务器虚拟化老大，将计算资源做到了标准化。

2、**vSAN**：将每台服务器的本地硬盘组成一个逻辑的存储池，使用软件策略为业务提供不同的存储容量、不同的存储服务、不同的保护级别。vSAN为上层虚拟机提供一致的存储，底层可以是任意厂商的服务器，搭载任意品牌的硬盘(同vSphere一样，有兼容列表)。

3、**NSX**：为虚拟机提供一致的网络服务，包括常见的交换、路由和安全，**极大简化了物理网络的架构和配置**。

4、**带外管理网络**：这是很多数据中心忽略的地方，VMware的软件让80%的操作通过远程Client进行；19%则是硬件本身的管理，例如排错，重启，更新固件；剩下1%才是硬件真的故障，去机房进行硬件更换。**如果建立好带外管理网络，将所有交换机、服务器带外管理网络建立起来，那有99%的工作都可以通过远程完成，不需要去机房**。

架构的标准化：

架构包含虚拟化层之上的“虚”的规划，和虚拟化层之下的“实”的规划。

具体到技术：vSphere的规划、IP地址规划、命名规划、虚拟机规划，软件层面的设计追求完整可扩展。

以及：物理设备带外管理网络规划、接线原则的规划、功能配置的规划，硬件的设计追求简而稳固。

以上完全可以用一张表格实现，在同类项目中直接套用。

*下图为双中心VMware虚拟化的网络规划及硬件标准接线图。*

**

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrebIsZu3kibRVUibyVM9SZEFclvhedDUJCR2vKH7NzCFNDISwCBSpnHdg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

**

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrH5icFyq696nXibuUOxtGrS4lic2ibuPhHq4ibIOGa29Plz1BMhsuRfsJia4Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

服务的标准化：

**IT 系统的最终目的是给用户提供服务**，提供服务的方式有两种，被动和主动。

在做完以上基础架构的标准化之后，终归要给业务部门或者最终用户提供服务，IT 管理员类似于服务供给者，在没有服务标准化之前，业务部门要什么我们提供什么，长久下来业务逻辑千疮百孔，基础架构管理员也无能为力了(比如我遇见过一个虚拟机使用了64G内存，24颗vCPU)。

如果在最初，就定义好标准，未来用户只能选择“安全区”内的资源和服务，那一方面可以提高服务的供给速度，降低IT运维人员的工作负担，一方面也可以很好地进行监控和控制，避免各种不合理发生。

这实际是很多云计算平台在做的，云计算包含以下关键词：

1、量化：量化才能决定最小单位，决定如何分配资源

2、运算资源：CPU，内存，存储

3、网络：有了网络才能到达每一个地方

4、界面：有了界面才能提供通用性服务

5、服务：将资源加壳变成服务的模样，方便计价和计量

6、用户：需要服务的对象

VMware SDDC做到了前三项，云管理平台做4和5，用户一直在那里。

*下图为VMware vRealize 蓝图界面，在蓝图中可以将计算、存储、网络、安全、操作系统、应用、业务逻辑等资源进行标准化模板定制。*

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrPmkGgtem1cTotuJWrknq0ibH4etckUSZx5Xt4nUbyatW2njQSrrPvmA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

操作流程的标准化：

操作流程标准化有一部分包含在服务标准化中，服务制定包含了要提供的资源以及如何获取这些服务(也就是流程)。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QrjfuBPr6CTKWZic7bBiaOkRkOl6Uuy6Yl3ibDFkx9r2Ra4cicfkic1QzM6Og/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

另一部分是对运维人员而言的标准化流程，**虚拟化固然好，但前提是正常操作**。如果在不将主机进入维护模式直接下点，就是存在一些风险。

在操作流程标准化里，包含日常运维的内容，每个不同的组件都有自己特定的维护方式；需要有设备更换的流程；需要有故障排除的流程；需要有合理的升级流程。

*下图为一份VMware组件升级顺序*

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNLgeVUfySvESsqPvAIj4QribcQwYk3Cq3LpZ6j1fc1viaX2aHSAbalR0dLKFO8NHoPPkC8VI7LDK7Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

以上，做完各种标准化之后，剩余一个不近不远的问题：**新架构的支持**！

最近几年常谈的关键词是：容器、人工智能、区块链、混合云。

其中**真正需要基础架构做出适配的只有容器和混合云，**而现在的 VMware，都是支持的。



从此话题开始，会依次介绍一些细节或与具体场景相关的NSX知识。有些话题目前写时机还不成熟，所以未来文章的序号会有跳跃性，为一些话题预留空间。

安全是一个比较大的话题，从不同角度看有多种安全。以一台个人电脑为例，如何才能保证电脑的安全？

首先要清楚安全的最终目的是什么。对于个人用户来说，**最重要的是数据**，比如珍贵的文稿，珍贵的照片。

普通电脑的所有资料只是保存在单个硬盘之中，保证数据安全的第一步就是避免单硬盘的故障，最简单的解决办法是两个硬盘保存同样的数据。在企业环境中，同样地，任何的业务系统首先会做到存储层面的多设备冗余，避免物理设备的单点故障。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05Tvick1NBiad3j6Ex9yrC01AeV9eXjQTEzJ0yJhpZVeiaRz4rjHcfjvRcPBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

但是第一种办法有个缺陷，假如电脑丢了，或者误操作删除了文件，普通用户很难去找回数据，那这时候有一个移动硬盘备份就很好，定期备份重要数据，如果误删除，使用历史备份数据恢复，也不至于损失太大。企业环境里，会有专业的备份系统支撑，相比移动硬盘，设备可靠性更高，且可以近乎实时的备份数据。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05Tv026Hdr88tvWfPjKnCiaYx78c3xBnmYIDiaEY3fT2NiaAQXWNNz6w1Dgxg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

前两种办法做了数据层面的安全，**但****只做了防丢失**，而很多时候**重要的数据也****要求防窃取**。

曾经去一些涉密单位做项目，经常看到他们屏幕保护程序上会有动画演示，要求所有终端不接U盘，不连互联网。还说不要将涉密U盘插在家用电脑上，即使断网也不行，因为家用电脑上可能存在恶意软件，在电脑联网后，恶意软件会将涉密文件上传到互联网。

上面演示的是一种最基本的数据泄露的途径。围绕上述场景有几个关键字：**恶意软件、互联网、人员**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05Tvh6oib6XFbwmNMd73LTLibFXozEpcXjKWAQJjzdqicbM8ibfuibKE1ChkMdw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

企业为了解决这些问题需要使用各种各样的手段。为了避免恶意软件，需要在每个终端上部署**防病毒软件**；为了避免恶意软件通过网络进入终端，需要在终端和互联网之间加一道**防火墙**；通过技术手段或者行政手段，要求人员禁用移动存储设备传输涉密文件，禁止将涉密文件传输到互联网。

以上是最基础的防护手段，从原理上说，防病毒软件需要通过病毒特征库来杀毒，而特征库是基于病毒样本，也就是说防病毒只能做到“事后”安全，为了将杀毒软件作用发挥到最大，需要在第一时间更新病毒库。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05TvoZ7QK2apJ2qYtCtAd3Gd2A4ODjACWbdyCgK4AibXYNYL7F0iaLiaF5zEw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

防火墙分传统防火墙和下一代防火墙(NGFW)。传统防火墙只做基础的阻断，让流量按照我们预期放行或者阻止，其策略形式为：电脑A禁止访问服务器B；NGFW在传统防火墙的基础上，添加了一些针对应用、用户或内容的访问控制，其策略可以是：用户A不能将文件B发送到互联网，或者用户A不能访问优酷等视频网站。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05TvkWibEFg4ia42XurPJ1cCeXqtOzAIDZ1iboQ3iaOPtQIZ1HxDYYWiaax3NoA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

针对人员，普通企业的管控可能为0；或者基础防护：每台电脑装防病毒软件；或者高级防护，每台电脑里再装文件加密软件，文件在其他地方被打开时要求密码；或者再高级，员工使用Flex或者VDI解决方案，所有与企业相关的数据都在数据中心统一保管，通过技术手段禁止员工进行敏感数据的传播。

*下图为标准VDI(虚拟桌面)解决方案示意图：*

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05Tv6q22NX19Flv26cXZ3CSicJ5qIzX2SKnLC0RbkdOgJZqoMS3jvkD3CUA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

然而，做到以上并不安全，以2017年全球爆发的基于“永恒之蓝”的“WannaCry”为例，此恶意软件的传播途径是操作系统的漏洞，利用的是Windows的一些常见端口。**传统的安全的前提是：****在我允许的范围内，没有安全风险**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05Tv0YehoSvUdjFYQ8lk6E1P1A44Nt3oYyL1AoT7NZhicGsibr3GfDSYTFHQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

现实很残酷，现在更多的安全事故都发生在“允许的范围内”，比如很早就有的针对网站漏洞的注入攻击，简单说就是攻击人员利用合法的访问权限，利用一些网页程序自身的开发漏洞进行的攻击。注入攻击和WannaCry，可以同归为“**利用应用的漏洞来发起的攻击**”。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05TvpXro0CGH59Dm5e0J7mEOcfWicfFZ05uCuNrXZ241Jv21eccvibM9JQDw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

除了此攻击，还有针对用户的社会工程学攻击，比如收集与某个人相关的个人信息，如手机号、生日、或其他网站登录时用的一些登录凭据(曾经CDSN暴库后有大量登陆信息)，通过各种各样的信息发起对个人的攻击，攻击后的最终对象可能是电脑、互联网账户等，不过这类攻击主要面向与人，且攻击成本比较高，对于企业来说，防护的方法或许就是强制要求每个员工定期修改强密码以及**用动态登陆口令替代固定密码登陆**。

以前也遇到过 iPhone 被神奇重置，开机后要求输入密码，而与 iPhone 关联的 Apple账号已经无法访问。这可能是针对于个人的社会工程学攻击，或许可能只是在某个公众场所连了“**错误的WiFi**” ，或许是装了不该装的App。其实被重置还好，要是iCloud里的数据丢失才是大事。现在很多企业允许 BYOD(Bring Your Own Device)，更多员工也在使用移动设备进行办公，处理企业敏感信息，在这样的环境下，**移动终端的安全也是未来必须关注的**。

从以上的种种介绍，可能有人注意到了，**安全风险在向更精细化的方向走****，而这时候，安全防护也要向更精细化走**。

**NSX 的安全，正是为了达成这一目标。**



NSX 的第一个变革，是改变安全边界：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05Tv5V6SfZg7WQibpxQoibQm6kWDPaK28nRkmejwiccBazN4VbcpfWax6Co2g/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

为了让安全策略可以更精细化，**传统思维是在每个有通讯的业务之间(边界和边界间)加一些防护策略**，如果这样的方式使用物理防火墙，那未来网络拓扑图就是上面左图所示，实际，上左图准确应该表述为业务逻辑图，是未来安全策略要达到的效果。真实的防火墙部署应该像右侧一样，**在最接近业务系统的地方(虚拟化中即为虚拟机的虚拟网卡)，部署一道防火墙**。

右侧，就是真实的 NSX 防火墙的部署图。听众可能会想，这么多防火墙怎么管理？在 NSX 的世界里，所有防火墙都可以统一一个界面管理！即统一管理分布式处理。

实现原理很简单：

1、**与每个虚拟机的虚拟网卡最近的位置是vSphere，NSX 的防火墙就运行在每个 vSphere 上面。**也就是说，在NSX之后，所有业务流量第一跳经过就是安全设备。

2、**统一图形化界面配置所有安全策略，最终安全策略会经过筛选只发放给与其相关的虚拟机。**这样的方案既不失管理性，也不会因为全局策略过多导致防火墙性能下降。1000个策略应用给1000个虚拟机，每个虚拟机只继承一条策略，其性能和1000个策略分别应用于1000个虚拟机的性能是有质的差别。

3、**NSX 与 vCenter 关联后，可以获取非常丰富的可以描述/区分业务的信息**。例如虚拟机名称、虚拟机操作系统、登陆用户、网段，真正**做到安全策略与 IP 地址弱相关**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05TvKRFX3Z8eUyicEtmSCUV6icUxLq85fSvwbEnaxtS6vImkkiboqLnNnICPA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

NSX 的第二个变革，是改变安全思路：

在 NSX 的世界里，有一个词叫 DMZ Anywhere。DMZ 是网络中常见的一个术语，运行一些易被攻击或攻破的应用，在DMZ区和后端的服务器端会设置一道墙，避免DMZ区被攻陷后影响后端最最重要的数据。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05TvicsqBW6bXlURe3bDPx7ia5gicYWlrJDbcQT8DStEly1BzQhZmTeDtrHoQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

前面提到这样的安全边界的定义已经不够，NSX 提供的解决方案叫“微分段”。基于微分段，可以做的两个事是：

- DMZ Anywhere：**根据业务来将一个或多个虚拟机定义为一个安全域，在此基础上设置域间安全策略**。
- 零信任：限制**任意业务虚机到任意业务虚机不能访问**，通过白名单方式只允许应用间授信的访问。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05Tv5ecgsBnZ3fzmLUBza8Ms10VsxwlEc47ohszgqaUAkSrIsdM2oKibDdA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

上图是个简单的环境，使用NSX实现图示的安全只需要三步：

1、定义两个以部门为单位的安全组

2、禁止部门间相互访问策略

3、设置Web层虚拟机之间互访

NSX 的第三个变革，是打破安全产品间壁垒：

之前的文章多次提到传统网络和安全的一些问题，有个重点是平台不够标准化，NSX 同 ESXi 一样，实现了网络和安全平台的标准化。

在安全方面，NSX 将**虚拟机终端安全的接口(EPSec)**和**网络安全接口(NetX)**集成在了一起。以NSX为中心，上层是业务虚拟机，下层是第三方安全产品，NSX提供标准接口、提供基础的安全功能，通过接口和基础的安全功能，将一系列安全解决方案融合在一起。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05TvvcLjus24cxKZRa4GOqtjmcgMVsdMmEN3W7BLLibaOobaFSWAIvvoEpQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

NSX 的第四个变革，是改变了安全能触碰到的最小单位：

NSX目前有两个版本，一个是适用于vSphere环境的NSX-v，另外一个是兼容更多开放平台的NSX-T。

在vSphere平台内，NSX-v做到了安全策略细化到每个虚拟机的每个网卡；

如果将NSX-v用于移动终端环境，还可以做到 Per-APP VPN。传统的 VPN 大部分是基于设备的，就是说**一个 VPN 连接可以被多个移动终端上的多个应用使用**，一个VPN的后端，直通数据中心内部，安全风险显而易见。NSX 的解决方案就是做到**授信应用****到数据中心内****指定业务的访问****，消除了任何多余的访问权限**。

**为了实现下图的效果，需要搭配使用 Airwatch*

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05TvdZ4YJc8ianOkMt9gNqnm0HCYxeFXuVl5ibEI7qNXgCUD5appOTxia2eibA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在 NSX-T 环境中，安全变得更接近应用：

NSX-T 支持vSphere平台，也支持KVM等虚拟化平台；

NSX-T 支持新型的应用基础架构：容器；

NSX-T 支持 AWS 等公有云平台。

NSX 目前在集成 SD-WAN No.1 VeloCloud 的解决方案，未来安全更是会到每个网络可及的地方。毕竟，NSX 有一个小目标： NSX Anywhere

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05TvJ9aic1GaS7aciaPcwbvP42HwibdBr2mVuCblQtOf8vZpKicJDb7aRmKF5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

NSX 最后一个，算不上变革，只是弥补空缺的功能：虚拟化里的网络流量可视化：

传统的流量分析工具有两种获取流量的方式，一种是要求硬件网络设备将**所有流量转发**给分析设备；另一种是采用 Netflow 等标准协议，将网络流量的统计信息发送给流量分析设备。

第一种方式在小规模下可行，在稍大的环境中，让一个设备将所有流量转发到一个地方已经不现实了。

第二种方式，Netflow 是个比较简单的协议，最多只能用来进行统计、分析，满足诸如排错、监控网络异常、绘制拓扑这样的需求就不够了。

NSX 将虚拟化的网络和安全统一管理起来，任何与虚拟机有关的网络监控、排错、拓扑都可以轻松实现。配合 VMware 的其他工具，可以实现**从虚拟化到物理设备，从业务虚拟网卡到途经每个逻辑或者虚拟接口的可视化，实现 360° 网络可视化。**

关于监控这部分内容，可以读读“[NSX之后如何运维](https://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483750&idx=1&sn=0ec79de4d16a905a5505aab02149c1b0&chksm=f9827148cef5f85eca335f0cf8bbc9756d1020b3f011520b6deb65002340f856f4b68f55d322&scene=21#wechat_redirect)”，未来可能会有更详细的针对每个产品的介绍。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNQIOsYjLCFgiapicYgOPR3sxCZHaic0mF12yRxh7tr3hTG8qF3ITjQEQlzZF3l6vdp4Qq2ZRmyZibhVw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

后续：

文章写的比较长，能看到最后的都是真爱，不要忘记点赞哦！希望通过这一篇文章，能让大家能够熟悉 NSX 在安全方面的变化。NSX 安全是 NSX 诞生以来应用最多的一项功能，在以往很多话题里我将 NSX 描述为网络与安全，主要原因是** NSX 有着相对独立的两个大的功能集网络与安全，功能集之间可以没有任何联系，对于没有网络基础的读者，安全是一个非常好的入门方案**。接下来，我会分享与 NSX 安全相关的场景、功能细节以及部署，在这之后，开始网络的篇章。

[上一篇](https://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483778&idx=1&sn=12ad2a474588eb24d7542d46b24b58a3&chksm=f98271accef5f8baef234a74a97d40536ca4afccfad8a396a99d8ce3dca3f77dff75f0668018&scene=21#wechat_redirect)提到NSX在安全方面的一些特点，其中突出的优点是安全的细颗粒度以及安全的思路。这篇讲一下 NSX 如何去实现这些变革。

NSX安全的核心组件叫"分布式防火墙"(Distributed Firewall)，传统防火墙都是集中式的数据处理，而 NSX 将防火墙功能放在了每台 ESXi 上，生效级别是每个虚拟网卡，实现集中管理和分布式处理。

从架构上来说，NSX 防火墙分为两层：**管理层和数据层**。

管理层包含**NSX Manager**，NSX Manager 为单独的一台虚拟机，一般同 vCenter 在一个网段，作为虚拟化的管理组件。

数据层是最终进行数据处理的地方，包含两个组件：

- 主机上的**vsfwd**进程(/etc/init.d/vShield-Stateful-Firewall)，负责与NSX Manager 通信，接收 NSX Manager 下发的防火墙策略。
- 主机上的**esx-vsip**进程，最终进行所有网络流量的处理(放行还是阻止)。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOy61iaa6UYOL2jMxwicBdI6sPens8834emPAkPYeCvbcUmHgsF3APrw8Xib3bnYXRVeE2jQNPTIuSAw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

下面我们 Step by Step 部署一套 NSX 防火墙：

1、打开 vCenter，选择部署 ovf 模板

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOy61iaa6UYOL2jMxwicBdI6sU1Ze0RicFKPCFesb39PrfrTILOwVCQUlHW9pZbwd4YjUexJZ5dnky5g/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

选择已经下载好的 NSX Manager ova 文件

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOy61iaa6UYOL2jMxwicBdI6sDd1HPkoNiblS5sVqV2QG2QOib2zYICBEEicOnic9yOsFqGtzloONdaOzHw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

按照向导完成 IP 地址以及密码等配置

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOy61iaa6UYOL2jMxwicBdI6s9F8VSEYAd86wNwwmicnMROUckSQUkxpsErhnQtGdRnpPBnMCJQRukicQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOy61iaa6UYOL2jMxwicBdI6sd1NLn1bFxntmoshhuqSgGyx47Ac9cPX1A85MG8iaJfSp1LA8nWq7poA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOy61iaa6UYOL2jMxwicBdI6sibicicuhbiaI6FyFd4oG3MpJCvuL6MqtGibPCvCfl34smC2Jw69ht3H9eIw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

2、将部署的 NSX Manager 开机，浏览器打开 https://NSX-Manager-IP 并登陆

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOy61iaa6UYOL2jMxwicBdI6sN4aU257TOTAhERYuCo5W3hb44W48aicmSY8IH7iaDt44dJGck9xFXR5w/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

3、点击"管理>通用"进行基础的配置，例如配置时区。（此处建议语言使用en-US，否则未来做 NSX Manager 备份时可能出现兼容问题）

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOy61iaa6UYOL2jMxwicBdI6sjDsaqiaWsCIiabaF4icVSatzcjmT36RDgsc2MrdN7zbEeEjaaXH43RBfA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

4、进入“Manage>NSX Management Service” 配置 NSX Manager 和 vCenter 以及 PSC (lookup service)的关联

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvnXWicOTibOLibwpPd1ZfTBjRqPBvrfNibPao7DoZ7dtj3YeTMVOerCV8YQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvKHsEGKBPumS88vNic1bw19kfr12JcaE4uq1AM3icZJqz9SqvBLr3LEhQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

配置完成后两个连接状态都应该为绿色Connected

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvRQx9BVyVfmmRL7NLTwmpAIcF1OuuicsEEndpyFrMOicuSh5BfDIOGreg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

5、登陆 vCenter，会发现多了一个网络与安全的按钮(如看不到，稍等片刻再重新登陆 vCenter)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvcOk11JZIC8toLicAW3XqL5lRBmTWtpmiavTnjmO7Sma4sRia6Yl08voyw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

6、在“安装和升级”中进行主机准备(即安装NSX数据层面的组件)，主机准备是以集群为单位进行的。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvcs0nde7OPMBMI8tYia9ag0YsumYhSjvCGibNBsjyVnjOgFibicLFfxC6Xg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhviaTXRYibgYnCrKa7tQzC5gWxq571jyZYmxQzOF075JxjJzbDibXyUfPYA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvzf6ib0goEX6ScBlDHRp0oJuGBtUaIyw01KofF1TgFKAW3gUc1PW3GSQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

安装完毕，看到防火墙状态为“已启用”，可以开始测试 NSX 分布式防火墙了！

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvIUssknY3Ru4rDBsRpd63pWfmgX00CGArjtHGguaiaicodVXJp5fWpSOw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

------

我们先建立一个简单的防火墙规则，测试 NSX 能够实现同网段虚拟机的网络隔离。

点击防火墙，新建名为“同段VM隔离”的**区域**。

**区域是 NSX 中比较好用的功能，如下图所示，其图标是文件夹，也就是代表它的作用类似于我们电脑的文件夹，用于区分以及存放东西。在最终使用NSX防火墙时，可以\**按照部门或者业务组来创建区域**，在区域内中添加与部门或业务组有关系的安全规则，这样未来管理运维会很清晰。*

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvNBdl2icYqaibRVmaoR8wxvY4SZDGgicIqgeGWdqicicfymAYoo36t5t47nQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

同传统防火墙规则一样，NSX 防火墙规则是按照由上至下的顺序执行的。因此创建区域时需要确定好其位置。

**区域中有三个高级选项：*

*1、在源中启用身份认证是 NSX 6.4 的新功能，其适用场景是 Horizon RDSH或者 XenApp(使用vSphere作为底层时)环境，其功能是实现共享桌面环境下per**用户Session的访问控制，未来会有章节详细介绍这块内容。*

*2、启用 TCP 严格策略也是 NSX 6.4 的新功能。NSX 6.4 起支持简单的 7 层防护，目前支持识别 50 多种常见应用。*

*3、支持无状态防火墙模式。*

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhv81iaJrGibfXWQgYFpkX8fpkEDWibrHUJ0qelDHAq9vU2WdevF1MdHAm1Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

创建完区域后，下一步是在区域内创建防火墙规则：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvRHibKwWY4RSgDrPpz2Ooh4Pp6SsKichXaxZy65xiab2oq4ovnPuLe9RBA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

我们设置规则的**源**为虚拟机“win7-1”，**目标**为“Ubuntu-2”，操作为**阻止**

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvEYkSfGNcGdnOibt9HJQhTUYBeyic3eHD85T0ZmR7dhGwIsHnPtibFKuBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvcicCsQIfvlUjhNvSOWpqO8MqXHowH5joIHJLk6LuHumzUqtxp6dW5ew/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvKicliae6nFKgpzsKc9JHaqSFvsAYMuJVINrjQOs9MTfK024Z5RuSiapaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在发布之前，在Win7-1 上长 ping Ubuntu-2，网络可达

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvoL4sdicsg0ia8wy5zwJbskKxQB2c63AHETibPYbiciaYyyic3QFkngelESwQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

返回 vCenter 点击“发布更改”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvr2VdASvxK2sTAR3dImklH4saj2wNRDkibDqI1eib9Gxqkz3HvQlBJp0A/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

虚拟机之间的通信中断，基础的防火墙功能验证成功！

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvmU3ZsZibMKcRW3NP6hI7JA5fvNHictLyswL6sdV91Ee2mZfbCdFl8yUA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

------

**在 NSX 中，几乎所有与虚拟机有关的信息都可以用来建立防火墙规则**，刚刚演示的是直接指定虚拟机为源和目标，下面我们尝试利用“端口组”来设置防火墙规则，**实现同网段任意虚拟机到任意虚拟机不能互访**。

**端口组是 vSphere 环境下一个概念，*[*第二篇*](https://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483712&idx=1&sn=65b665c274fbcf16fb389c4b198b6410&chksm=f982716ecef5f878492752148fba45c5e68cd60d90c8185999494449f6a2be9c5bfc66a0d71e&scene=21#wechat_redirect)*文章介绍过，一般与端口组相关的是\**一组同网段的虚拟机**。*

删掉刚才的策略，添加新防火墙策略，将防火墙的源和目标均设置为VM Network 端口组。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvh3VJiaXqxzJ3Z1JEMiarCNPicicgceXHPp4yaA51icF37gypEgLNOhibQ8qQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvibz0F9s0OZKywiaAKY3DXadIZ6ic6Xr4SHIdGs38Cmj4xyZYHrbftTicIQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvXBzZpyNqvogicYUeXfdib5K2jueia6f5txtTH3MANqPIGel9w3seXmYqg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在下发防火墙策略前，虚拟机之间访问没有问题。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvuscLmOcxkxpicNcI0jAsiaBJ3dBoibuqiazceUx5aJTNn8dUk46c4NlFlw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

下发防火墙策略后，虚拟机之间通信中断。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMXyyIhgGT6SpTCeWibTdqhvOM1XFkhZSThZuiabaafiaQ5FvBObCESsLoicPAtZRxcSrgcicFswTxuYSA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

------

在上述测试中，NSX 防火墙规则均是用一些静态的对象配置的，相比传统防火墙基于IP五元组的配置方式，**NSX 基于对象的安全策略可读性更高**，但在大型环境中，尤其是云计算平台中，使用静态对象的安全策略很难满足业务有序或者无序的变化，因此 NSX 引入了“**安全组**”这一概念，**使用静态对象与动态对象结合的方式来****描述一组有共性的业务**，真正做到安全与业务组挂钩。下一篇，我们会详细讲一讲安全组以及围绕安全组的策略制定。

上一篇提到，NSX 使用基于对象的安全规则使得防火墙策略更加容易辨识。但对象始终是静态的东西，对于动态的业务来说，两者之间不一定有恒定的关联关系。因此 NSX 提出了**安全组**这一概念，可以简单理解为：安全组=业务组，或安全组=一组需要执行相同安全策略的虚拟机。

从安全策略制定上来说，会变为下图这样，安全组在中间，之上对应的是业务组，之后关联的是安全规则。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMlbpxvI3AxwWL1bIgeN07vkuHibqqjDAjAjOuErrzlF8cF0jbCT1VlWw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

这样做有一定的好处，举个例子：下面的**Finance部门的用户**和**Finance部门服务器****都属于可能会变动**的，如果使用静态的安全规则，如果用户和服务器有任何变动，安全规则需要手动修改。

如果有安全组作为“容器”，**动态**将所有Finance部门用户关联在一个组，将Finance 部门的服务器关联到另一个组，针对两个安全组去做安全规则，未来用户再怎么变，**安全规则是不用变的**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMeJDxghkeqibqdgGZseRg9R9TNRhpIZccpVLicMawiav858qlrelYU3Nkw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

为了能更精确地描述出一组业务，安全组包含多种匹配规则：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMp8JC047EaGoOyeJsuqahQQkax3BG5XNwWaBiaIgdzmGwV5IWKJVWEXA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

举个例子：

将所有与部门 Finance 相关的 CentOS 虚拟机加到一个组。

安全组配置方式：

动态包含： **虚拟机名称包含 Fin ****且 ****计算机操作系统名称包含 CentOS **

静态包含：空

静态排除：空

对应在 NSX 中配置方法：

打开NSX配置>安全性>服务编排>安全组，点击“安全组”按钮

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMlrF2vj0n1N4SiaOVhZ33Ps5qzI7MaTfGgsACx2k3faTgpm14d1LqV5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

安全组名称为：Finance CentOS 虚拟机

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMic7icazmgERtn39CqOrOHGmlhtG6t5rbdvjYVHbApx3ke1vztpARnutg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

动态成员规则第一条：** 匹配虚拟机名称包含Fin；**

动态成员规则第二条： **匹配虚拟机操作系统名称包含 CentOS；**

两条规则需要同时匹配(**全部匹配**)。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMcAdfjGmLib0HeaOfUuLwg1fVJ9MzcwceLiaDgHy3xLBXw9xiaIyhwljqQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1)

其他选项无需配置，直接点击完成。

返回查看安全组，已经关联了所有 Finance 的 CentOS 虚拟机。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMpwRvXFTTpiaX8n9TDgsoLtQZEjv74dBGso4UunfVpEbibewx0LqddTlw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

创建好安全组后，可以直接返回防火墙，使用安全组去建立防火墙规则：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMcSooZdKmalxW6KExtkc8nzjfLgCPfkgKU6LFBGFdlExh2bjCjxWFJA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

------

NSX 除了可以将虚拟机加组，也**可以将多条安全规则设为组**，在 NSX 中，这个功能叫“ 安全策略”。

使用安全策略，有以下功能和优点：

1、将多条安全规则（例如防病毒规则、DFW防火墙规则、第三方防火墙规则）嵌套在一起，应用在一个组上；

2、将安全策略做成组后，可以反复套用给各种业务；

3、替代传统防火墙策略设置规则，直接将安全策略应用给业务组。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMYWCiaPWyB9OOGCUPYhuiabf1iaTH7yiaPtU1z6qILmOksa5SHqP5qt3Vjw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

个人觉得，最实用的功能是1和2，使用3来创建安全规则会增大安全策略的理解难度。

NSX 现在已经集成了 EPSec 和 NetX 两个接口，分别用来集成第三方防病毒产品和NGFW产品。而**集成各种安全产品的核心就是****安全组和安全策略**。

以 NSX 集成无代理防病毒软件为例，在 NSX 中，操作步骤是：

- 设置防病毒安全组，将需要进行安全防护的虚拟机加到该组（使用上面介绍的各种动态、静态匹配规则）；

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFM8MHibmTzzBY7C3djtDoFm6bCHm5YTQS2FjV1icVlRDibNxakpXsoF8RicQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

- 设置安全策略，在安全策略中配置 Guest Introspection 服务，将流量应用给第三方安全产品；

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFM7FKcdvzECKGWqbjhOCGdiav0YZIlGq8n06icqfckhyEagY1IkWALPTRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

- 将安全策略应用给安全组

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMLxk5QSW3MCjMpm0SsxwsxPN9ciaZx8hmVI49FmTyErbibc18aWcKuu9A/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMGSdx0Uou6zlXK4tHBveolx4FRvoQ2L2tvotcJVjTdGgq6brlfE3iaNw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

------

为了让安全组与各种安全规则的关联关系更清晰，NSX 里多了个**“画布 Canvas” **功能。**画布可以围绕着安全组，展示与安全组相关的安全标记、虚拟机、安全策略、Guest Introspection规则、防火墙规则、NGFW 规则**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMIV6se4vicbiao8wic1r685r7GOkE7ytdxnic8LXqLMrFGDeXrlt05sSXDQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

------

在以前的文章中，介绍过 NSX 和杀毒软件联动做安全，而要实现这一功能，需要介绍 NSX 的另外一个组件：**安全标记**。

前两篇文章我们介绍了 NSX 防火墙的创建及使用，而这些规则都是基于 NSX 和 vCenter 的对象。**而在有些时候，安全规则不能直接附加给基于 vCenter 的安全组**。比如：

1. 安全规则和 vCenter 的目录结构不能匹配；
2. 基于名称的安全组可能因为名称被误修改而失效（尤其是有多个 vCenter 管理员时）；
3. 第三方安全解决方案和 NSX 结合做安全联动；
4. 第三方云管理平台和 NSX 结合去做安全。

使用安全标记后，架构会变成这样子，安全标记作为安全组的一个中间角色使用：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMqYaNu1qY0MEUGEZ39FcSlzJCuBibic2j4FjiaV5hWrZ9nuick2LNkpkIvQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

如果要实现下图中的安全联动，则步骤为：

- 一旦防病毒软件查出虚拟机有病毒，为其打上 ANTI_VIRUS.VirusFound 的安全标记。
- 基于安全标记预先配置安全组“隔离”
- 设置安全策略，规定虚拟机只能访问安全工具，不能访问其他任意网络
- 将安全组与安全策略做关联

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMZm2QLGCAm34uhUrmicEB3wicnqPF9Oam4iaPQf6bMPlSp9KC7GV3PnUfA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMF4Ilaw5csPqUmkY1mc7x1L3k4vMy69icKxWwXhGJ0ejSuzCxp4M3AIw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

------

小小的总结：

以上两篇文章由浅入深地介绍了围绕着 NSX 防火墙的各种组件，组件之间配合可以加快安全规则的制定，让安全规则更加易读，但缺点是关联关系会变得复杂。

在生产环境中，需要熟悉每个组件的功能和优势，在适宜的时候去使用，安全规则的制定没有最佳实践，但是有一个统一的目标：**管理方便、足够安全**。希望两篇文章只是个引子，帮助大家更好了解 NSX 带来的一系列变化，在不变与变化中取舍。

前两篇介绍了 NSX 微分段的架构以及使用，NSX 微分段可以满足大部分客户对于虚拟机网络访问的控制，在访问控制之外，**NSX 也提供了一些增强的安全功能以及分析功能，帮助用户更好地使用 NSX**。

01

—

Spoofguard

在任何一个企业中，IT 管理员总是试图做到“可控”，尤其是像 IP 地址这样重要的资源，在物理网络中，管理员可能要求所有终端使用静态IP地址，并在对应的二层交换机上启用地址绑定功能，防止设备私接、防止IP地址篡改，甚至防止 ARP 欺骗攻击。

**ARP 欺骗是局域网中常见的一种恶意攻击，通常可以用来嗅探其他同网段主机的流量，或者导致其他主机无法正常上网。其原理非常简单：发送伪造的 ARP 广播包，使得被攻击主机认为其网关为攻击者，然后将所有流量发送给攻击者。在传统网络内，只能使用交换机的 ARP inspection功能才能避免此攻击。*

**在数据中心内，这样的需求依然存在**。

NSX 自带了一个名为 Spoofguard 的功能，其作用就是实现虚拟化中虚拟机 IP 地址到 MAC 地址/网卡的绑定，并阻止合法的 ARP 包通过，简言之，**可以实现防 IP 篡改以及 ARP 欺骗攻击**。

下面来做个简单的实验，实验环境中，有两台在同网段的 WindowsXP（请忽略操作系统类型..）测试机，左侧 XP 正常可以访问互联网的 223.5.5.5 ，而如果在另一台攻击者机器上运行“局域网终结者”工具，左侧机器网络会立即出现问题：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CmgMoh8QKKNLejcUWNP7015WnCUtT1kR2ccBtzVSSfqpBctCic1PJ8iaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

返回 NSX 网络与安全配置界面，点击左侧的 Spoofguard，默认功能是被关闭的

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CQh95wFgmWW04WGf5ORCfohMYt8UQEPb8OHmlWTntB6IQQoM3boG7fw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

点击编辑按钮，勾选 Spoofguard 为已启用。

操作模式有两种：可以选择信任虚拟机用的第一个 IP，或者手工检查所有配置的 IP。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CtZwOy7P94XDdTrsBGgibEQA9Iiafm9wMm8E4MKfGO3maePfCK9lLTKdw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

根据需求配置以上选项，点击完成即可。

返回测试机，可以看到左侧虚拟机网络立刻恢复正常，**右侧虚拟机的 ARP 欺骗包完全不会影响左侧虚拟机**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CRiakqibK1lA01KtmabhJzdnzDEU0IOYgyG2043LRw9pcTvmSPgRLzQcQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在 NSX Spoofguard 界面中，可以看到两个 XP 对应的 MAC 地址以及审批通过的 IP 地址。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CoFpUTudpVXbSCrMK4urJNtjzMvxZTOEmFSapD391s0OX1eRACzhV2w/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

接着我们测试手动修改 XP-1 的 IP 地址，观察其网络通信情况：

**修改完 IP 地址后，任何访问均失败，包括到其网关的访问**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CYfPElup47rVBiaNRcyQvdPcniamRUYgAtmyDLhymD9CqZyocuEjCFcKA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CLvHHcgCVia50KMkI0WXL1mHBohEXmpoJ7BF98cx7FUp4gx1jXrCHWeA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

再回到 NSX 配置界面，在“非活动虚拟网卡”中，能看到 XP-1 的批准 IP 与实际 IP 不符。

此时解决办法只能是将 XP-1 的 IP 地址改回 10.10.50.101，或者管理员将 10.10.50.105 添加为批准 IP。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CanE38ZRBM6eXMyKF1zcMAC2FF2Ficic4MbtP9Nh2ojgWPSlr10oHX6ew/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

测试将 IP 地址改回批准的 IP 后，网络立即恢复正常。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0Cx3SicfQdDjHQkOoDdEkNG5oljdwibPjyG0iaxlCk4aj3DRAXZzu00D6zA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

Spoofguard 功能就是这样的简单直接，而在有些时候却能避免很大的故障发生。

曾经遇到过一个 CASE，客户的邮件系统故障，最终发现是网络原因，再具体点，是某公司搭建自己的业务系统时，配置的 IP 和邮件系统冲突了。出错只在一刹那，而排查整整花了半天。相信在启用 Spoofguard 之后，这样的问题不会存在。

02

—

Flow Monitoring

NSX 的防火墙很好用，但最终，也是需要安全管理员来根据业务互访关系去创建这些策略；在策略创建完成后，也需要对流量进行可视化的监控，或者将虚拟化的流量发送给更加专业的流量监控分析工具。

**NSX 自带了一个小工具流量监控(Flow Monitoring)，可以帮助实现以上需求**。

NSX 流量监控集流量收集、展示与分析为一体，具体有以下功能：

- 流量统计：展示流量的概况，流量排行榜，流量放行/阻止情况

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CCuGxB5TibRLTH5OrEe8xGvFyXCZ9FQcLZNvQPTficFjSK9ic8JsodwoRA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

- 按服务(端口)展示流量的明细，并提供了快捷的防火墙规则创建按钮。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CLxiaJKOg2uImNSAqpTNycqiaV2l5bGxqnF08DsLoFyGx5vZy8V1mYLBw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

- 虚拟机接口实时流量监控

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CJk8VqskribpPwWiaerojHf0ibLtmXVDdRK5T8jrSLibIG5eL0vgI6AQ62A/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

- 通过 IPFix (Netflow v9)协议将流量统计信息发送给第三方收集器

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0Cuas937t6uXib2VEXuicHZPWUibu3yLtruhBb1pnUwnWtlgFuWkdZnkuWw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

- 应用程序规则管理器：收集与业务组相关的流量，帮助管理员根据业务互访关系创建防火墙策略

下面来演示一下应用程序规则管理器如何使用：

打开应用程序规则管理器，点击启用新会话

![img](https://mmbiz.qpic.cn/mmbiz_jpg/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CibRDNtXRQK1QichdYmNdSTCblfa6IwrlIPF4az7uwfWcLiaicichPUreDsQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1)

设置会话名称，并将要收集流量的虚拟机加入该组

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CepeCs5piasg3U3N5cXJ9zeHRC0fsVrfBPcjeBqOHQoSDE4B58fW4H6Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

点击确定自动开始收集流量

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CXa6LrKLeiaKwRcuPMCXKv92KkDUcAkNiaKfLKFctb5QWIdGupzUQwy4Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

待收集完成后(收集时间越长数据越准确)，点击“停止”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CEjAjwianxc3edQDibJUxyqCtAPeUWLyTrvWV9R3WTQPt4Wd9PqIkibicJw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

点击“分析”，**流量会由 IP 五元组变为 NSX 可以识别的对象**，例如XX虚拟机、XX服务

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CN8Z4vWFAhQ0SwGU95QyrzdiayboUIIczoJiaQeRJhTS1XeHVbp08Miafg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0C6gkaesg1SPEr54uwCA7IIrDyuMA56CnaHAzMao9OYNpMaOXT88KjsA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

选中分析后的流量，点击“创建防火墙规则”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CozDBibzXYKx24G2iag8cuQmiamry3nv1wR2anuAvPwffVy0Xd7g3Uc0jw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CSicv1N8lhxeLvR4Mho4quLZicoHnVWIhRJAE7nzwNKCAwZPj7DdHicX6Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

如果觉得以上规则不完全贴切环境，可以稍作修改。例如将源改为空（即代表任意），禁止 Ping 操作。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0C0uhibeNrOMlnBm3dciaYRDbkyEtBwuWXMdjltc1dZxfus8nibSw0QvQbA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

点击确定后，防火墙规则会被自动创建，但尚未发布

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0Cyp96wTDAeJhWsibVLWHY6RiaeN5vP5JfKvUf4VmXW7Ad1FYC6jeviciaeQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

点击“发布”，会要求选择防火墙区域的存放位置，我们选择放在最顶端，点击确定

**防火墙区域的介绍请参见“*[*NSX从入门到精通(6)：NSX 微分段架构、组件及实践 Part1*](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483815&idx=1&sn=8adf10bffb7e7f2846abd0635d4ca52f&chksm=f9827189cef5f89facc7cc788a53191ed1abe662291ac8555754268f4eb038103403bc209cac&scene=21#wechat_redirect)*”*

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0ClsR00yich7uiaLRHy1fn6R8kl3iaL2FYShAQibuKy0GaO30PubO1CbdtgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

打开 NSX 防火墙配置界面，可以看到已经通过应用程序规则管理器创建了防火墙规则。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CrkmXeRAdyk1hNO3dXzDbfUyODdLAayMAa1yzBzmvuN3Xb4NnSWNVYQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

03

—

vRealize Network Insight

之 Plan Security

在第二部分，介绍了 NSX 自带的流量分析工具 Flow Monitoring，在简单监控排错时功能足够，而在稍大规模环境下使用则有些吃力，尤其是在长时间流量收集上。

因此 VMware 提供了一款更专业的流量收集、分析、排错工具 **vRealize Network Insight，简称 vRNI**，属于 NSX 家族的一员。 

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0C5k5hCRYbFBjiaiaGBUuKfKehXJK5hOA4TPVHDYjnDIYc5TqYnMzZ9oAg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

概括来说 vRNI 有三大功能，在和 NSX 防火墙搭配使用时，只需要使用第一个功能：**Plan Security**

****

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CbHvsH7jNxgWtKib4YyTNu5XtJu4t6qYicswicBxarDpVHsV5ZdDyKeWcw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

Plan Security 的功能和 Application Rule Manager 类似，不同的是 Plan Security 会**收集所有虚拟化的流量信息**，然后再过滤出需要使用的信息。

点击 Plan Security，选择展示过去1天的流量信息

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CVQ008HHqyJCkibdu0Ty21vcics7QLNsU2DLvuqMRwLzlWGA2WmKicPnkg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

左侧会按照“VLAN/网段/集群/文件夹/虚拟机/安全组”等多个属性展现流量关系。

右侧是一些汇总的统计信息

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CeX7icu6Sxb1AjgsLWlPiaAtM95gibKOd5CzIxobFOJq8CSBTYtx1K468A/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

如果点击任意的扇形块，可以展示出与这个块相关的流量信息

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CXuibRUsSibDxftJXcWpPCUFx5xe0PhhzZBneO7nDZMsz6wRPQFd7B5UA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0CDO6Xav20KQ1BUHs7qXJ5Y5FJy4EbzQx2hv4sNCktmDIoaia2B2mrC8g/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

系统会自动根据历史收集到的流信息，推荐一组防火墙规则，管理员则可以依靠这个去 NSX 防火墙中设置规则。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNEaFtcKaOT0jY6rOM5gj0C2Mc6OMyErVVmn1X3CY7icAPDUKu791uNt4aic7mJzCliao34OlggbDkfA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

vRNI 的其他功能，未来会有专门的篇章介绍

------

回顾一下，本文着重介绍了 NSX 微分段相关的三个功能：

- **Spoofguard**：地址防篡改、防 ARP 欺骗攻击


- **Flow Monitoring**：简单的流量收集、监控，规划微分段安全策略


- **vRNI - Plan Security** ：更加专业的流量统计、分析、规划微分段安全策略

------

文章最后，附上几个演示视频（原视频均可在“[NSX 相关的资料，全部放在了这里](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483867&idx=1&sn=770834662ff3178db3cb26fb8e2b90be&chksm=f98271f5cef5f8e3054ff659cca2f107f36d9d679c0941b4f16128d7d12699e152e663c4c5c0&scene=21#wechat_redirect)”中找到）：

1、Spoofguard 演示

2、NSX TraceFlow 及 Flow Monitoring 演示

前面几篇文章依次介绍了 NSX 微分段的技术实现、基本的安装和使用功能，这篇介绍下 NSX 微分段的一些使用场景，希望能够抛砖引玉，让大家发现 NSX 微分段的更多潜力。

01

—

阻断

在[NSX从入门到精通(4)：NSX 带来的安全转型-上篇](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483774&idx=1&sn=498cb1b994cfd5a9d6ea6b65f487371f&chksm=f9827150cef5f846a26aca25757819974dec1c5176f847eac83928b1919b902c6f59c3602bb5&scene=21#wechat_redirect)中，提到了一些企业可能遇到的安全风险，其中有一项是攻击者利用应用的漏洞来攻入企业内网中，**而在传统的安全下，内网之内很少会有隔离手段**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05TvpXro0CGH59Dm5e0J7mEOcfWicfFZ05uCuNrXZ241Jv21eccvibM9JQDw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

因为这种架构的缺陷，在2017 WannaCry 爆发时只要企业有一台机器被感染，通常很多内网机器也会受影响，当时全国几乎所有企业都在做一件事：**封端口**！

*下图为360企业安全发布的WannaCry工作原理及临时解决方案

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOxuwswgicokvnhcG9WfazXqU2pTxy07FAv9JbwcDyM7Se9dcXmJSOicK2zQQAmfIUFxOygMJQRbQibQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOxuwswgicokvnhcG9WfazXqdY6ohSVZ1NjOmbZojo39Wib45ADHf5yZvrYZSACiaErWSSxfEfADdPAA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

可能有人注意到 WannaCry 的一个感染流程是：**扫描其他漏洞主机。**很多恶意攻击的基础原理就是先进行**端口扫描**，端口扫描可以获得两个信息：**开放端口(应用程序)和操作系统类型及版本**。再利用这些信息**查询相关的操作系统或应用程序漏洞**，再利用漏洞进行攻击。

而以上的工具和资源，对于任意一个懂计算机的人都是可以轻易获取到的。

*下图为通过nmap扫描一台Windows机器的结果

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOxuwswgicokvnhcG9WfazXq6L1dFGTcB2k93R6wTezIDTyhhSRWDfKy6lNeUQksaWhMCPheVg1spg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

操作系统和应用的漏洞是不可避免的，是软件就有Bug，我们能做的，就是阻止恶意扫描操作，阻止任何不合规的访问。

如果要在 NSX 环境中进行 WannaCry 的防护，只需要创建几条防火墙策略，点击立即发布即可。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOxuwswgicokvnhcG9WfazXqSqKccyHGWfwABiaQFOAiaaXT74iaEnLD7lSXGNHsI1w9frSrZ0P90IveA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

*使用上述 NSX 防火墙策略后，再次通过nmap扫描结果

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOxuwswgicokvnhcG9WfazXqj190pTTm931z2fWt8ibV2P8vt3SZZDIHsn7aFgntbAozAwriaSsycLMA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

通过简单的 NSX 防火墙策略，原本操作系统开放的一些端口被关闭，恶意软件则无法再利用这些端口去进行攻击。

02

—

隔离

第一个场景提到 NSX 可以做一些阻挡的操作，不过这样针对性的阻止只能是事后的，最稳固的办法，是**一开始就了解该放行哪些端口，剩下的所有访问全部拒绝**。

一般的Web应用架构分为三层：Web层、Application层和DB层。

*Web层(表现层)直接面向最终用户，提供接收用户输入并展示数据的功能；*

*Application层(业务逻辑层)，处理Web层用户的请求并进行处理，需要和DB层连接调取数据；*

*DB层(数据层)，存储数据，一般运行数据库。*

这样的分层架构有着扩展性强、方便维护、安全等特点。通常的访问关系是这样子：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOxuwswgicokvnhcG9WfazXqBwPA32pTkDOgZZxCurw8HbZuUCI0hTOfyXicZ0xNVs5hoaq3B1RxG2g/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

外部可以访问 Web 层的 80/443 端口，Web 层访问 App 层的指定端口，App 层访问 DB 层的指定端口，其他的访问都必须阻止。

**传统的安全可以做到不同层之间的访问控制(虽然网络架构会较复杂，难以维护)，但做不到同层之间的安全隔离**。

而 NSX 的防火墙**可以做到任意源到任意目的的访问控制，弥补这部分空缺。**

------

如果企业内有很多部门，而部门和部门之间没有任何业务逻辑关系，那部门之间也需要做到隔离，对于 NSX 来说也不是什么难题，**只需根据部门业务关键词(例如Fin和HR)，配置一条 Fin 虚拟机到 HR 虚拟机不能相互访问的策略即可**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeOxuwswgicokvnhcG9WfazXqtQ3vvOEWicXAPPoqsssNVwaz2YlLsibGiaPCPqKeTZyG0nrXWricLMtedg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

------

或许正是因为察觉到了目前虚拟化的安全风险，在2017年国家颁布的**《信息安全技术网络安全等级保护基本要求 第2部分：云计算安全扩展要求》**中，明确要求云租户能够设置不同虚拟机之间的访问控制策略，且要求安全策略能跟随虚拟机进行迁移。

NSX 正是满足等保 2.0 云计算安全扩展要求的最佳解决方案。

下面为 NSX 动态安全组和防火墙的演示视频，时长13分钟：

03

—

安全联动

NSX 提供多种接口供第三方安全产品集成，通过一些标准的组件（如安全标记，详见[NSX从入门到精通(7)：NSX 微分段架构、组件及实践 Part2](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483851&idx=1&sn=f54ddb78543d718bf72b79663ec0f6c5&chksm=f98271e5cef5f8f3515d06862c36472276a12c9636aec96e348a5eb0661019e87a79b153fa89&scene=21#wechat_redirect)）实现安全产品间的联动。

NSX 介于虚拟机和第三方安全产品之间，提供安全的标准接口，提供安全融合的接口。 

在具体应用上，有以下使用场景：

**1、防病毒软件与 NSX 实现的无代理防病毒**

NSX 之前，虚拟化防病毒有两种存在形式：每个虚拟机都部署防病毒代理、或者利用 VMware 提供的vShield Endpoint进行无代理防病毒。

从 vSphere6.5 以后，vShield Endpoint 已经被 NSX 替代，未来 vSphere 环境的无代理防病毒解决方案都需要通过 NSX 来实现。

集成之后一个很大的好处是能够以安全标记为中心，**做到防病毒软件和 NSX 防火墙的联动，或者说安全自动化：**

****

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePX5RpqyV6gvHNWzT1vXBFMZm2QLGCAm34uhUrmicEB3wicnqPF9Oam4iaPQf6bMPlSp9KC7GV3PnUfA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

**2、****NGFW、IPS、IDS 等与 NSX 集成**

NSX 具备2~4层防火墙功能，目前6.4及以后版本支持7层应用识别，而在一个完整的安全解决方案中，通常还需要NGFW、IPS、WAF等组件。

NSX 提供NetX接口，可以让多个厂商的安全产品以虚拟串联的形式集成起来，虚拟机的流量可以依次通过各个厂商的安全设备进行扫描和分析，借助安全标记，也可以实现安全的联动。

例如， IPS/IDS 通常会扫描经过其的数据包，通过一些特征库判断是不是存在攻击，而判断结果可能为“确认为攻击”、“疑似威胁”和“正常”。

对于确认的攻击可以直接进行阻拦；对正常的包进行放行；而疑似威胁的包可以临时让 NSX 防火墙进行隔离，例如禁止访问互联网。

接着 IPS/IDS 将疑似威胁的包发回安全产品的云端沙箱进行分析，分析完成后 IPS/IDS 再根据分析结果进行更加确切的阻止或放行操作。

**3、等保 2.0 的安全要求**

在等保2.0中除了对虚拟机网络访问控制有要求外，也提到了虚拟机之间、虚拟机到物理机的流量识别和监控，在[NSX从入门到精通(8)：NSX 微分段的周边](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483907&idx=1&sn=a270e839478beabac7c4abb01f0a095f&chksm=f982722dcef5fb3bae6425aec4d14ccd4ca95e4de92b5447fa01a27420b2cb71b87b57fce354&scene=21#wechat_redirect)第二、三部分均介绍了如何在 NSX 环境下做到流量可视化；

等保2.0还要求平台提供开放接口，允许接入第三方安全产品，这其实就是将上述两个安全产品结合的场景列为了等保建设的必需项。

04

—

多数据中心的统一安全

**多数据中心的统一资源池**一直是近几年很多企业的 IT 建设方向，尤其是有多个分支的企业、多个校区的高校、多个分院的医院等等。

多中心统一资源池建设有以下优点：

1、统一管理，减少管理成本；

2、资源池化，灵活进行资源调度，提高各个中心计算资源的利用率，减少硬件投资，降低整体运营成本；

3、灾备和双活，在多地存储重要数据，避免自然灾难带来的数据丢失，尽量减少重大事故时重要业务的中断时间；

4、方便在其之上建立云平台，云平台只负责资源的分发和业务自动置备，而无需关心底层架构的复杂性。

**多中心资源池的建设基础是虚拟化**，虚拟化能够排除基础架构底层硬件的差异性，为虚拟机提供标准的计算和存储资源，进而**实现业务在多中心不同的硬件架构上任意移动**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePnMzqBM8mWtmOIUtAKIv2s1d4thKgH3QOleHiaWwlP9jTEBgEd6M7ibd3ibsZic6f0jIPOO7GQGe9JSg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在上述架构中，虚拟机跨中心在线迁移技术已经很成熟，但有个小小的问题，虚拟机在线迁移后也需要对外提供服务，而这时候，要求**无论虚拟机到哪里，网络和安全就需要跟随到哪里**。

以两个数据中心为例，为了保证任意一个中心的故障不会导致其他中心无法管理，需要在每个数据中心部署一套 vCenter，每个 vCenter 只管理本地的主机和虚拟机；为了实现统一管理，需要配置 vCenter 增强链接模式将多个 vCenter 链接在一起，登陆任意一个 vCenter 都可以统一管理所有资源。

*下图为 vSphere + SRM 实现双数据中心灾备的架构图，每个中心都有独立的管理组件存在，对应组件之间配置了关联

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePwMDDwCLiaXZWMfU8Dqz4axEQ8Dqr4wZhPcXoiacHLh115G2yc1WC60eYRF8FFtwXNsOZQ2sY2qWAQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

*vSphere 6.7之后，增强链接模式的架构略有变化，详见：[vSphere 6.7 的新 PSC 对我的环境意味着什么？](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483666&idx=1&sn=b20a6abb5b97cc241f4dbfde8a85f3bc&chksm=f982713ccef5f82a421a1f1bccfda0cdc8a43bacaf5e78f2778ee150f060ad97917211dc3a9a&scene=21#wechat_redirect)

一个 vCenter 就是一个管理域，在 NSX-v 的架构下，每个 vCenter 需要对应一个 NSX manager，意味着多个 vCenter 时，会有多个 NSX manager，也就意味着有多个 NSX 环境。

NSX 从 6.2 版本开始发布了 Cross vCenter NSX 这一功能，允许将多个 NSX manager 关联起来，实现统一的安全。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePnMzqBM8mWtmOIUtAKIv2sBNkwFLRBjwicGbSibmN5Kial9XPpF6Piaqlod1ucJey7zcl6cZIlv9bRTg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

如上图，Cross-VC NSX 有一个 Primary NSX Manager 节点，一般运行在主数据中心，负责全局网络和安全策略的配置及下发。其他站点的 NSX manager 均为 Secondary 节点，通过 Universal Synchronization Service 服务和 Primary 节点同步配置信息。

在 Cross-VC  NSX 之后，虚拟化环境里会有两种防火墙，一种是本地的 分布式防火墙 DFW，另外一种是全局分布式防火墙 UDFW（Universal DFW）。

UDFW 即是可以跨越多个数据中心，多个 vCenter 管理域，多个 NSX manager 的**全局统一防火墙**。

在功能上 UDFW 与 DFW 无任何差别，策略统一配置而分布式数据处理，能保证虚拟机迁移到哪里，安全策略跟随到哪里。

在配置上最新的 NSX 支持利用以下元素创建全局防火墙策略：

- 全局 IP 地址组
- 全局 MAC 地址组
- 全局动态安全组（虚拟机名称、IP/MAC 地址组、安全标记）

防火墙具体的配置和使用可以参照 [NSX从入门到精通(6)：NSX 微分段架构、组件及实践 Part1](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483815&idx=1&sn=8adf10bffb7e7f2846abd0635d4ca52f&chksm=f9827189cef5f89facc7cc788a53191ed1abe662291ac8555754268f4eb038103403bc209cac&scene=21#wechat_redirect)及[NSX从入门到精通(7)：NSX 微分段架构、组件及实践 Part2](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483851&idx=1&sn=f54ddb78543d718bf72b79663ec0f6c5&chksm=f98271e5cef5f8f3515d06862c36472276a12c9636aec96e348a5eb0661019e87a79b153fa89&scene=21#wechat_redirect)

*使用 NSX 安全组创建 Universal 防火墙策略，相关对象会有一个小地球的标识

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePnMzqBM8mWtmOIUtAKIv2s1uY7evUjiavjRC2PxOVruDkKA0rPvHBekpoLU0sjsBgeOsxC966x4cA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

除此之外，Cross-VC NSX 在多中心网络上也有很多增强功能，未来的网络部分会详细介绍。

------

以上，便是 NSX 在数据中心安全的一些场景，汇总一下，使用 NSX 可以实现阻断、隔离、安全联动和多数据中心统一安全。

- 阻断：弥补传统安全的空缺，通过微分段实现虚拟网卡级别的最细化的安全管控；
- 隔离：实现任意组即为安全域，一条策略实现域间的隔离；
- 安全联动：开放安全接口，可以和第三方安全产品集成实现安全联动，从各个层面增强数据中心安全；
- 多数据中心统一安全：跨中心实现统一安全管理，保证业务在哪里，安全跟随到哪里。

接下来两篇，会依次介绍 NSX 在 EUC 环境和 vRealize Automation 环境的使用场景。



> 引用一位前辈的话：“在普遍的认知中，VMware 是做服务器虚拟化的，是不做前端的，只存在于数据中心，因此你很难跟你的家人亲属讲清楚 VMware 是做什么的， 而在 VMware 收购 Airwatch 后，VMware 的业务方向似乎在逐渐变化，直到 VMware 收购 VeloCloud，一切变得更加清晰”

- **后端**：后端以软件定义的数据中心为基石，承载任何数据中心的应用，服务于企业用户：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeP7ynxiattmWzNMCB0pYAM79J1TaoKfUcibN3ADcye49aQoASDaE1SiaXicek35wb1fUcZPoFo3AeibPKg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

- **前端**：前端以 Airwatch 为核心，实现用户端设备的统一管理，提供统一的服务界面和一致的应用体验：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeP7ynxiattmWzNMCB0pYAM79IDsicmfInYEnMxXjohLPO1XrQTBicI05yWdib9wPibs7KNgK9YMxZ8XDFg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

- **中间**：中间，是数据中心到最终用户的通道，这个通道即是网络，而为了保证传输的数据不被窃取，需要再加上安全。NSX 便是实现通道的关键。在早期 NSX 只是一个产品，名为 NSX for vSphere，也是目前 NSX 从入门到精通主讲的产品，而现在 NSX 已经成为一个家族，逐渐从数据中心走向了各个角落：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeP7ynxiattmWzNMCB0pYAM79ibUWQBHj7UAiaLGoGcWn0NQtJBS1rfxD3T4w576GTvXEibV8ADwxhrpng/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeP7ynxiattmWzNMCB0pYAM79c4vt3gUdFs3yH0PNeib7duGsW5ttphYM5FweuU3ibzq2N2qZXVicrFryg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

01

—

桌面虚拟化的网络安全

首先请大家思考一个问题，桌面虚拟化和云计算有关系吗？

个人觉得，两者最相同之处即：终极目标都是**服务**。

同是服务，又有些许差别：桌面虚拟化为企业各个员工提供操作系统（OS）、应用（Application）、内容（Content）和数据（Data）服务；云计算为用户提供基础架构（IaaS）、平台（PaaS）、软件（SaaS）、任意即服务（XaaS）。

同是服务，又必须有相似之处：服务的前端是用户，服务的后端是 IT 管理员。用户想要更快捷的**个性化服务**，IT 管理员想要在**合规**的前提下尽力提供**快捷的服务**。

而通常，个性化和合规会有一定冲突，为了解决这种冲突，需要设立一个界限：服务需要满足大部分人的需求，同时做到100%的合规，再预留一定个性化的空间。拿办公环境举例，大部分人需要的办公软件必须提供（例如Office），企业安全合规必须满足，而留有 Portal 或者权限允许用户自定义使用部分合规的软件或者工作空间（桌面等个性化配置）。

回到正题，[“](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483774&idx=1&sn=498cb1b994cfd5a9d6ea6b65f487371f&chksm=f9827150cef5f846a26aca25757819974dec1c5176f847eac83928b1919b902c6f59c3602bb5&scene=21#wechat_redirect)[NSX从入门到精通(4)：NSX 带来的安全转型-上篇”](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483774&idx=1&sn=498cb1b994cfd5a9d6ea6b65f487371f&chksm=f9827150cef5f846a26aca25757819974dec1c5176f847eac83928b1919b902c6f59c3602bb5&scene=21#wechat_redirect)一文中讲不同层面的安全时，提到了桌面虚拟化这一概念，VMware 的桌面虚拟化解决方案名为 Horizon View。

桌面虚拟化的核心架构是**分离，**将用户端和桌面服务端做分离，实现用户和服务及数据的松耦合。

分离之后，两个系统相互独立，服务的更新不会影响用户的终端接入设备，不同的终端设备始终能获取到一致的服务。

经典的虚拟桌面架构分为三段：

- 最左是**用户+设备**：用户以部门、工作内容、级别等元素成组，每个组对应的服务不同；设备可以是公司配发的PC、瘦客户机，也可以是自带的笔记本、平板甚至手机；
- 中间是传输**网络**：网络为终端设备和后端服务器的桥梁，服务通过网络递交给最终用户；
- 最右是**用户桌面**：多个桌面操作系统运行在标准的虚拟机基础架构平台上，静态（一一对应）或动态（浮动桌面池）分配给最终用户。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05Tv6q22NX19Flv26cXZ3CSicJ5qIzX2SKnLC0RbkdOgJZqoMS3jvkD3CUA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

使用虚拟桌面有非常多的优势和价值：

- **减少 PC 采购投入，保护资产**。传统企业通常会批量采购很多同款的 PC，批量采购可以降低单价减少整体投入，同款 PC 也可以方便未来的桌面运维。通常为了满足业务需求，也会有一个硬件更新周期，一般为3~5年。桌面虚拟化之后，PC 的作用仅是输入/输出，只要硬件不坏，PC 使用年限可以大大延长（笔者有个鼠标用了10年）；
- **满足用户对于 PC 硬件的个性化需求**，除了统一配发 PC，更多企业愿意为用户提供电脑补贴，让用户自行购买喜欢的 PC，这样可以从硬件层面最大程度满足用户的个性化需求，也可以节省部分统一维护的成本；
- **数据安全**，分离之后，所有用户数据保存在数据中心，在数据中心内部，可以使用多种备份手段保证数据安全；
- **提升桌面在线时间**，在桌面虚拟化架构中，所有桌面都会运行在计算虚拟化之上，企业级硬件+企业级软件+各种高可用功能保证桌面系统稳定运行，服务器的硬件维护也不会影响桌面的使用（详见[NSX从入门到精通(2)：NSX介绍-网络工程师篇](http://mp.weixin.qq.com/s?__biz=MzUxODgwOTkyMQ==&mid=2247483712&idx=1&sn=65b665c274fbcf16fb389c4b198b6410&chksm=f982716ecef5f878492752148fba45c5e68cd60d90c8185999494449f6a2be9c5bfc66a0d71e&scene=21#wechat_redirect)）；
- **更多服务选择**，桌面虚拟化架构有个关键词叫“模板”，每个模板可以区分提供不同的服务内容，例如Windows 7 桌面，或Windows 10 桌面，而这些服务都可以即时提供给用户，实现一套设备多种桌面；
- **标准化服务**，在一个模板内，可以预安装很多应用，提供标准的应用服务；
- **增强安全**，同样，在父模板内可以预安装安全代理、审计工具等组件，做到安全可控、事件可追踪；
- **即时应用交付**，操作系统可以做成模板，应用也可以做成模板。这样的架构可以实现秒级应用安装、秒级应用更新；
- **应用容器**，可以将同一个应用的不同版本封装成“容器”，满足不同业务对于终端软件的兼容性要求（比如XX网银只能使用IE8，而YY网页却必须用IE10才能正常访问）；
- **统一管理**，桌面虚拟化之前，每个 PC 都是独立的存在，还不方便移动，如果出现任何软件、硬件问题，只能是 Helpdesk 到处跑（此处想起了英剧 IT 狂人的片段...）。而桌面虚拟化可以通过一个控制台管理所有“PC”，处理任何系统和软件层面问题。
- **减少运维成本**，统一管理之后，可以实现很多批量的操作，例如打补丁、更新软件、下发安全策略，极大减少管理员工作负载；

不过目前的虚拟桌面方案一直存在一个安全漏洞，即**无网络安全**。

具体来说有两类风险：

1. VDI 桌面之间的非授权访问
2. VDI 桌面到后端服务器的非授权访问

前面多篇文章提到了 NSX 微分段，在 VDI 环境下微分段可以轻易实现多个 VDI 桌面之间的隔离，做到不同部门的安全隔离。同时，NSX 防火墙也可以设置 VDI 桌面到后端服务器/研发环境的访问控制。

如下图所示：一个企业内部有多个部门，也有很多外来驻场开发人员，NSX 可以通过一条策略禁止不同部门之间的互访，再根据用户和业务系统关系，有条件地放行用户到业务的访问。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePnMzqBM8mWtmOIUtAKIv2sN13sxDn7zuDC9Z1om2MlCS967OiaZGsJJNG9TicaSLO1AuiaYyeNyFtcQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在背后的实现上， 以 AD 的安全组为中心，Horizon View 提供桌面池、控制用户权限、可使用的软件等等功能，NSX 实现**针对用户的安全策略，**架构如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeMHJXA0XXliaL4Q2bCVABceiaWbM6elYhLFdTusfUfPKicpIQ1SOGh9iccibW7EBibibSmvRS3rxhiaribJ7iaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

下面是一个 NSX 与桌面虚拟化集成的演示视频，时长7分41秒，里面演示了如何在 NSX 下配置针对用户的访问控制：

------

02

—

共享桌面的网络安全

前面提到的桌面虚拟化通常为**一个用户至少对应一个桌面**。从资源利用上来说，依然是每个用户至少要占用2~4G内存，1~4核CPU（简单办公场景），如果可以**让多个用户共享一个操作系统**，则可以大大提高单个虚拟桌面的利用率，降低整体的计算资源消耗。

微软 Windows Server 的 RDSH（Remote Desktop Session Host） 就是这样的一种解决方案。

VMware 将 Windows Server 的这一项功能无缝地集成到了 Horizon View 中。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNkpZcgqRxfLLk1dZGkggyPZs2Fh1H3ZeDtX2dVdLpTJpNMVU9RW4rOsmdhEooqHREVQXgupHBQ2g/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在 NSX 之前，这种共享式桌面虚拟化架构依然存在章节1中提到的安全隐患。

早期的 NSX 版本不支持共享虚拟桌面下** Per User Session **的安全控制，难点在于 NSX DFW 的生效位置是虚拟机的虚拟网卡，而**在共享虚拟桌面环境下，多个用户共享一个虚拟网卡**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5LejlTR1cjhIlb1teia9yPCpuj61IRo7fYFicdHyFtJiaPhiaeZu1wJW1wA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

**在 NSX 6.4 及之后版本，NSX 可以通过 Guest Introspection 组件区分出同一台虚拟机中不同用户的会话，并针对用户来做网络访问控制。**

下面通过一个实验演示如何进行相关的配置：

实验环境：Windows server 2016 一台，已经安装 RDS 服务，已经加入 Windows 域，两个用户 user1 和 user2 分别属于不同的用户组 IDFW-group1 和 IDFW-group2 。

测试项目：user 1 只可以访问内网的业务，禁止访问互联网；User 2 则可以访问任意网络。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5zDh3nIicF4SZjZ2picKAp19n7OGtO7uSNOic8hibfIDeeqiasYb512McVRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

1、打开 NSX 配置界面，点击“安装和升级”>“服务部署”，部署 Guest Introspection

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5mC4TQfvME0ia5NVsrVpzXZqxYAuKiaGpUDAWlj65mgjkxacOrqu0zM4Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5ia8r7l0c9DusGrCLrf7iamiavduoostw5T29YicjISEkzEtnRBQhicZnX7g/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5RNc4hjnibC8h3nTSLtVeuWuXpnwicj2iblcRPTYqD9pkH4GUGlvibpiaibrg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

使用 IP 地址池给 Guest Introspection 虚拟机分配地址。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5UXRw4S3yuFcFhETrEN9AVjC0qbcHcgYDxOl8f4QFSfBB6NVf1sWE4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5SAAOMnjLwJVz65XXibvsMPLIdqZZDJrNELvX13Kpq9SuHhTaXpRiacvw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5Vd6PtnJrTSic3WMc1ajpmhCJjHG1LaJxtLHLgT9to0L6g7QjvLWghAQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

部署完成后，等待安装状态和服务状态均为绿色状态。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5icjawP8eVXJn14kAuTMNK0iaC91qKXG1yofvTtucX3ibXMxYCZbZMyNgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在NSX>安全性>防火墙中添加区域，名为“RDS server 用户身份防火墙”，勾选“在源中启用身份认证”

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5tbX3BArZe17Lfea19AQvFfPbaibGxoVGODZRiaRyrsEdzs3YmrEZeAow/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

创建如下的防火墙策略：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5rtDUicr5ZicicMVSD7QsPMEpam25iclnRMcujTEXXgMiaebGetibbxHbeP3A/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

使用 user1 登陆 Windows Server 2016，无法访问互联网。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5PtkM20uq5CPUxkUKZR8u7pxDgfF07MbbibZQcEgFL0ibgEt3QHIk5zsQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在 NSX DFW 界面可以看到 IDFW-group1 的源显示为 User1：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5CzwdQlnicbzLqjIiaRPrGVib9daEhdjRDDKVTMibTfgb2NwNXURdbiaKJSQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

使用 user2 登陆Windows Server 2016，可以访问所有网络。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5g4OIzs5LAyN47aqorO2xK0kbNZtRzgnvh2Wb4HQzHTDuFRTvLZQpjA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

在 NSX DFW 界面可以看到 IDFW-group2 的源显示为 User2：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5vl1nqaHiahg04YF8GWWvcWE71r6AIkUreHa3f2iafKZjDvS2pkEynYGA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

这里需要重点提的是，两个用户是同时登陆到 Windows Server 2016的：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5O9gyIZriaGibL64INGenyvhnHSy6bh2LBg93xM7tcUNJeibb2c1FY4iaibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

------

03

—

应用虚拟化的网络安全

与共享桌面虚拟化类似，应用虚拟化可以直接将应用的界面推送给用户使用（而不是推送一个完整的桌面环境），在用户体验上更佳。

例如我使用的是 Mac OS 系统，但是可以通过 Horizon View 打开远程的 IE 浏览器来使用：

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO50SJ2H0ycGCr0KFB3lia57DCBs5BwaEwhbujoe2yC38WY2JIcIQ1hYUQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

VMware 对应的解决方案为 RDSH App，Citrix 对应的解决方案为 XenApp。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5Qn9lafbJeAegowM1t8BlkrzTFRjibXOCRuG9ic0MliapliaOUa1HcxYRoA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

对于 NSX 来说，共享式的桌面虚拟化和应用虚拟化工作原理并无太大差异，因此依然可以根据用户身份来做网络访问控制。

配置方法与章节 2 一致。

------

04

—

移动终端的网络安全

现在越来越多的企业都会有移动办公的需求，为了让远程办公用户可以访问到企业内部的系统，通常有以下三种服务部署模式：

- 直接将服务器部署在公有云，所有用户均可直接访问；
- 服务器部署在企业数据中心，通过 NAT 将网站发布到互联网供员工使用；
- 服务器部署在企业数据中心，用户通过 VPN 接入内网，再通过内网地址访问服务器。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5oSJe7YIAiaiax9U6HAtynGHXByyhVOkmEl5NpRbn1WqN6Vl3upDehtRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

如果去分析安全风险，1 和 2 都直接将服务置于整个互联网，**任何人都能够访问到这些服务**，而这时候不可避免地，有人会去恶意攻击这些服务器，获取企业敏感数据。

而 3 相对之下安全很多，不会将内部服务和数据直接置于互联网，用户需要先进行 VPN 认证才能访问到内部系统。

使用第三种解决方案时，通常需要在远程办公电脑上安装一个 VPN Client，和总部的 VPN Server 建立安全隧道，这样远程办公用户便可以通过安全隧道访问到企业内部的系统。

这时候，问题又来了，一个远程办公电脑上会运行非常多的软件，而且**同时连接了互联网和企业内网****，相当于直接将企业内网环境暴露在了互联网中**，一旦电脑中有恶意软件，会直接影响到整个内网。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5PRabRcAibZ9sNWZkU9vuwuOz66helbn2xTSzkRtxdUG7571DRGh588g/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

解决办法就是再将 VPN 细分，由以前的 Per-Device VPN 变为 Per-App VPN，每个应用一个 VPN 隧道，VMware 的 Airwatch 中便提供了这样的功能。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWePo09JHqo5aRNia2rFoDWnO5ZzsgIR8rxn9Nvot4lEM7WcS94G32ZMicg60iaxG34AeCqXQsINDGc82g/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

Per-App VPN 可以控制终端上的应用能否访问内网，而**不能控制这些应用具体可以访问哪些内网系统，不能访问哪些内网系统**。这时候，则需要搭配 NSX 来实现真正的**端到端安全访问控制**。

![img](https://mmbiz.qpic.cn/mmbiz_png/ory2UDHYWeNUGLGWibZcZX9iaiamOkY05TvdZ4YJc8ianOkMt9gNqnm0HCYxeFXuVl5ibEI7qNXgCUD5appOTxia2eibA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

下面是 Airwatch 与 NSX 集成的演示视频，长1分35秒：

------

再回到第一个章节的问题：桌面虚拟化和云计算有关系吗？

除了前面提到的异同，可能有人注意到了，两者都建立在基础架构平台之上，即 vSphere 平台。

NSX 在基础架构平台中提供网络和安全的功能，必然是可以和其上层建筑结合实现端到端的网络和安全，而在当今的安全形势和网络快速发展形势之下，这样的架构也成为了必需。

------

小知识补充：

文中提到了很多 EUC（终端用户计算）的词汇、功能和产品，汇总如下：

- 桌面虚拟化：将用户使用的桌面环境与其终端设备进行分离的技术；
- VDI ：Virtual Desktop Infrastructure，等同于桌面虚拟化；
- Horizon View：VMware 桌面虚拟化解决方案；
- AD：Active Directory，Windows 活动目录；
- ThinApp：可以将一个应用封装成单一可执行文件包，允许在一个环境中不安装此软件直接运行，或用于一个操作系统同时使用多个版本的同一软件；
- App Volume：将虚拟机中安装的应用封装在 VMDK 中，即时将其挂载给任意虚拟桌面使用；
- RDSH：Remote Desktop Session Host，微软 Windows Server 的一种远程桌面服务；
- Airwatch ：统一的企业移动化管理平台，具体功能见本文第二张图。

------

此文写的比预计的长了太多（也怪我想在一篇文章中介绍一个和NSX同样庞大的系统 - EUC），中间花了很多时间收集整理素材，希望可以让各个层次的读者了解虚拟桌面及其安全，能够有所收获。









