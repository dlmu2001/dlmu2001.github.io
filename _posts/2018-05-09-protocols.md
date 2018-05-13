---
layout: post
title: 常用网络协议介绍 
categories: [物联网]
tags: [基础]
description: team内部培训，面向智能硬件或者物联网的同学 
---

tomorrow.cyz@gmail.com 

# 1. OSI模型

>>![](/assets/media/network_osi_model.png) 

# 2. wlan

## 2.1 基本概念

### wlan 
  
wlan是wireless local area networt的简称，是在有限区域内通过无线通信将两台或者多台设备连接在一起的无线计算机网络。

关键特性 1）局域网 2）无线（可以移动)

### WiFi
大部分wlan都是基于IEEE 802.11实现，这种基于802.11的无线局域网技术称为WiFi。

### 无线频谱
无线设备被限定在某个特定频段(frequency band)上操作。每个频段都有相应的频宽 (bandwidth)，亦即该频段可供使用的频率空间总和。频宽是评价链路(link)数据传输能力的基准。较大的频宽可以传输更多的信息。

无线频谱(radio spectrum)的使用受到主管当局严格控管。

### STA 和AP
STA是Station的简写，指的是有能力使用802.11协议的设备，比如手机、电脑等。

AP是access point的简称，是用来帮助Wi-Fi设备接入有线的网络硬件。

### BSS
BSS是basic service set的简称，是802.11网络的基本原件(building block)，由一组彼此通信的工作站构成。BSS有两种形式

>>![](/assets/media/network_wlan_bss.jpg) 

Independant BSS简称IBSS，它一般是少数几个工作站针对特定目的而组成的临时网络，有时也叫ad hoc network。它是一种去中心化的网络。

Infrastructure BSS，任何设备之间的通信，都需要通过AP。

### BSSID
BSSID是每个BSS的唯一标识(48位），BSSID一般是Mac地址，不可配置。

### ESS
ESS是Extended service sets的简称，它是指通过骨干网将几个BSS连接在一起。

>>![](/assets/media/network_wlan_ess.jpg) 

### SSID
所有位于同一个ESS的AP将会使用同一个服务组合识别码，这个识别码就是SSID(Set Identifier)。

>>![](/assets/media/network_wlan_arch.jpeg) 

## 2 wlan的类型

### 2.1 infrasture

infrastructure是大部分WiFi网络的部署方式，在infrastructure方式中，所有的STA必须通过AP才能通信。

### 2.2 Peer-to-Peer

### 2.3 wlan mesh

## 3 802.11协议
### 2.1 802.11协议在osi模型中的位置

>>![](/assets/media/network_wlan_osi.jpg) 

### 2.2 802.11协议族

>>![](/assets/media/network_80211.jpeg) 

## 1.4 协议主要功能

协议的主要功能是传输和管理。具体可以分成以下服务。
* 传输(Distribution)
* 整合(Integration)
* 连接(Association)
* 重新连接(Reassociation)
* 解除连接(Disassociation)
* 身份认证(Authentication)
* 解除认证(Deauthentication)
* 机密性(Confidentiality)
* MSDU(Mac Service Data Unit)传递
* 传输功率控制(TPC:Transmit Power Control)
* 动态频率选择(DFS)

## 1.5 MAC
### CSMA/CA 

# 3. IP协议

IP协议提供从发起者(source)到目的地(destination)的数据传输。

IP协议的基本功能：寻址(Addressing) 和分片(fragmentation)

## 3.1 寻址

Internet协议的功能和目的是通过一个互联的网络传输数据报。

这是通过从一个internet模块到另外一个internet模块传递数据报直到目的地址来实现。

Internet模块位于主机上或者internet系统上的网关。

数据报通过基于一个internet地址的解析从一个internet模块选路到另一个internet模块。

### 3.1.1 IP地址

区别于门牌号的地方是，这个host的ip地址可能会动态改变。(域名-->集群-->IP地址池)

IP数据报想要传递到目的地（destination），前提是必须知道目的地ip地址(dest ip addr)。

IPV4中，地址固定为4段8位的地址(32位），分为网络地址(network)和host地址(host)。IP地址的类型有三种

A类地址:第一段的最高位是0，其它7位为网络地址,剩下的24位为host地址。其中127.0.0.0~127.255.255.255为环回地址,用于本地环回测试等用途，
0.0.0.0表示当前主机。

B类地址:第一段的前两位为10，剩下的6位加上第二段共14位为网络地址，最后两段（16位）为本地地址。

C类地址:前三位为110，前三段剩下的21位为网络地址，最后一段（8位）为host地址。

D类地址：前四位为1110，前三段剩下的20位位网络地址，最后一段为host地址，一般用于组播。

E类：前四位为1111，其中255.255.255.255为全网广播地址。

>>![](/assets/media/network_ip_addr_class.jpg) 

私有地址:在IP地址的三种类型里面，都保留了一个区域作为私有地址，比如C类地址，保留了192.168.0.0 -- 192.168.255.255

### 3.1.2 子网

在IP地址分类中，同一个网络地址下面的host太多了（最少的C类地址，一个network可以有255个host），这种情况下，将它们再
分成不同的子网会更有利于管理。

比如，对于192.168.1这个网络地址，host有192.168.1.1 ---192.168.1.255 公255个host，可以分成两个子网（需要从8位host地址
额外拿出一位来做子网标识)

子网可以增加安全性，提高性能（减少广播），利于组织。

子网掩码用来分隔network地址和host地址，在子网存在的情况下，network地址包含子网标识。子网掩码就是network所占的位全部

置1，host地址所占的位位0。比如上例的子网掩码，位11111111.11111111.11111111.10000000,也就是255.255.255.128。

通过将IP地址和子网掩码进行位与计算，可以得到子网地址。

>>![](/assets/media/network_ip_subnet.jpg) 

### 3.1.3 寻址模型

#### 3.1.3.1 每个希望同其它host通信的host拥有一个ip地址

这个能力涉及到IP地址的管理和分发，局域网中，现在常见的是用DHCP协议来实现。Host端实现DHCP Client，网关实现DHCP server。

公网IP地址的管理和分发，则由专门的机构负责（NIC,APNIC）。

#### 3.1.3.2 L2提供包传输能力，需要知道对端的Mac地址

#### 3.1.3.3 Host以及Host所在的网络具备IP和Mac地址转化的能力
    
这个能力主要通过ARP协议来实现。

#### 3.1.3.4 路由器提供转发和寻径能力
    
路由器在互联网络中处于关键地位。它的主要工作就是为经过路由器的每个数据帧寻找一条最佳传输路径，并将该数据有效地传送到目的站点。

有很多路由协议用来支撑这个工作，比如RIP,OSPF等。

路由协议主要有三个功能 1）发现网络上的其它路由器 2）路由管理：记录所有可能destination以及到达这些destination的路径 3)路径决策

