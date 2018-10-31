---
layout: post
title: "网络基础"
date: 2018-10-24 
author: "WangX"
catalog: true
header-style: text
tags:
    - network
    - linux
    - basic
---
>update 2018-10-27    
update 2018-10-31

## OSI七层协议
>OSI: Open System Interconnection


!["OSI七层模型"](/img/network/OSI.png "OSI七层模型")
上图中的OSI七层协议：
1. 物理层：只能传输0和1这种比特信息，必须定义所使用的传输设备的信号，同时必须了解数据帧转成比特流的编码方式
2. 数据链路层：该层介于物理层和网络层之间，比较特殊。向下与物理层交互，偏硬，主要负责的是MAC(Media Access Control),这个数据包称为MAC数据帧，
MAC是网络接口设备所能处理的主要数据包裹，也是最终被物理层编码成比特流的数据。向上与网络层交互，偏软。主要指逻辑链接层(Logical Link Control,LLC)所控制，
主要在多任务处理来自上层的数据包数据(packer)并转换成MAC的格式，负责的工作包括信息交互，流量控制，失误问题的处理。
3. 网络层：IP(Internet Protocol)在这一层定义。同时也定义出计算机之间的连接建立，终止与维持等，数据包的传输路径选择等。最重要就是IP和路由(route)的概念。
4. 传输层：定义了发送端和接收端的连接技术(如 **TCP、UPD**技术)，同时包括该技术的数据包格式，数据包发送，流程的控制、传输过程的侦测检测与重新传送等，以确保
各个资料数据包可以正确无误的到达目的端。
5. 会话层：主要定义了两个地址之间的信道连接与中断，建立应用程序之间的会话，提供其他加强型服务：网络管理，建议与断开，会话控制等。以确保网络服务建立连接的确认。
6. 表示层：将来自本地应用程序的数据格式转换成为网络的标准格式，然后交给下面的传输层等的协议来进行处理。主要是数据格式的转换，包括数据的加解密也是在这个层次上处理。
7. 应用层：并不属于应用程序，定义应用程序如何进入该层的沟通接口，从而将数据接收或发送给应用程序，最终展示给用户。

>OSI七层模型是一个参考的理想模型，实际应用中则是以TCP/IP协议族的四层模型居多。

## TCP/IP四层协议
如下图所示：TCP/IP将应用、表示、会话三层整合成一个应用层，传输层没有变，且依据传送的可靠性又将数据报格式分为面向连接的TCP和无连接的UDP包格式。网络层没有变，主要是IP数据包和
路由功能。数据链路层和物理层则整合为一个网络接口层，包括定义硬件信号、数据帧转换为比特流的编码等，主要与硬件有关。
!["TCP/IP"](/img/network/TCP-IP.png "TCP/IP协议")

## TCP/IP的网络接口层的相关协议

#### 广域网

>传输距离较远，连接介质低廉，网络速度较慢且可靠性较低。

* 传统电话拨号连接，使用调制解调器，电话线及计算机的九针串行端口。通过 **PPP(Point-to-Point Protocol)** 协议，速度比较慢，拨号连接建立后就不能使用电话。PPP协议支持TCP/IP、NetBEUI、IPX/SPX等通信协议。
* 整合服务数字网络(Integerated Services Digital Network,ISDN)。电话线加两端调制解调器。ISDN的传输有多种信道可供使用，可以将多个信道整合应用，速度也成倍增长。
* 非对称数字用户环路(Asymmetric Digital Subscriber Line,ADSL)。电话线拨号获取IP，使用电话的高频部分，可以边上网边通话，由于上下行带宽不同，称为非对称环路。使用PPPoE，将PPP仿真在以太网卡上。
* 电缆调制解调器(Cable Modem)将有线电视的缆线作为网络信号介质。

#### 局域网(以太网为例)

>网络世界并非仅有以太网这个硬件接口

* 数据传输的单位为每秒多少bit,即bits/second(b/s)。而一般常用文件容量为bytes。1byte=8bit

以太网的传输协议：**CSMA/CD**

>CSMA/CD(Carrier Sense Multiple Access with Collision Detection) 载波监听多点接入/碰撞检测

