## Storage I/O 和 VVOL’s 基本概念



## 使用传统存储面临的问题

- LUN的大小受限，为了保证业务未来的冗余，一般都会预留一部分空间，有时会造成资源浪费。
- 多个LUN的管理复杂，资源利用可能不均，且没有有效的管理方法
- vSphere不能去管理存储的LUN，存储的管理和vsphere环境的管理分开，造成管理复杂，较长的置备周期。
- 不能利用存储自己的技术去克隆或复制单个虚拟机（除非使用VAAI）。
- 存储的颗粒度最细为LUN，不利于VM的生命周期管理




## 解决方案






## vVOL组件

### 1. VASA Provider

是一个vSphere API，用于storage awareness

使用VASA查找VM的VMDK文件。





### 2. Storage Container

LUN的第一个功能：存储数据的部分

Logical storage constructs for grouping of virtual volumes.

最大容量取决于物理存储的容量

进行VM的存储需求进行逻辑分离

每个存储最小一个storage container

一个storage container可以同时被多个PE访问

---

这里有个Datastore的概念，本来可以直接做对象存储，但是vSphere的部分功能例如DRS只能识别Datastore，所以讲抽象的东西具体化了

---

### 3. Protocol Endpoint

**LUN 存储数据、LUN同时是access point。这样只能一一对应的使用LUN。**

**PE （access point）只处理ESXi和存储之间的IO。**

由存储管理员创建。

属于物理存储的fabric。

将access point和storage分离后，只需要少数的access point，就可以和后端很多LUN通信。

---

PE 支持以下SAN和NAS协议： iSCSI ，NFSv3，FC，FCoE

支持现有的多路径策略

SCSI PE 在 ESXi的rescan时被发现

NFS PE are maintained as IP addresses or file paths

ESXi will identity PE and maintain all discovered PEs in a database

 



### 4. SPBM



(IBM XIV 配合vVOL)[https://www.youtube.com/watch?v=HZtf5CaJx_Y]

两个用途

1、不再有VMFS层，虚拟机的vmdk直接存放在存储上，做存储快照时可以直接对虚拟机进行快照。而之前只能对vmfs对应的LUN做快照。

2、可以实现快速的复制



