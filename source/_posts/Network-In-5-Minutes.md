title: Network In 5 Minutes
author: Sunkz
tags:
  - tcp/ip
  - http
  - dns

photos:

  - https://s2.ax1x.com/2019/10/09/u4HkM4.png

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

### DNS协议

#### Resolving Process

Browser Cache

> chrome://net-internals/#dns

OS Cache

> ipconfig /displaydns

HOSTS

> C:\Windows\System32\drivers\etc\HOSTS

ISP Cache

> ISP(Internet Service Provider) DNS cache

Real Resolve

> DNS Protocol

#### DNS - LB

> 基于DNS的负载均衡  : 域名--IP   1:n
>
> 比如 : Round Robin DNS 

#### Record Type

- A : 正向解析 域名 -> IP
- NS : 指定域名DNS服务器
- MX : 指向邮件服务器
- CNAME : 别名,指向另一个域名
- AAAA : 指向 IPv6

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

## HTTP协议

### HTTP

#### C/S

```
Browser --->  tomcat,jetty,netty...
 ↓                   ↓      
Socket  --->  SocketServer
```

#### Request

```
URI : abc://username:password@example.com:123/path/data?key=value&key2=value2#fragid1
URL : https:// --------- 

Method  :  GET HEAD(只返回请求头不返回消息体) POST(创建) DELETE(删除) PUT(更新) OPTIONS(预请求,获取服务端支持的Request Method)

HOST : 域名
Referer : 从哪个url跳转过来的, 可用于防盗链 , CSRF攻击
```

#### 攻击

