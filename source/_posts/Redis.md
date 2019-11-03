title: Redis
author: Sunkz
tags:
  - redis

photos:

- https://s2.ax1x.com/2019/11/03/KX7Vij.png

categories: []
date: 2019-06-03 17:49:00

---
### 数据类型

- String

  ```
  SET name skz
  ```

- Hash

  ```
  HMSET person_1 name skz age 23;
  HGET person_1 age
  ```

- List

  ```
  LPUSH animals cat dog
  LRANGE animal 0 1
  ```

- Set

  ```
  SADD animal cat
  ```

- ZSet

  ```
  ZADD db 1 redis ZADD db 3 mongo ZADD db 2 mysql
  ZRANGE db 0 10 WITHSCORES
  ```

```
String  (k,v)
    存储token    k=userId,  v=token
    记录IP访问次数  k=ip  ,  v=count    IncryBy命令将v+1
Hash  (k,filed,value)
    存储用户信息  (userId,name,skz)  (userId,age,22)
    可以Hget(k,filed)获取单个字段,操作单个字段
List  (k,list)
   	push pop 模拟队列,最新消息排行
Set  (k,v)
    去重, 求交集,并集,共同好友.
Zset (k,score,v)
   以某一权重排名
```

### 数据结构

- 字典(dictionary)

  ```
redis的key就是存储在字典中的,类似hash,key的查找时间复杂度为o(1). 这也是redis快的原因之一.
  ```
  
- 简单动态字符串(Simple Dynamic String)

  ```
  c字符串是以\0结尾的字符数组 获取字符串长度的时间复杂度为o(1)
  sds 获取字符串长度的时间复杂为o(1)
  sds 根据free可以防止buf溢出
  ```

  ![image-20190829125004899](https://tva1.sinaimg.cn/large/006y8mN6gy1g6ggdpx231j317y0dcadp.jpg)

- 跳跃表

  ```
  空间换时间
  ```

### 持久化机制

```tex
RDB : save 父进程阻塞 ; bgsave fork出子进程进行备份
AOF : 记录所有指令 , 恢复时顺序执行指令
RDB无论怎么设置<seconds><changes>都可能发生部分操作无法保存 , 而AOF则相反
```

### 线程模型

```
主线程采用带线程模型 : IO事件的处理,过期key处理,集群协调.多个线程并发访问串行处理,减少切换.
I/O多路复用机制 : 底层封装了select poll epoll kqueue等函数,  针对不同操作系统选择不同的I/O复用机制
```

### 主从同步

```
1.全量同步
	1).slave 向 master 发送 sync .
	2).master 执行 bgsave 并将 RDB文件 发送给slave
	3).master 后续的操作日志(AOF文件 )增量发送给 slave 执行	
	4).一般情况下后续所有写操作在master执行,所有读操作在slave执行.
```

### 哨兵机制

```
由于主从同步后,master负责写操作,slave负责读操作.
当master挂掉后,就没办法进行写操作.所以需要一种机制,来从slave中选举出一个master.
```

```
sentinel.conf monitor master ip:port score #选举规则
sentinel down-after-milliseconds master 1000 #选举时机
```

```
Gossip反熵算法 : 在杂乱无章中寻求一致 
	每个节点都随机的同其他节点进行通信,最终所有节点的状态达成一致.
```

### 分布式锁

```java
//但一个线程设置成功,其他线程就将阻塞,直到成功的线程释放.
//设置时候应当设置过期时间,防止已经获得锁的线程出现异常无法释放锁.
if(redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK, REDIS_LOCK, Duration.ofSeconds(5){
	//业务逻辑
	redisTemplate.delete(REDIS_LOCK);
}
```

### 缓存淘汰策略

```txt
# how Redis will select what to remove when maxmemory is reached
volatile-lru -> Evict using approximated LRU among the keys with an expire set.
allkeys-lru -> Evict any key using approximated LRU.
volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
allkeys-lfu -> Evict any key using approximated LFU.
volatile-random -> Remove a random key among the ones with an expire set.
allkeys-random -> Remove a random key, any key.
volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
noeviction -> Dont evict anything, just return an error on write operations.
```

### 集群原理

```
实现均摊数据,数据分片,降低单节点压力,实现多机负载均衡
	求余取模,hash环,CRC16算法
```

- [Redis Cluster 原理分析](https://www.imooc.com/article/43576?block_id=tuijian_wz)
- [Redis Cluster Tutorial](https://redis.io/topics/cluster-tutorial)

### 其他问题

- 找出指定模式的key

  ```
  KEYS pattern : 匹配指定模式的key. 扫描所有key非常多时不适合,将会造成长时间阻塞
  ```

  ```
  SCAN cursor : 扫描部分key, 返回部分匹配的key
  ```


