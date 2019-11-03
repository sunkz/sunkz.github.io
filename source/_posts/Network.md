title: Network
author: Sunkz
tags:
  - tcp/ip
  - http
  - dns
photos:
  - 'https://s2.ax1x.com/2019/10/09/u4HkM4.png'
categories: []
date: 2018-09-19 12:34:00
---
# 基础知识

## 运输单元

![uhMD9P.png](https://s2.ax1x.com/2019/10/08/uhMD9P.png)

## OSI参考模型

![uhQAUA.png](https://s2.ax1x.com/2019/10/08/uhQAUA.png)

![uhQmgf.png](https://s2.ax1x.com/2019/10/08/uhQmgf.png)

## TCP/IP模型

![uhQWrD.png](https://s2.ax1x.com/2019/10/08/uhQWrD.png)

![tcp](http://tva1.sinaimg.cn/large/0060lm7Tly1g4yn6x4izlj30fd0hzmyl.jpg)

# TCP/IP详解

## 数据链路

### 局域网(LAN)

- WLAN(无线局域网)
- 以太网(MTU 1500字节),FDDI(MTU 4352字节),PPP,ATM

- 集线器(Hub):多端口中继器(Repeater),连接网络扩大网络传播范围

- 拓扑结构

  ![uf5YlR.png](https://s2.ax1x.com/2019/10/08/uf5YlR.png)

### 以太网(Ethernet)

- 最普遍的局域网技术

- 现代以太网结构

  ![ufIe4e.png](https://s2.ax1x.com/2019/10/08/ufIe4e.png)

#### 以太网帧格式

![ufh75Q.png](https://s2.ax1x.com/2019/10/08/ufh75Q.png)

#### 以太网交换机

- 数据链路层,多端口网桥,二层交换机
- 能识别MAC地址,可以从一个端口接受数据根据目的mac地址将数据发送到指定端口
- 此处的端口与TCP/UDP传输层协议中的端口含义不同

![ufgTLF.png](https://s2.ax1x.com/2019/10/08/ufgTLF.png)

#### 环路检测技术

- 采用生成树算法可避免帧在环路中被不停的转发

![uff4k4.png](https://s2.ax1x.com/2019/10/08/uff4k4.png)

## IP协议

#### IPv4协议头

![IP-header](http://tva1.sinaimg.cn/large/0060lm7Tly1g4yn7iipshj30h70aqdhr.jpg)

#### 地址分类

​    分类          网络号长度   首尾网络号

- A   0|000  ——  8 	  1-126
- B  10|00   ——  16     128.0 - 191.255
- C  110|0   ——  24      192.0.0 - 223.255.255
- D  1110     ——  32      224-239  多播地址,无主机号

#### 私有地址

| 始          | 末              |
| ----------- | --------------- |
| 10.0.0.0    | 10.255.255.255  |
| 172.16.0.0  | 172.31.255.255  |
| 192.168.0.0 | 192.168.255.255 |

#### 数据包分片

![uhKJds.png](https://s2.ax1x.com/2019/10/08/uhKJds.png)

#### MTU发现机制

![u4MNZQ.png](https://s2.ax1x.com/2019/10/09/u4MNZQ.png)

## IP相关协议

### ARP协议

#### 同一网络

```txt
请求帧 目的MAC : FF-FF-FF-FF-FF-FF  交换机转发到目的IP
应答帧 返回MAC地址 , 双方建立 IP<->MAC 的映射
```

#### 不同网络

```txt
路由器判断源IP和目的IP是否属于同一网络, 不属于同一网络就转发到下一跳
```

#### ARP缓存

```
arp -a 查看ip-mac映射cache
```

#### ARP攻击

```txt
中间人的IP与ARP请求帧的目的IP不同 , 但却将自己的MAC地址响应给了源IP
```

### DHCP协议

#### DHCP DISCOVER

```
目的MAC : FF:FF:FF:FF:FF:FF   源MAC  : 自身MAC
目的IP  : 255.255.255.255     源IP   : 0.0.0.0
目的端口 : 68                  源端口  : 67
```

#### DHCP OFFER

```txt
DHCP server 检查数据包 发现 源IP是0.0.0.0 目的IP是255.255.255.255 说明是DHCP DISCOVER数据包
则把IP和其它参数(子网掩码,GATEWAY,DNSSERVER)发送给DISCOVER数据包的MAC地址
```

### NAT协议

#### 分类

- 静态转换StaticNAT  **1:1**

  一对一,只将内网中特定的一台服务器通过一个外网ip映射出去

- 动态转换Pooled NAT  **m:n**

  类似数据库连接池或者线程池 , 当某个内网ip想要映射到外网 , 就从NAT地址池中拿一个外网IP , 用完即释放 . 多用于外部地址略少于内部地址时使用.

- 端口转换  **n:1**

  - SNAT (SourceNAT)

    192.168.1.24:13244 发送给外网时  **修改源IP:PORT** 201.12.1.1:13244  发送给  201.12.1.13:8080

    201.12.1.13:8080 响应时  目的IP为201.12.1.1:13244   再转为内网  192.168.1.24:13244

  - DNAT (DestinationNAT)

    内网有多个服务 192.168.1.24:8080 192.168.1.25:8080 192.168.1.26:8081 时

    外网发送的数据到 192.168.1.1:80  , 然后**修改目的IP:PORT**  使其**均衡负载**到内网不同服务器上 

#### 应用

- **解决IP不足**: 通过NAPT端口复用达到m:1
- **数据伪装:** 隐藏内网拓扑结构，使用全局地址替换私有地址。
- **端口转发:** 外网请求利用NAT可轻松转发到内网
- **负载平衡:**DestinationNAT可将外部请求落在不同的服务器上
- **失效终结:**检测到服务器宕机,将请求转发到其它或备用服务器上
- **透明代理:**例如自己架设的服务器空间不足，需要将某些链接指向存在另外一台服务器的空间；或者某台计算机上没有安装IIS服务，但是却想让网友访问该台计算机上的内容，这个时候利用IIS的Web站点重定向即可轻松的帮助我们搞定。

#### 穿透

NAT影藏了内部拓扑结构 , P2P(Peer2Peer)需要端到端的连接 , 故需NAT穿透

## TCP/UDP

### UDP

```
DNS,SNMP,视频音频,广播通信使用
```

![u4cAAJ.png](https://s2.ax1x.com/2019/10/09/u4cAAJ.png)

### TCP

![u4cm1x.png](https://s2.ax1x.com/2019/10/09/u4cm1x.png)

#### 滑动窗口

```
停止等待 累计确认 选择重传
```

#### 拥塞控制

```
慢开始  拥塞避免  快速重传  快速恢复
```

![拥塞控制.png](http://tva1.sinaimg.cn/large/0060lm7Tly1g4ynohjxqmj31m20pu7wh.jpg)

#### 三次握手

```
三次握手 : 双方互相确定 Seq Num. Seq Num如果是从1开始,则十分容易预测,所以需要随机设定并相互告知.
```

![handshake.png](http://tva1.sinaimg.cn/large/0060lm7Tly1g4ynowksbcj316y0s87fu.jpg)

#### 四次挥手

```
任何一方都可主动关闭连接
```

![四次挥手.png](http://tva1.sinaimg.cn/large/0060lm7Tly1g4ynp9k0tij31c40u0h67.jpg)

#### Wireshark 

```
curl http://sunkz.top    ||  curl http://106.12.194.111
三次握手 -- keepalive -- 四次挥手
```

![HTTP](http://tva1.sinaimg.cn/large/0060lm7Tly1g4ynpm7p50j318o08xq3u.jpg)

#### SYN flooding

```
Client不进行第三次握手 , Server进行重试第二次握手 , Servr处于Busy状态. 大量Client攻击时,Server将资源耗尽一直处于空忙碌状态.
```

### SCTP

[SCTP cookie 解决 SYN flood](https://baike.baidu.com/item/SCTP/5867122?fr=aladdin)

### CDN

#### RR(Round-Robin)-DNS LoadBalance

```txt
client --> DNS Server --> RR WebSever IP
DNS级别的Load Balance轮询响应IP,不能判断IP与客户端的远近
```

#### CDN (Content Delivery Network)

```txt
client -> a.com -> LDNS -> 网站授权DNS -> 返回a.com的CNAME a.bc.com的IP -> LDNS ->
DNS调度系统 -> 返回离 client 最近的边缘节点 IP -> client -> 边缘节点 -> 内容不存在 -> 边缘节点请求源站
```

![](http://ww1.sinaimg.cn/large/006tNc79gy1g58v49p7zwj30w60ni79k.jpg)

### DDoS

- Syn Flood
- UDP Flood
- ICMP Flood
- Smurf Flood
- CC
- Slowloris
- DNS Query Flood
