title: RocketMQ In 5 Minutes
author: Sunkz
tags:
  - rocketmq
photos:
 - https://s2.ax1x.com/2019/10/16/KFNeZF.png
categories: []
date: 2019-10-16 17:47:00

---
### 基本概念

```
Topic : 一级标识   consumer.subscriber("Topic","Tag1 || Tag2",()->{})
Tag : 二级标识
MessageID : RocketMQ 自动生成全局唯一
GroupID : 一类 Producer 集群 或 Consumer 集群
Queue : 每个 Topic 默认有4个队列
```

### 集群消息

```
一个 GroupID 的 Consumer 平均分摊 Topic 上的 9条消息
```

### 广播消息

```
一个 GroupID 标识的 Consumer : 分别消费 Topic 的所有消息
```

### 分组消费

```
同一个Topic在不同的Group之间是集群还是广播互不影响
多个Group订阅同一个Topic,则多个Group都可消费该Topic中的消息
```

### 死信队列

```
Consumer 消费失败会进行重试, 当达到最大重试次数后, RocketMQ 不会将这条消息丢弃, 而是放在 Consumer 的死信队列中(Dead-Letter Queue).
```

### 发送消息

- 普通消息

  ```
  同步发送 : send and wait for response 
  异步发送 : send and register SendCallback
  单向(oneway)发送 : send without response
  ```

- 定时,延时消息

- 顺序消息

- 事务消息

  ```
  两部确认机制
  ```


### 消息消费

```
Push 模式 : consumer 设置 Listener 回调(异步), Broker 主动将消息推送给 consumer
Pull 模式 :	consumer 主动从 Broker 拉消息
```

### 消息重试

```
消息队列 RocketMQ 默认允许每条消息最多重试 16 次

```

### 存储结构

![image-20190812135141891](http://ww2.sinaimg.cn/large/006tNc79gy1g5wumavkvkj316u0tonf4.jpg)

```
Topic的所有消息都存在CommitLog中, MessageQueue只存储CommitLog中某条消息的逻辑索引位置.

```

### 同/异步刷盘

- RocketMQ 的消息是存储到磁盘上的，这样既能保证断电后恢复，又可以 让存储的消息 量超出内存的限制

- 异步刷盘

  ```
  在返回写成功状态时 ，消息可能只是被写人了内存的 PAGECACHE ，写操作的返回快，吞吐量大;当内存里的消息量积累到一定程度时，统一触发写磁盘动作，快速写人。
  
  ```

- 同步刷盘

  ```
  写人磁盘时,才返回写成功状态.
  
  ```

### 同/异步复制

```
同步复制方式是等 Master 和 Slave 均写成功 后才反馈给客户端写成功状态, 更加可靠
异步复制方式是只要 Master 写成功即 可反馈给客户端写成功状态, 性能更高

```

### 消息幂等

```
消息重复 : 发送时消息重复,投递时消息重复,负载均衡时消息重复
处理 : 不建议以 Message ID 作为处理依据。 最好的方式是以业务唯一标识作为幂等处理的关键依据
			message.setKey("ORDERID_100");

```

### 部署架构

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4yn8ppm0oj316e0nsjwh.jpg)

- Name Server: 存放 Topic—Broker 映射.
- Broker：将 Topic-Broker 上报 Name Server . 默认 30s 上报一次.
- Producer：从 Name Server 获取 Topic 对应的 Broker 的关联信息, 将 Msg 推送到 Topic 对应的 Broker.
- Consumer： 从 Name Server 获取 Topic 对应的 Broker 信息, 并从 Broker 中获取该 Topic 的 Msg.

### 订阅模式

![sub-pub.png](http://tva1.sinaimg.cn/large/0060lm7Tly1g4ynnogh7jj31lp0u0al7.jpg)

