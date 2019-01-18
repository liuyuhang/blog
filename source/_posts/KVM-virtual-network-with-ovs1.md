---
title: kvm虚拟化网络手动实践1-虚拟化环境准备
tags: kvm
categories: network
date: 2019-01-19
---

撰写本系列文章，主要是想要亲自手动实践一下目前主流在虚拟化、云的环境中使用的网络技术，当前已经有很多开源的云计算平台对网络部分功能实现的非常完整，比如开源的Openstack的neutron模块，大家也可以学习其源码，加深理解。但是基于平台的这种网络功能，其设计原则就是尽量屏蔽底层细节，操作只是在界面上配置，底层技术人员在刚开始学习时并不能很明确的知道底层真正是如何工作的。

这个系列的主旨就是想通过原生的KVM，结合OVS、linux bridge、iptables等技术，实现目前云平台中的网络互联、网络隔离、DHCP、虚拟路由、Floating IP、虚拟防火墙、虚拟负载均衡等功能。

## 虚拟化环境准备

在一切的开始，我们先准备一下我们在接下来要使用的硬件必要环境：
|    实体       |   要求      |
| -------- | -------------- |
| CPU      | 支持硬件虚拟化 |
| 网卡数量 | 2块或以上       |

如果是使用虚拟程序创建，例如workstation等，创建虚拟机过程中需要打开硬件虚拟化透传技术，以便能够在这个host虚拟机中再创建kvm虚拟机

### 安装操作系统

操作系统：CentOS7 我使用的是[7.6.1810的minimal版本](http://mirrors.ustc.edu.cn/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso)

安装完成后验证是否支持硬件虚拟化：

```bash
egrep -o '(vmx|svm)' /proc/cpuinfo
// 如果输出vmx 或 svm则说明支持硬件虚拟化
```

### 配置基础网络及更新

先配置一下基础网络，让虚拟机可以上网

```bash
// 根据用户硬件情况配置ip及网关
ip link set dev eth0 up
ip addr add 10.211.55.200/24 dev eth0
// 配置默认网关
ip route add default via 10.211.55.1
// 设置DNS
echo "nameserver 8.8.8.8" > /etc/resolv.conf
// ping 一下看是否连通
ping 8.8.8.8
```

安装完操作系统之后登陆进虚拟机中，添加epel包并更新：

```bash
yum install epel-release
yum update
```

### 安装必要软件包

安装 libvirtd、qemu-kvm、vim等

```bash
yum install libvirtd qemu-kvm vim wget
```

修改启动虚拟机的用户权限

```bash
vim /etc/libvirt/qemu.conf

//将其中的两行：
# user = "root"
# group = "root"
//取消注释
user = "root"
group = "root"
```

将firewalld关闭

```bash
systemctl disable firewalld
systemctl stop firewalld
```

启动libvirtd

```bash
systemctl enable libvirtd
systemctl start libvirtd
```

验证：

```bash
virsh list
 Id    名称                         状态
----------------------------------------------------


// 代表libvirtd运行正常
```

### 创建第一个虚拟机

1. 创建虚拟机文件夹

   ```bash
   // 镜像存储目录
   mkdir -p /opt/vms/base
   // 创建第一个虚拟机的文件夹
   mkdir -p /opt/vms/vm1
   ```

2. 下载镜像

   ```bash
   cd /opt/vms/base
   wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
   ```

3. 拷贝镜像、撰写配置文件

   ```bash
   cd /opt/vms/vm1
   // copy 镜像
   cp /opt/vms/base/cirros-0.4.0-x86_64-disk.img ./
   // 配置文件
   vim vm1.xml
   ```

   ```xml
   <domain type="kvm">
         <name>vm1</name>
         <memory unit="MiB">256</memory>
         <currentMemory unit="MiB">256</currentMemory>
         <vcpu placement="static">1</vcpu>
         <cpu mode="custom">
             <topology sockets="1" cores="1" threads="1"></topology>
         </cpu>
         <resource>
             <partition>/machine</partition>
         </resource>
         <os>
             <type arch="x86_64">hvm</type>
             <boot dev="hd"></boot>
             <boot dev="cdrom"></boot>
         </os>
         <features>
             <acpi></acpi>
             <apic></apic>
             <pae></pae>
         </features>
         <clock offset="utc"></clock>
         <on_poweroff>destroy</on_poweroff>
         <on_reboot>restart</on_reboot>
         <on_crash>destroy</on_crash>
         <devices>
             <emulator>/usr/libexec/qemu-kvm</emulator>
             <disk type="file" device="disk">
                 <driver name="qemu" type="qcow2" cache="none"></driver>
                 <source file="/opt/vms/vm1/cirros-0.4.0-x86_64-disk.img"></source>
                 <target dev="hda" bus="ide"></target>
             </disk>
             <controller type="ide" index="0">
                 <alias name="ide0"></alias>
             </controller>
             <controller type="pci" index="0" model="pci-root">
                 <alias name="pci.0"></alias>
             </controller>
             <controller type="usb" index="0" model="ich9-ehci1">
                 <alias name=""></alias>
             </controller>
             <controller type="usb" index="0" model="ich9-uhci1">
                 <alias name=""></alias>
             </controller>
             <input type="tablet" bus="usb">
                 <address type="usb" bus="0" port="1.1"></address>
             </input>
             <graphics type="vnc" port="5900" autoport="yes" listen="0.0.0.0" passwd="123"></graphics>
             <video>
                 <model type="vga" vram="32768" heads="1"></model>
             </video>
             <memballoon model="none"></memballoon>
             <hub type="usb">
                 <address type="usb" bus="0" port="1"></address>
             </hub>
         </devices>
     </domain>
                       
   ```

   这个是一个相对来说比较普适的一个xml配置文件

   可以根据自己的需要进行更改，可以参考：[libvirt xml](https://libvirt.org/formatdomain.html)

   注：这个配置文件中没有配置网卡，之后我们会在这个配置文件上进行修改，逐步实现虚拟化环境中的各种典型网络配置

4. 开启虚拟机

   ```bash
   # cd /opt/vms/vm1
   # virsh create vm1.xml
   域 vm1 被创建（从 vm1.xml）
   
   // 验证：
   # virsh list
    Id    名称                         状态
   ----------------------------------------------------
    2     vm1                            running
   
   ```

   说明我们的虚拟机已经创建成功了

5. 登陆vnc控制台

   我们可以使用vncviewer等类型的工具进行查看，访问主机IP:5900端口 ，密码是123 则可访问创建的虚拟机桌面：
   
   ![image-20190118161221604](/images/vnc-1.png)
   
   ![image-20190118162223402](/images/vnc-2.png)

   


至此，我们的虚拟化环境已经准备完成，接下来我们会将网络从简单到复杂的实现过程进行逐步讲解。