#### 3.1.3.5 寻址过程

HostA（src_ip）要发送数据报给HostB（dest_ip），首先将dest_ip与自己的子网掩码按位与，得到dest_ip的子网地址

如果dest_ip的子网地址与自己的子网一致,

### 3.1.4 公网和内网，NAT

通常我们将从Nic申请的IP地址称为公网地址，这个公网地址资源有限。通常一个公司只有一个会几个公网IP地址。

公司内部，有自己的局域网，可以分配给公司内的Host，这个地址是内网地址，只对公司这个局域网有效。

但是，公司内的Host要同外部的Host通信怎么办呢？

NAT（Network Address Translation)是这个问题的主流解决方案。

基本的NAT实现的功能很简单，在子网内使用一个保留的IP子网段，这些IP对外是不可见的。

子网内只有少数一些IP地址公网IP。

如果这些节点需要访问外部网络，那么基本NAT就负责将这个节点的子网内IP转化为公网IP然后发送出去。

    
如3.1.1所述，我们知道IP地址分成网络地址和host地址,网络地址相同的host，可以互相通信。

网络地址不同的host，必须通过route。

网络地址相同的host，要通信，需要使用L2提供的能力，而L2需要Mac地址。

ARP协议解决了IP地址到Mac地址的映射问题，Host A不知道Host B的Mac地址，可以广播ARP报文，Host B收到报文需要回复自己的Mac地址。

这样Host A就知道了Host B的Mac地址，可以使用L2的功能来发送IP数据报。

如果Host C和Host A网络地址不一样，那么Host A会把报文发给路由器RouteA，RouteA需要选择一个能到达目的网络地址的路由（比如RouteX)，把IP数据报

发给它（L2的能力，Mac地址)，RouteX收到这个IP数据报，同样选择一个能到达目的网络地址的路由（这里假设RouteX可以到达RouteC)，把IP数据报发给它，RouteC收到数据报以后，

