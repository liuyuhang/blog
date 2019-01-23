---
title: kvm虚拟化网络手动实践3-4种网络逻辑架构简述（vlan）
tags: [kvm,network]
categories: network
date: 2019-01-21
typora-root-url: ../../source
---

之前我们看过了local和flat的网络结构，但我们在多租户、或者多网络需要隔离时，local和flat的网络就不能满足我们的需求了，我们要借助于虚拟局域网技术Vlan，或者更高层级的的Overlay Network（Vxlan）

## Vlan Network

vlan的整体网络结构和flat的network是相同的，如下图所示：

![image-20190123140052151](/images/image-20190123140052151.png)

不同的地方主要是在vm的网络流量进入到br-int后，经过处理的过程。其中使用了openflow的流管理方法。

我们先简单的梳理一下从vm1发送给vm4的流量是怎样流通的

vm1 -> 物理网络流程

1. 流量从虚拟机vm1网卡出来到达tap-vm1设备
2. tap设备将流量发送到安全网桥Linux Bridge（qbr-1）上，经过安全组规则过滤后，将流量发送到br-int
3. 流量到达br-int之前，会被打上一个tag标签，这个标签只在本机范围之内起作用，一般是用于隔离虚拟机之间的东西流量。
4. 流量到达br-int网桥后，首先去掉内部tag标签，然后根据之前设定的vlan tag，将流量打上对应的真实vlan tag标签，再发送到br-eth1上。
5. br-eth1收到流量后，就将流量发送给物理网卡eth1，至此，虚拟机的流量已经正确的属于固定的vlan在物理网络上流通了

注意：在vlan的网络模型中eth1所连接的交换机端口要是trunk模式，允许通过我们设定的vlan

物理网络 -> vm4

1. 从vm1发来的流量在到达物理网络之后通过交换机发送到server2的eth1网卡
2. eth1网卡将流量直接发给ovs网桥br-eth1，br-eth1再将流量发给br-int
3. br-int将真实的vlan tag标签去掉，根据规则，再绑定vm4在server内部使用的tag标签，然后发给qbr-4
4. qbr-4进行网络安全组过滤，通过的流量经由tap设备，最终被vm4接收

整个vlan网络的逻辑大概是这个样子，在之后的章节我们会将上述的流程，通过各种配置进行实现。