以太网的传输主要就是网卡与网卡之间的数据传输。网卡在出厂时被赋予一个唯一的卡号，就是所谓的MAC(Media Access Control)地址。
网卡之间的数据传输通过IEEE 802.3的标准CSMA/CD技术。
* 集线器是一种网络共享介质设备，网络共享介质在单一时间点内，仅能被一台主机使用。

考虑一下场景：A B C D四台主机连接在一个集线器上。A的网卡要给D的网卡发送数据，则CSMA/CD的流程如下：
1. Carrier Sense: A网卡发送数据前，先对网络的使用情况进行监听，没有其他网卡使用后，才能够发送数据帧。
2. Multiple Access:A网卡发送的数据会被集线器复制一份，然后发送给所有连接此集线器的设备。A所发送的数据，B,C,D都能收到。但由于目标是D，B和C则会将此数据帧丢弃，而D则会处理。
3. Collision Detection: 该数据帧具有检测能力，若A和B刚好同时发送数据帧，造成Collision，则这两个数据帧被损毁，那么A与B就会各自随机等待一段时间，然后重新通过第一步再一次发送改数据帧。

* 标准的数据帧在网卡与其他以太网媒体一次只能传输 **1500bytes**，因为大文件会被拆分成多个小数据报，然后一个个传送，每个数据包发送前都要经过CSMA/CD机制。
* 超高速的以太网媒体若支持 **Jumbo frame(巨型数据帧)**，则可将数据帧的大小改为 **9000bytes**。

#### MAC的封装格式
>MAC帧是整个网络硬件上面传送的最小单位

!["MAC数据帧"](/img/network/mac_frame.png "MAC数据帧")

上图为MAC的帧结构：
1. 目的地址和源地址指的就是网卡卡号，MAC地址。6字节中前三个字节为厂商的代码，后面三个字节是该厂商自行定义的配置码。
2. MAC帧的传输如果跨过不同的子网，那么源地址的MAC地址会发生改变，因为变成了不通网卡之间的通讯了。只有在同一个子网内才能够直接进行MAC帧的收发。
3. 从LLC数据段可以看出数据容量最大可达1500bytes，最小的46bytes是由CSMA/CD机制算出来的，若要实现冲突检测，则数据帧总数据量最小需要有64bytes。再扣除目的地址，来源地址、校验码(前导序列不算)
就可得到数据量最小需要46bytes。若小于46bytes，系统会自动补上填充码，以补齐至少46bytes的容量。

* MTU(最大传输单位)。MAC帧的数据量最大可以为1500bytes,这个值被称为MTU。每种网络接口的MTU都不同，如有1492bytes的MTU。以太网的标准定义就是1500bytes。MTU的配置，要考虑MAC帧所经过的所有网络设备。

#### 集线器、交换器与相关机制

* 集线器(Hub)，网络共享设备，可能发生冲突，需要CSMA/CD。
* 交换器(Switch)，内部有内存区域，可以记录每个Switch port 与其连接设备的MAC地址，每个数据帧可以直接通过交换器的内存数据而传送到目标主机上。Switch不是共享设备，每个port都具有独立的带宽。

## TCP/IP的网络层的相关数据包和数据

#### IP数据包的封装

>这里介绍的是IPV4

下图的为IP数据包的包头资料：

!["IP数据包"](/img/network/IpPackage.png "IP数据包")

上图中每一行所占的位数为32bits

