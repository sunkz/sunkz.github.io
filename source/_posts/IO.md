title: IO
author: Sunkz
tags:
  - io
photos:
  - 'https://s2.ax1x.com/2019/12/30/lQ8KcF.png'
categories: []
date: 2019-12-30 17:12:00
---
阻塞IO

```
等到数据在内核空间准备好才返回,用户再将数据从内核Copy到用户地址空间
```

非阻塞IO

```
直接返回,用户主动轮询是否准备好.内核准备好后,将数据从内核Copy用户地址空间.
```

IO多路复用

```
复用 : 单个线程跟踪多个IO Stream状态.是对多线程(进程)非阻塞IO模型的优化.
节省了资源的同时,少了线程(进程)创建销毁的开销.
```

```
select : 上限1024;数据准备好后,仍需用户去轮询哪个IO Stream准备好,用户再将数据从内核Copy到用户地址空间.
```

```
poll : 去除select的跟踪IO Stream数上限.
```

```
epoll : 可以告诉用户是哪个IO Stream上准备好数据.
```

异步IO

```
内核准备好数据,主动将数据从内核Copy到用户空间并通知用户.
```