---
title: balanceLvs
date: 2019-10-08 08:52:44
tags: [Linux]
categories: tool
---
# 负载均衡之lvs
[官方文档](http://www.linuxvirtualserver.org/zh/index.html)
<!-- more -->
## 1.基础知识
### 1.1 linux命令
* `route` 查看本机的路由表
![route](/images/balanceLvs/route.png)
> -n 不带地址转换，IP显示`route -n`

* `netstat -natp` 查看本机的TCP链接程序
![netstat](/images/balanceLvs/netstat.png)
> -n 不进行转换，全以IP地址显示
>-a 是所有的链接
>-t tcp链接
* `arp -a`查看MAC地址表
### 1.2 一些名称
全称|缩写|描述
:---:|:--:|:---:
虚拟IP地址Virtual IP address|VIP|提供服务的IP地址,如www.baidu.com通过DNS得到的地址
转发的网络地址Director IP address|DIP|负载均衡器上面的IP
真实IP地址Real IP address|RIP|后端集群主机
客户端地址IP Client IP address|CIP|源IP地址

lvs负载均衡的拓扑结构
![tupu](/images/balanceLvs/topuGraph1.png)
> 说明
>1. 负载均衡器工作在网络层，只转发数据包，不进行握手连接
>2. 要保证同一个客户端三次握手，四次分手不可分割，即转发到同一个服务器镜像
>3. 后端的真实服务器都是镜像，一模一样的。
### 1.3 模型概述
#### 1.3.1 D_NAT与S_NAT
客户端请求的数据包源地址和目的地址`CIP-VIP`
当其到达负载均衡器入口时，如果不对该数据包做处理就转发，那么RealServer看到的目的IP地址就是VIP，不属于RealServer的IP,不会做出回应。
**因此在负载均衡器中需要对数据包中的目的地址进行转换**
修改目的地址(**D_NAT技术**)，数据包变为`CIP-RIP`,这个时候RealServer可处理该数据包
> **问题** 此时RealServer建立的Sockt连接是：`RIP:Port---CIP:Port`,所以响应包的源地址和目的地址是：`RIP-CIP`。当这个数据包达到客户端的时候，与客户端发的请求包中的地址不对应了？

因此，负载均衡器还需要负责将响应包的源地址修改为VIP(**S_NAT技术**)。
为了实现这个，RealServer的默认网关(default/0.0.0.0)需要指向LVS负载均衡器。
> 缺点：所有数据(上行，下行)都要经过LVS，I/O瓶颈在这里会产生。
> 可行方案： 每一个RealServer不要将响应数据发给LVS,直接将响应包发给客户端。
#### 1.3.2 DR模型
> 让RealServer拥有一个VIP这样的IP地址，但是不暴露在公网，自己有一个隐藏的VIP地址

客户端发送`CIP-VIP`数据包，经过LVS到达RealServer,因为RealServre有一个隐藏的VIP地址，所以会接收该数据包，然后将响应包`VIP-CIP`直接发送给客户端，不用通过lvs负载均衡器。
* 问题： 客户端的数据包是`CIP-VIP`，RealServer暴露的IP是RIP，lvs如何将客户端数据包发给RealServer。
> 解答： 通过arp欺骗。lvs与RealServer必须在同一个局域网内

**缺点** :要求lvs与RealServer在同一个局域网内，对分布在地理位置不同的数据中心无法做负载均衡。
#### 1.3.3 Tunnel与VPN模型
客户端发出数据包`CIP-VIP`。数据包到达lvs的时候，建立一个新的请求，发送数据包`DIP-RIP`,这个数据包会将客户端的所有数据都封装起来。然后通过公网到RealServer。
RealServer会拆包，得到里面的`CIP-VIP`请求。自己有隐藏的VIP地址。
### 1.4 linux隐藏IP的方法
对内可见，对外隐藏。
**需要隐藏的IP配置到外部无法访问的网卡上**就是lo网卡，环回接口
注意：一个网卡可以添加多个IP地址，所以环回地址127.0.0.1不受影响。
>基础知识：在一个局域网内，如果主机A的arp表中无主机B的mac地址且要发送数据包到主机B，就会进行arp请求。这个时候主机B会发送响应请求，告诉主机A它的IP和对应的MAC地址。

配置文件：`/proc/sys/net/ipv4/conf/`
* 对于`arp_ignore`控制系统在收到外部的arp请求时，是否要返回arp响应,**配置为1**
>arp_ignore为0：回应任何网络接口（网卡）上对任何本机IP地址的arp查询请求
为1：只回答目标IP地址是本机上来访网络接口（网卡）IP地址的ARP查询请求 。比如eth0=192.168.0.1/24,eth1=10.1.1.1/24,那么即使eth0收到来自10.1.1.2这样地址发起的对192.168.0.1的查询会回应，而对10.1.1.1 的arp查询不会回应。
* `arp_announce`定义将自己地址向外通告时的通告级别，**配置为2**
>0：将本地任何接口上的任何地址向外通告；
1：试图仅向目标网络通告与其网络匹配的地址；
2：仅向与本地接口上地址匹配的网络进行通告


## 2. DR实验
### 2.1 lvs简介
lvs： Linux Virtual Server，linux虚拟服务器。
核心程序：ipvs（嵌入到了Linux内核中）
管理程序：ipvsadm
调度算法：
<table>
<tr><td colspan="2">调度方式</td><td>英文</td><td>描述</td></tr>
<tr><td rowspan="4">静态调度</td><td>轮询调度</td><td>Round Robin 简称'RR'</td><td>依次循环的方式将请求调度到不同的服务器上</td></tr>
<tr><td>加权轮询调度</td><td>Weight Round Robin 简称'WRR'</td><td>权值越高的服务器，处理的请求越多</td></tr>
<tr><td>目标地址散列调度</td><td>Destination Hashing 简称'DH'</td><td>算法先根据请求的目标IP地址，作为散列键（Hash Key）从静态分配的散列表找出对应的服务器，若该服务器是可用的且并未超载，将请求发送到该服务器，否则返回空</td></tr>
<tr><td>源地址散列调度</td><td>Source Hashing  简称'SH'</td><td>先根据请求的源IP地址，作为散列键（Hash Key）从静态分配的散列表找出对应的服务器，若该服务器是可用的且并未超载，将请求发送到该服务器，否则返回空。</td></tr>
<tr><td rowspan="6">动态调度</td><td>最小连接调度</td><td>Least Connections 简称'LC'</td><td>新的连接请求分配到当前连接数最小的服务器</td></tr>
<tr><td>加权最小连接调度</td><td>Weight Least Connections 简称'WLC'</td><td>加权最小连接调度在调度新连接时尽可能使服务器的已建立连接数和其权值成比例。调度器可以自动问询真实服务器的负载情况，并动态地调整其权值</td></tr>
<tr><td>最短的期望的延迟</td><td>Shortest Expected Delay 简称'SED'</td><td>请求交给得出运算结果最小的服务器</td></tr>
<tr><td>最少队列调度</td><td>Never Queue 简称'NQ'</td><td>无需队列。如果有realserver的连接数等于0就直接分配过去，不需要在进行SED运算。</td></tr>
<tr><td>带复制的基于局部性的最少连接</td><td>Locality-Based Least Connections with Replication  简称'LBLCR'</td><td>按'最小连接'原则从该服务器组中选出一一台服务器，若服务器没有超载，将请求发送到该服务器；若服务器超载，则按'最小连接'原则从整个集群中选出一台服务器，将该服务器加入到这个服务器组中，将请求发送到该服务器。同时，当该服务器组有一段时间没有被修改，将最忙的服务器从服务器组中删除，以降低复制的程度。</td></tr>
<tr><td>基于局部的最少连接</td><td>Locality-Based Least Connections 简称'LBLC'</td><td>LBLC调度算法先根据请求的目标IP地址找出该目标IP地址最近使用的服务器，若该服务器是可用的且没有超载，将请求发送到该服务器；若服务器不存在，或者该服务器超载且有服务器处于一半的工作负载，则使用'最少连接'的原则选出一个可用的服务器，将请求发送到服务器。</td></tr>
</table>

### 2.2 ipvsadm的使用
配置主要分为两个阶段，第一配置哪些数据包需要进行转发，第二配置数据包转发到哪里去。
* 安装管理模块
```
centOS: yum install ipvsadm -y
```
* 配置哪些数据包需要lvs转发
```
格式： ipvsadm -A -t|u|f ServerAddress [-s schduler]
-t: tcp协议的集群 地址为IP:PORT
-u: udp协议的集群 地址为IP:PORT
-f: 防火墙标记 地址为MarkNumber
```
eg：`ipvsadm -A -t 192.168.10.10:80 -s rr` 目的地址为192.168.10.10:80的tcp协议的数据包需要进行lvs，使用rr调度算法进行负载均衡。
* 配置数据包转发到RealServer
```
格式：ipvsadm -a -t|u|f service-address -r server-address [-g|i|m] [-w weight]
前部分是配置的哪些数据包需要lvs转发，即配置的集群服务
-r 该参数后面是RealServer的地址，可以是IP:PORT
-g DR模式
-i Tunnel模式
-m NAT模式
-w 该参数和值定义服务器权重
```
eg: `ipvsadm -a -t 192.168.10.10:80 -r 192.168.10.8 –g`


### 2.3 实验拓扑
![topu](/images/balanceLvs/drModel1.png)
蓝色的路径是请求数据包的路线，绿色的是响应数据包的路线。忽略了网关。


### 2.4 实验过程
* step1: 三台虚拟机，eth0在同一个网络192.168.10.0
* step2: 配置node1(lvs)的VIP
```
ifconfig eth0:0 192.168.10.10/24
echo “1” > /proc/sys/net/ipv4/ip_forward
```
> 注意：当计算机收到一个目的地址不属于自己IP的数据包时候默认会丢弃，因此lvs这个需要设置ip_forward的值为1，充当路由器的功能，将数据包转发出去。
* step3: 修改RealServer的arp响应和通告级别
```
echo “1” > /proc/sys/net/ipv4/ip_forward
echo 1  > /proc/sys/net/ipv4/conf/eth0/arp_ignore
echo 2  > /proc/sys/net/ipv4/conf/eth0/arp_announce
echo 1  > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2  > /proc/sys/net/ipv4/conf/all/arp_announce
```
* step4: 配置每一台RealSerer的VIP(隐藏IP)
```
ifconfig lo:8 192.168.10.10 netmask 255.255.255.255
```
> 注意: 响应和通告级别一定要先配置，不然VIP后被通告出去。

至此，三个节点的信息如下
![nods](/images/balanceLvs/nodes.png)
* step5: 启动RrealServer上的httpd
```
yum install httpd -y
#启动命令
service httpd start

```
静态网页的路径`/var/www/html`


#参考文献
[1] http://www.linuxvirtualserver.org/zh/index.html
[2] https://www.cnblogs.com/gaoxu387/p/7941381.html
[3] https://blog.csdn.net/weixin_40470303/article/details/80541639