* Version，声明这个IP数据包的版本
* IHL(Internet Header Length,IP包头长度)
* Type of Service 这个项目的8位对应为“PPPDTRUU”。PPP：表示IP数据包的优先级，目前很少使用。D：0表示一般延迟，1表示低延迟。T：0表示一般质量传输，1表示为高质量。R：0表示一般可靠度，1表示高可靠度。UU：尚未被使用。
* **Total Length** 这个数据包的总容量，包括包头和数据部分，因为这个栏位功16个bit，所以能表示的最大值为65535bytes,也就是IP数据包可达的最大值。
* Identification(识别码)，由于IP数据包比MAC数据帧的要大，IP数据包需要被分成更小的IP分段后才能塞进MAC数据帧中，Identification用了标明每个小的IP分段是否来自同一个IP包。
* Flags(特殊字段)，对应内容为“0DM”。D：0代表可以分段，1表示不可分段。M：0代表此IP为最后分段，1表示非最后分段。
* Fragment Offset(分段偏移)：表示目前这个IP分段在原始IP数据包中所占位置。通过Total Length、Identification、Flags以及Fragment Offset就能够将小的IP分段在接收端组合起来了。
* **Time To Live** (TTL，生存时间)：8bit，范围0到255。当这个IP数据包通过一个路由器时，TTL就会减1，当TTL为0时，这个数据包将会被直接丢弃。
* **Protocol Number** 传输层和网络层本身的其他数据都是放在IP数据包中，这个字段记录了IP数据包内的数据是什么，如TCP、UDP和ICMP
* Header Checksum(报头校验码)，用于检查IP报头是否错误
* **Source Address** (来源IP地址)，32位
* **Destination Address** (目的IP地址)
* Options(其他参数)，额外功能，如安全处理机制，路由记录，时间戳，严格与宽松的来源路由等。
* Padding(补齐)，若Options不足32位时，由Padding主动补齐。

#### IP地址的组成与分级

IP地址是32位，有32个0或者1组成。IP的表示方式：将32bits分成四段，每段8bits并且转换成10机制。所以IP的范围是从 0.0.0.0 ~ 255.255.255.255。

>**32位的IP地址中，可以将32位分为Net_ID(网络号码)和Host_ID(主机号码)两个部分**

* **同一网络**：在同一物理网段内，主机的IP具有相同的Net_ID, 并且具有独特的Host-ID。
IP在同一网络的意义：
1. 在同一网段内，Net_ID是不变的，而Host_ID则是不可重复的。而且Host_ID在二进制的表示法中，不可全部为0也不可全部为1。全0表示整个网段地址(Network IP),而全为1则为广播地址(Broadcast IP)。
2. 主机可以通过CSMA/CD的功能直接在局域网内通过广播进行网络的连接，也就是MAC数据帧。
3. 即使同一物理网段，主机若不是同一网络，则广播地址不同，需要使用路由器来进行连接。
4. Host_ID所占用的位数越大，Host_ID则越多，表示同一网络内可用以设定的主机IP数量越多。

IP的分级，Inter NIC 将整个IP网段分为五种等级，等级的范围与32位的前几位有关：    
Class A: Net_ID开头为0,  十进制：0.xx.xx.xx ~ 127.xx.xx.xx     
Class B: Net_ID开头为10,  十进制：128.xx.xx.xx ~ 191.xx.xx.xx     
Class C: Net_ID开头为110,  十进制：192.xx.xx.xx ~ 223.xx.xx.xx    
Class D: Net_ID开头为1110,  十进制：224.xx.xx.xx ~ 239.xx.xx.xx   
Class E: Net_ID开头为1111,  十进制：240.xx.xx.xx ~ 255.xx.xx.xx      
Class D 是用来作为组播地址，Class E是没有使用的保留网段。一般系统上面只有A B C三种等级的IP。

#### IP的种类与获取方式

* IP的种类只有两种：
1. Public IP：公共IP，经由InterNIC所统一规划的IP，也就是常说的公网IP。
2. Private IP：私有或保留IP，主要用于局域网内的主机连接规划。
私有IP分别在A、B、C三个class当中各保留一段作为私有IP：
Class A: 10.0.0.0 ~ 10.255.255.255     
Class B: 172.16.0.0 ~ 172.31.255.255     
Class C: 192.168.0.0 ~ 192.168.255.255
私有IP只能再内部网络使用，若要连接公网需使用NAT(NetWork Address Transfer)服务。

* Loopback IP 网段，127.0.0.0/8 网段，默认的主机localhost的IP 是127.0.0.1
* IP的获取方式：
1. 手动配置 static
2. 通过拨号取得，ISP提供
3. DHCP

## Netmask、子网与CIDR(Classless Interdomain Routing)

>如何使得网段的分隔粒度更细

