title: Wireshark
author: Sunkz
tags:
  - http
  - https

photos:

- https://s2.ax1x.com/2020/03/10/8iSpr9.png

categories: []
date: 2020-03-10 16:08:00

---
# 1. HTTP POST

### 1.1 Postman请求记录

![](https://cdn.shenlanbao.com/consultants/800322674_top_banner_750x330.png)

### 1.2  Wireshark抓包记录

##### 1.2.1  三次握手

![](https://cdn.shenlanbao.com/consultants/1611089354_搜狗截图20200310144449.png)

##### 1.2.2  请求与响应

![](https://cdn.shenlanbao.com/consultants/635333598_搜狗截图20200310144617.png)

##### 1.2.3  keepalive

![](https://cdn.shenlanbao.com/consultants/180781462_搜狗截图20200310144810.png)

##### 1.2.4  四次挥手

![](https://cdn.shenlanbao.com/consultants/453199923_搜狗截图20200310144906.png)

##### 1.2.5  请求体

<img src="https://cdn.shenlanbao.com/consultants/782558937_搜狗截图20200310145152.png" style="zoom: 50%;"  />

##### 1.2.6  响应体

<img src="https://cdn.shenlanbao.com/consultants/724218821_搜狗截图20200310150319.png" style="zoom: 50%;" />

# 2. HTTPS POST

### 2.1  Postman请求记录

![](https://cdn.shenlanbao.com/consultants/648989139_搜狗截图20200310150651.png)

### 2.2 Wireshark抓包记录

##### 2.2.1  三次握手

![](https://cdn.shenlanbao.com/consultants/985524581_搜狗截图20200310151844.png)

##### 2.2.2  Client hello Server Hello & exchange public key

![](https://cdn.shenlanbao.com/consultants/167395833_搜狗截图20200310152452.png)

##### 2.2.3  请求响应加密传输

![](https://cdn.shenlanbao.com/consultants/1849811974_搜狗截图20200310152530.png)

##### 2.2.4  keepalive

![](https://cdn.shenlanbao.com/consultants/770501579_搜狗截图20200310152714.png)

##### 2.2.5  四次挥手

![](https://cdn.shenlanbao.com/consultants/306147674_搜狗截图20200310160409.png)

# 3. HTTP & HTTPS

- 发送方用接收方的公钥加密data发送给接收方

- 接收方接收发送方已加密的data并用自己的私钥解密
- Client 和 Server 互为发送方和接收方