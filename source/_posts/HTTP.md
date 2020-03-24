title: HTTP
author: Sunkz
tags:
  - http

photos:

-  https://s2.ax1x.com/2019/11/02/KbgOAS.png

categories: []
date: 2019-3-29 01:33:00

---
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

#### 虚拟主机

```
Header中Host字段,用以支持虚拟主机技术,默认为值url
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


