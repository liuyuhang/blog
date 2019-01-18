---
title: iproute2 常用命令汇总
tags: linux   iproute2
categories: network
date: 2019-01-17

---
目前大部分的Linux操作系统都是以iproute2作为系统预装网络管理包，原有的ifconfig、route、netstat等都不做为默认的安装包安装了。

iproute2提供了统一的网络配置命令，对之前分散在各个工具中的功能进行了整合，虽然用惯了之前的命令，但新的iproute2命令是未来的趋势，需要好好掌握。

### IP地址管理

1. 列出IP address

   ```bash
   ip address show
   //简写
   ip a
   ```

2. 列出具体设备address

   ```bash
   ip address show eth0
   ```

3. 只列出up状态的address

   ```bash
   ip address show up
   ```

4. 设置IP

   ```bash
   // 为eth0设置10.1.1.22的ip
   ip addr add 10.1.1.22/24 dev eth0
   ```

   这里我们可以看到原来的ifconfig还可以设置gateway，这里是没有的，需要的话可以使用 ip route进行添加

   ```bash
   // 指定gateway
   ip route add 10.1.1.0/24 via 10.1.1.1
   ```

5. 清除IP

   ```bash
   // 删除指定IP
   ip address delete 10.1.1.22/24 dev eth0
   // 删除所有IP
   ip address flush dev eth0
   // 仅删除ipv4的IP
   ip -4 address flush
   ```

### 链接管理（Link）

1. 列出所有link

   ```bash
   ip link show
   ip link list
   ```

2. 网卡up和down

   ```bash
   // 启用eth0
   ip link set dev eth0 up
   // 禁用eth0
   ip link set dev eth0 down
   ```

3. 开启、关闭组播

   ```bash
   // 开启多播（组播）
   ip link set eth0 multicast on
   // 关闭多播（组播）
   ip link set eth0 multicast off
   ```

4. 配置vlan

   ```bash
   // 在eth0上添加一个vlan100的接口
   ip link add name eth0.100 link eth0 type vlan id 100
   // 删除 vlan100 的接口
   ip link delete dev eth0.100
   ```

5. 配置网桥（bridge）

   网桥配置：

   ```bash
   // 创建网桥
   ip link add name br0 type bridge
   // 删除网桥
   ip link delete dev br0
   // 和brctl不同，ip link创建的网桥不会自动up，需要手动up
   ip link set dev br0 up
   ```

   网桥接口配置（interface）：

   ```bash
   // 为网桥br0添加一个接口eth0
   ip link set dev eth0 master br0
   // 将接口eth0从网桥br0删除
   ip link set dev eth0 nomaster
   ```

6. 配置绑定（bond）

   基本配置：

   ```bash
   // 创建一个bond设备
   ip link add bond0 type bond
   // 设置bond类型
   ip link set bond0 type bond miimon 100 mode active-backup
   // 添加em1至bond0
   ip link set em1 down
   ip link set em1 master bond0
   // 添加em2至bond0
   ip link set em2 down
   ip link set em2 master bond0
   // 启动bond0
   ip link set bond0 up
   ```

   为bond设备设置vlan

   ```bash
   ip link add link bond0 name bond0.200 type vlan id 200
   ip link set bond0.200 up
   ```

   将打过vlan的bond绑定至vlan

   ```bash
   ip link add br0 type bridge
   ip link set bond0.200 master br0
   ip link set br0 up
   ```

### Tap and Tun

1. 列出所有tap、tun 设备

   ```bash
   ip tuntap show
   ```

2. 创建tap、tun设备

   ```bash
   // 创建tun0设备
   ip tuntap add dev tun0 mode tun
   //创建tap0设备
   ip tuntap add dev tap0 mode tap
   ```

3. 删除tap、tun设备

   ```bash
   // 删除tun设备
   ip tuntap delete dev tun0 mode tun
   // 删除tap设备
   ip tuntap delete dev tap0 mode tap
   ```

### route路由管理

1. 列出所有的route

   ```bash
   ip route
   //或
   ip route show
   ```

   

2. 列出某个子网的所有route

   ```bash
   ip route show to match 192.168.0.0/24
   ```

   

3. 添加gateway route

   ```bash
   ip route add 192.0.2.128/25 via 192.0.2.1
   ```

   

4. 添加设备route

   ```bash
   ip route add 192.0.2.0/25 dev eth2
   ```

   

5. 修改route

   ```bash
   // change用于修改当前路由
   ip route change 192.168.2.0/24 via 10.0.0.1
   // replace用于修改路由，如果不存在，则创建路由
   ip route replace 192.0.2.1/27 dev tun0
   ```

   

6. 删除路由

   ```bash
   ip route delete 10.0.1.0/25 via 10.0.0.1
   //或
   ip route delete default dev ppp0
   ```

   

7. 设置默认路由

   ```bash
   ip route add default via 192.168.0.1
   ip route add default dev eth0
   // 等价于
   ip route add 0.0.0.0/0 via 192.168.0.1
   ip route add 0.0.0.0/0 dev eth0
   ```

   

8. 黑洞路由（blackhole router）

   ```bash
   // 添加黑洞路由
   ip route add blackhole 192.0.2.1/32
   // 删除黑洞路由
   ip route delete blackhole 192.0.2.1/32
   ```

   

9. 多路由，用于链路备份

   有的时候除去使用bond进行链路的备份，还可以使用多路由的方式进行链路备份，

   具体的实现方式是使用不同的metric，路由的时候会先选择metric小的转发，当转发失败后再使用次小的metric路由转发

   ```bash
   ip route add 192.168.2.0/24 via 10.0.1.1 metric 5
   ip route add 192.168.2.0 dev ppp0 metric 10
   ```

   例如上述设置，如果10.0.1.1转发失败，会使用ppp0再行转发

### namespace管理

1. 列出所有namespace

   ```bash
   ip netns list
   ```

   

2. 创建namespace

   ```bash
   ip netns add ns-name
   ```

   

3. 删除namespace

   ```bash
   ip netns delete ns-name
   ```

   

4. 在namespace中执行命令

   ```bash
   ip netns exec foo $command
   // 例：
   ip netns exec foo ip link list
   ```

   

5.  设置一个设备属于一个namespace

   ```bash
   // 将eth0放置在ns-name命名空间中
   ip netns set dev eth0 netns ns-name
   ```

   

6. 链接两个namespace

   链接两个namespace是使用veth设备进行连接

   ```bash
   //创建一对veth设备
   ip link add name veth1 type veth peer name veth2
   // 将其中一个移动到foo命名空间
   ip link set dev veth2 netns foo
   //设置启动并设定ip
   ip netns exec foo ip link set dev veth2 up
   ip netns exec foo ip address add 10.1.1.1/24 dev veth2
   // 设定默认命名空间veth ip
   ip link set dev veth1 up
   ip address add 10.1.1.2/24 dev veth1
   ```

   此时两个不同命名空间的veth就可以ping通了

### 列出网络状态

ss命令是是和netstat命令相似的一套命令，但数据信息会更详细，经常使用的命令如下：

```bash
ss -nplt
//详细使用信息请参照ss的帮助文档
ss --help
// 或者
man ss
```

