title: DNS
author: Sunkz
tags:
  - dns

photos:

- https://s2.ax1x.com/2019/11/02/Kb2aut.png

categories: []
date: 2019-04-09 01:46:00

---
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