- [XSS](https://www.cnblogs.com/phpstudy2015-6/p/6767032.html) [CSRF](https://www.cnblogs.com/shytong/p/5308667.html) [DDoS](http://baijiahao.baidu.com/s?id=1601066118227360188&wfr=spider&for=pc) [SQL注入](https://www.cnblogs.com/pursuitofacm/p/6706961.html)

#### 会话管理

```
Session  Set-Cookie : JSessionID= --> Browser --> JSessionID --> Server
Cookie   Set-Cookie : name=skz --> Browser --> Cookie --> Server 
```

#### CORS

```
浏览器允许<img src="http://"> <script> <link>等标签跨域 , 但不允许ajax跨域请求
ajax跨域时Server的response header中若没有 Access-Control-Allow-Origin:"域",浏览器会阻止
跨域默认允许的Method GET HEAD POST , 其它方法需要与请求验证是否允许
允许的Content-Type text/plain multipart/form-data multipart/x-www-form-urlencoded,请他Content-Type也需要预请求
预请求RequestMethod OPTIONS,服务端设置允许的Access-Control-Allow-Headers:"token" 和 Access-Control-Allow-Methods:"DELETE,PUT"
```

#### 缓存

**Cache-Control**

```
public 允许中间代理节点缓存
private 只有发请求的地方才能缓存
no-cache 可以缓存,但是每次请求不能直接使用缓存, 而是要去服务器验证缓存是否有效
no-store 不能使用缓存,每次请求都要从服务器拿新的内容
max-age 缓存到期时间
```

**Etag Last-Modified**

```txt
Cache-Control设置为no-cache时,客户端可以缓存
但下次请求仍要去服务器校验 Etag Last-Modified 判断缓存是否修改
返回 304  not modified 未修改 , 可使用缓存
返回 200  修改后的内容 , 客户端重新缓存
```

#### Cookie

```
max-age expires 过期时间
Secure https时才会发送cookie
domain a.skz.com  skz.com cookie共享范围
Session是服务端状态保持标记
```

#### 长连接

```
Connection : keep-alive 一个TCP连接可以发送多个HTTP请求,多个HTTP请求需同域
Connection : close 一个TCP连接只能发送一个HTTP请求
```

#### 请求协商

```
Accept -- Content-Type  : 
Accept-Encoding -- Content-Encoding
Accept-Language -- Content-Language
User-Agent : 告诉服务端自己的设备类型
```

#### Status Code

```
1xx 信息状态码,请求接受正在处理
200 success
301 永久重定向 重定向地址可以缓存在客户端,下次请求直接从客户端拿重定向地址
302 临时重定向 服务端返回 Location 重定向地址 , 客户端下次请求仍到原服务器
304 not modify , 可以使用本地缓存
400 请求格式错误
401 未授权(Unauthorized)
403 Forbidden
404 not found
500 Internal Server error
504 Service Unavailable
```

#### Basic Authorization

```
username:password 被浏览器用base64解密后 放在 header Authorization 
```

#### 版本

- HTTPS

```
443 ssl 证书 非对称性加密 ...
@See sunkz.github.io/Basic/HTTPS
```

- HTTP1.X

```
域名分片 : 一个域名浏览器只能创建6-8个 TCP 连接, 所以通过子域名(二级)来解决.
TCP建立 : 经过 DNS, 三次握手 , 慢启动. 相当占用资源
pipelining : TCP 采用应答模式相当耗时, 采用管线化,多个TCP请求批量发送
文本传输 : 对机器不友好 ,占用带宽
长连接 : 在一个 TCP 连接中, 多次 请求/响应.
```

- HTTP2

```
头部压缩算法 : HPACK  header 语义不变 语法从 文本改为 二进制
二进制分帧 : header + body 被差分成 frame 流. 模仿 TCP 每个 Frame 都带 id 在接受端进行 Frame 的组装
多路复用 : HTTP/2 复用一个 TCP 连接传送多个流的 Frame 片, 在接收端分用. HTTP/2 添加了控制帧管理流
```

![img](http://ww2.sinaimg.cn/large/006tNc79gy1g61o1958u6j30yc0a63yt.jpg)

![WX20190624-195355@2x.png](http://tva1.sinaimg.cn/large/0060lm7Tly1g4yn5xtatrj32000hual1.jpg)

```
多路复用(MultiPlexing) : 在 HTTP1.x 中一个TCP连接中只能传输一个流,一个请求就是一个流.在HTTP2中,一个TCP连接中可以传输多个 stream,多个流加上stream identifier复用为一个流在该TCP连接中进行传输, 在接收端根据stream identifier进行分成多个流.实现了单TCP连接的情况下,多请求-响应并行.
同一域下,使用一个TCP连接传输所有HTTP请求,减少TCP连接建立的损耗
此多路复用区分与 HTTP1.x 的长连接
```

```
Server Push
```

```
重置连接 : 可以再不中断 TCP 的情况下, 单独中断某个 stream
```

```
请求优先级 : 每个 stream 可以设置 依赖和权重.
```

```
流量控制 : 在应用层对每个 stream 做流量控制, 动态变更 stream 传播速率
```



- HTTP3

- [QUIC原理](https://blog.csdn.net/Tencent_TEG/article/details/79158266)

![img](http://ww3.sinaimg.cn/large/006tNc79gy1g5z0ed23coj30u01madzx.jpg)

![img](http://ww3.sinaimg.cn/large/006tNc79gy1g5z1rly2g7j30u010lqd1.jpg)

### HTTPS

#### 对称加密

```
A -> 文件 -> 放入密码箱，秘钥加密 -> B -> 拿到密码箱，秘钥解密 -> 文件
存在的问题 : Ａ如何将密码箱的秘钥安全的告诉Ｂ

DES、AES-GCM ...
```

#### 非对称加密

```
公钥加密,私钥解密
私钥签名,公钥验证
用公钥加密的数据只有对应的私钥可以解密
用私钥加密的数据只有对应的公钥可以解密
如果可以用公钥解密，则必然是对应的私钥加的密
如果可以用私钥解密，则必然是对应的公钥加的密
```

```txt
A -> 文件 -> 用B的公钥加密 -> B -> B用自己的私钥解密 -> 文件
B -> 文件 -> 用A的公钥加密 -> A -> A用自己的私钥解密 -> 文件
存在的问题 : 非对称性加密算法加密解密时间比对称加密算法长的多

RSA、DSA、ECDSA ....
```

#### 非对称+对称加密

```txt
B -> 设置密码箱秘钥 -> 用A的公钥对秘钥加密 -> A -> A用A的私钥解密,得到密码箱的秘钥
用非对称性加密算法传递对称加密的秘钥, 之后A-B互相通信就用对称加密进行通信

存在的问题 : 中间人X的攻击 , B没办法知道拿到的公钥确实是A的公钥
A给B发送公钥时被X截取,X将A的公钥替换为自己的公钥发送给B, B用X的公钥加密,X截取B发送的内容用X的私钥解密查看或修改内容后,X用A的公钥加密内容发送给A.
```

#### 信息摘要

```txt
A将自己的公钥及个人信息计算信息摘要后一并发送给B,B收到消息后计算信息摘要进行对比看收到的消息是否被中间人篡改

存在的问题 : 中间人X , 可能把信息摘要都一并篡改发送给B
即B没办法知道收到的信息摘要一定是A的消息的消息摘要

MD5、SHA-1、SHA-2、SHA-256
```

#### 认证中心

```txt
A将自己的公钥及个人信息　->　计算信息摘要 -> 信息摘要用CA私钥加密 -> 数字签名
A的公钥及个人信息 + 数组签名 -> A的数字证书
A将自己的数字证书 -> B -> B将数字证书中A的公钥及个人信息计算信息摘要
                  B -> B用CA的公钥对数字证书的数字签名解密得到信息摘要    进行对比

操作系统／浏览器中内置了CA的数字证书
```

#### HTTPS

```txt
B 用 https 访问 A 时, A 首先将自己的数字证书发送 B
B 内置的CA数字证书中对发送过来的数字证书进行验证,查看其是否真的是A的数字证书
B 验证通过后,生成 秘钥(对称加密的秘钥) , 将秘钥用A的数字证书中A的公钥进行加密 , 发送给Ａ
A 收到后用A的私钥进行机密到的到 秘钥(对称加密的秘钥)
A-B 之间随后通过对称加密进行通信

上述过程是TLS(SSL)的简版
Client-Hello   Server-Hello
```

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