发现是自己网段的地址，发送给HostC。

在这个模型里面,host需要实现 1)判断是不是同一个网段 2) 实现ARP协议，能够将IP地址转化成Mac地址 3)  

## 3.2 分片

当在一个允许大包大小的internet数据报的局域网络上产生,且必须在允许包大小为较小的局域网络上传输的时候,分片是必须的。

IP协议提供了分片和重组的能力。

internet分片和重组程序需要能够将一个数据报分割成任意数量的块,这些块可以在之后重组。分片接收者使用identification头部来确保不同的分片不被混在一起.

分片偏移(fragment offset)头部告诉接收者分片在原始数据报中的位置。分片偏移和分片长度(frament length)确定了这个分片所覆盖的原始数据报的块。more-fragments标志(通过重置)指示了最后一个分片。这些头部提供了足够的信息来重组数据报.

## 3.3 协议数据格式

>>![](/assets/media/network_ip_spec.jpg) 

控制位（Control Bits）：6 bits（从左到右）：

URG：紧急指针字段有效（Urgent Pointer field significant）

ACK：确认头部字段有效（Acknowledgment field significant）

PSH：强制函数（Push Function）

RST：重置连接（Reset the connection）

SYN：同步系列号码（Synchronize sequence numbers）

FIN：再没有来自发送者的数据（No more data from sender）

# 4. TCP

TCP是一个`基于连接`的，host to host的`可靠`协议。TCP利用了IP协议的能力，提供了以下基本功能：

`基本的数据传输`

`可靠性`

`流控`

`多路(Multiplexing)`

`连接`

`优先级和安全性`

## 4.1 TCP头部格式

>>![](/assets/media/network_tcp_header_format.jpg) 

## 4.2 可靠通信

在一个TCP连接上的数据流可以可靠的顺序地投递到目的地。

TCP通过确认机制和重传机制来保证可靠性。

每八位字节数据分配一个系列号码(seq)。一个分片里面的第一个八位字节数据同分片一块传输，被称为分片系列号码。

分片也携带一个确认号码(ack)，该确认号码是在相反方向的下一个期望的传输八位字节数据的系列号码。

确认机制是累积的，所以序号X的确认指示的是所有X之前但不包括X的数据已经收到了。


>>![](/assets/media/network_tcp_ack2.png) 

丢包的情况

>>![](/assets/media/network_tcp_ack.png) 

当TCP传输一个包含数据的分片的时候，他将该数据分片的拷贝放在重传队列中，然后开始一个定时器，当数据的确认收到的时候，该分片拷贝从队列中删除，如果在定时到达之前没有收到确认，分片被重传。这个定时器

>>![](/assets/media/network_tcp_timeout.png) 

在超时重传中，定时器超时了才认为发送的数据包丢失。

快速重传机制，实现了另外的一种丢包评定标准，即如果发送方连续收到3次dup ACK，发送方就认为这个seq的包丢失了，立刻进行重传。

对于接收方来说，收到了out-of-order的packet，就发送dup ack。

## 4.3 连接建立和清除

source ip/source port/dest ip/dest port 唯一标识一个TCP连接。

建立连接的过程利用了SYN标志，包含了三条消息的交换。这个交换被称为三步握手。


>>![](/assets/media/network_tcp_handshake.png) 

清除连接的过程利用了FIN标志，需要四步

>>![](/assets/media/network_tcp_close.png) 

## 4.4 滑动窗口

`TCP通过滑动窗口实现了流控特性`。

TCP为接收者提供一个办法让其控制发送者发送的数据的数量。这是通过在每个ACK中返回一个窗口（“window”,见4.1TCP头部格式）来指示超过最后成功接收的一个分片的可接受的系列号码的
范围。

窗口指示了发送者在接收到进一步的允许前可以传输的字节的数量。

>>![](/assets/media/network_tcp_flowcontrol.jpeg) 

## 4.5 多路(Multiplexing)
为了允许在一个单独的主机里多个进程同时使用TCP通信机制，TCP提供了一套地址和端口。从internet通信层同网络和宿主地址连接，这形成了一个socket。

一对socket标识了一个连接。

# 5. UDP

# 参考文档
## 802.11无线网络权威指南
## [RFC791](https://tools.ietf.org/html/rfc791)
## [RFC793](https://tools.ietf.org/html/rfc793)

关键特性 1）局域网 2）无线（可以移动)
