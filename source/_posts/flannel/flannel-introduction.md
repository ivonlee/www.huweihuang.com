---
title: "[Kubernetes] Flannel介绍"
catalog: true
date: 2017-07-08 10:50:57
type: "categories"
subtitle:
header-img: "/img/article_header_img/article_header.png"
tags:
- Kubernetes
catagories:
- Kubernetes

---

## 一、flannel是什么（what）

### （一）概述

Flannel是CoreOS团队针对Kubernetes设计的一个网络规划服务，简单来说，它的功能是让集群中的不同节点主机创建的Docker容器都具有全集群唯一的虚拟IP地址。
Flannel官网：https://github.com/coreos/flannel

### （二）补充知识点

#### **1、覆盖网络[overlay network]**

运行在一个网上的网（应用层网络），并不依靠ip地址来传递消息，而是采用一种映射机制，把ip地址和identifiers做映射来资源定位。

#### **2、路由**

互联网是由路由器连接的网络组合而成，路由器按照路由表、路由协议等机制实现对数据包正确地转发，从而到达目标主机。路由器根据数据包中目标主机的IP地址和路由控制表比较得出下一个接收数据的路由器。

**1）静态路由：事先设置好路由器和主机中的路由表信息。** 

 ![静态路由](/img/article/flannel/静态路由.png)

**2）动态路由：让路由协议在运行中自动修改并设置路由表信息。**

![动态路由](/img/article/flannel/动态路由.png)

## 二、为什么使用flannel（why）

在默认的Docker配置中，每个节点上的Docker服务会分别负责所在节点容器的IP分配。这样导致的一个问题是，不同节点上容器可能获得相同的内外IP地址。

Flannel的设计目的就是为集群中的所有节点重新规划IP地址的使用规则，从而使得不同节点上的容器能够获得“同属一个内网”且”不重复的”IP地址，并让属于不同节点上的容器能够直接通过内网IP通信。

## 三、如何实现flannel（how）

Flannel实质上是一种“覆盖网络(overlay network)”，也就是将TCP数据包装在另一种网络包里面进行路由转发和通信，目前已经支持UDP、VxLAN、AWS VPC和GCE路由等数据转发方式，默认的节点间数据通信方式是UDP转发。

### （一）flannel原理图

![flannel](/img/article\flannel\flannel.png)

1. 数据从源容器中发出后，经由所在主机的docker0虚拟网卡转发到flannel0虚拟网卡，这是个P2P的虚拟网卡，flanneld服务监听在网卡的另外一端。
2. Flannel通过Etcd服务维护了一张节点间的路由表。
3. 源主机的flanneld服务将原本的数据内容UDP封装后根据自己的路由表投递给目的节点的flanneld服务，数据到达以后被解包，然后直 接进入目的节点的flannel0虚拟网卡，然后被转发到目的主机的docker0虚拟网卡，最后就像本机容器通信一下的有docker0路由到达目标容 器。

### （二）实现说明

#### **1、UDP封装**

原始数据是在起始节点的Flannel服务上进行UDP封装的，投递到目的节点后就被另一端的Flannel服务还原成了原始的数据包，两边的Docker服务都感觉不到这个过程的存在。 UDP的数据内容部分其实是另一个ICMP（也就是ping命令）的数据包。

![UDP封装](/img/article/flannel/UDP封装.png)

#### **2、为docker分配不同的IP段**

Flannel通过Etcd分配了每个节点可用的IP地址段后，偷偷的修改了Docker的启动参数。

![docker启动参数](/img/article/flannel/docker启动参数.png)

注意其中的“--bip=172.17.18.1/24”这个参数，它限制了所在节点容器获得的IP范围。

这个IP范围是由Flannel自动分配的，由Flannel通过保存在Etcd服务中的记录确保它们不会重复。

#### **3、路由规则**

1）数据发送节点的路由表 

![数据发送节点路由表](/img/article/flannel/数据发送节点路由表.png)

2）数据接收节点的路由表 

![数据接收节点路由表](/img/article/flannel/数据接收节点路由表.png)

例如现在有一个数据包要从IP为172.17.18.2的容器发到IP为172.17.46.2的容器。根据数据发送节点的路由表，它只与 172.17.0.0/16匹配这条记录匹配，因此数据从docker0出来以后就被投递到了flannel0。同理在目标节点，由于投递的地址是一个容 器，因此目的地址一定会落在docker0对于的172.17.46.0/24这个记录上，自然的被投递到了docker0网卡。

### （三）flannel的安装与配置
#### **1、安装**
```
wget http://<官网>/flannel/flannel-0.2.0-10.el7.x86_64.rpm
yum localinstall -y flannel-0.2.0-10.el7.x86_64.rpm
```
#### **2、配置**
vi /etc/sysconfig/flanneld
```shell
# Flanneld configuration options
 
# etcd url location. Point this to the server where etcd runs
FLANNEL_ETCD="http://127.0.0.1:4001"
  
# etcd config key. This is the configuration key that flannel queries
# For address range assignment
FLANNEL_ETCD_KEY="/wae/flannel/product/network"
  
# Any additional options that you want to pass
FLANNEL_OPTIONS=" -iface=eth0"
```
#### **3、初始化flannel的etcd配置**
```
etcdctl set /wae/flannel/network/config '{
   "Network": "10.0.0.0/16",
   "Backend": {
       "Type": "vxlan"
   }
}'
```