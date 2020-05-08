title: MySQL
author: Sunkz
tags:

  - mysql

photos:

  - https://s2.ax1x.com/2019/11/02/KbyNwD.png

categories: []
date: 2018-12-02 00:24:00

---
## SQL

### 执行流程

```
SQL --> Cache --> parse -> pre --> optimization --> execute plan
```

### 查询流程

<img src="http://ww1.sinaimg.cn/large/006tNc79gy1g5k5izypjmj31400u0tpc.jpg" alt="img" style="zoom: 33%;" />

### 更新流程

<img src="https://s1.ax1x.com/2020/04/21/J8T8tx.png" alt="J8T8tx.png" style="zoom: 33%;" />

### 特殊处理

```sql
-- 自动更新时间戳
CREATE TABLE `ins_product` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `created_at` timestamp DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `deleted` tinyint(1) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

ALTER TABLE ins_product MODIFY created_at timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
ALTER TABLE ins_product MODIFY updated_at timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP

-- 单字段分类统计
SELECT   
count(if(is_read='0',true,null)) AS success,   
count(if(is_read='1',true,null)) AS fall   
FROM um_push_msg     
WHERE account='lic'

-- 双向触发器,解决死循环
CREATE TRIGGER sync_customer_phone_to_family_member_self 
AFTER UPDATE ON customer FOR EACH ROW
BEGIN
	SET @fm_slef_phone=(SELECT phone FROM family_member WHERE customer_id=new.id and relationship='SELF' and deleted=0);
	IF @fm_slef_phone!=new.phone THEN
	update family_member set phone=new.phone where customer_id=new.id and relationship='SELF' and deleted=0;
	END IF;
END;

CREATE TRIGGER sync_family_member_self_phone_to_customer
AFTER UPDATE ON family_member FOR EACH ROW
BEGIN
	SET @customer_phone=(SELECT phone FROM customer where id=new.customer_id and new.relationship='SELF');
	IF @customer_phone!=new.phone THEN
	update customer set phone=new.phone where id=new.customer_id and new.relationship='SELF';
	END IF;
END;

DROP TRIGGER customer_phone_to_family_member_self 
```

### 数据库设计

utf8mb4(emoji)    utf8mb4_unicode_ci(慢,准)  utf8mb4_general_ci(快,不准)

------

UUID(存储空间大,查询性能低)  自增ID(空间小,性能快)   自增ID+步长(也可应用于分分布式)

单实例—>自增ID    20节点—>UUID   20-200节点—>自增ID+步长   200+节点—>开源分布式ID解决方案

------

外键--增强数据一致性,但影响性能(数据库需要维护外键约束)

------

DATA+TIME = DATATIME ≈ TIMESTAMP(1970-2038)

TIMESTAMP 存储时间是UTC , 查看出来的时间将转成本机时区

TIMESTAMP时间显示错粗考虑MYSQL实例所在的docker容器的时区是否设置正常

------

price : POJO BigDecimal  db decimal

------

创建时间 timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,

修改时间 timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

------

逻辑删除---每次查询都要where and deleted=1

物理删除+转移表(删除的数据移到另一张表,触发器实现)——性能也有损耗,还会重建索引(保持树的平衡)

------

逻辑外键适合加索引

## MySQL

### 存储引擎

```
MyISAM  表锁
InnoDB  行+表锁，事务，外键	支持事务
```
### 日志系统

- redo log(重做日志) : InnoDB(存储层)特有的日志,循环写.

  ```
  WAL(write ahead logging) :当一条记录需要更新时,先把记录写到redo log,并更新内存,这个时候就算更新完成.等到合适(系统空闲)的时,在把值刷到内存.
  crash-safe : redo log 保证了InnoDB具有这个能力.它记录了每一条操作日志.
  ```

- binlog(归档日志) : server层日志, 所有存储引擎都可使用.只归档,不具有crash-safe能力.

- undo log : 用于事务回滚, MVCC.

### 事务机制

```
Automicity : 原子性	
Consistent : 一致性
Isolation : 隔离性, 事务处理的中间状态对外界不可见.
Duration : 持久性
```

```java
@Transactional
/** 底层AOP过程 **/
try{
 	transaction.begin();
    /**  **/
    transaction.commit();
}catch(e){
	transaction.rollback();
}
```

```java
LogService{
    @Transactional
    public void log(){}
}
UserService{
    @Transactional
    public void del(){
        logService.log();
        user.del();
    }
}
-----------------------------------------------------
/** 传播级别 **/
//默认  : 如果del()抛异常,则插入log()也会失败;log()将使用 del()的事务
@Transactional(propagation = Propagation.REQUIRED)
//del()失败,仍能插入log();log()仍使用自己的事务
@Transactional(propagation = Propagation.REQUIRED_NEW)
```

### 事务并发可能带来的问题

- 更新丢失(Lost Update)

```
当两个或多个事务选择同一行，然后基于最初选定的值更新时该行时，由于每个事务都不知道其它事务的存在，就会发生丢失更新问题-----最后的更新覆盖了由其它事务所做的更新。
解决：提交事务之前，另一个事务不能访问该文件
```

- 脏读(Dirty Reads)

```txt
事务A读到了事务B已修改但尚未提交的数据，还在这个数据基础上做了操作。此时，如果B事务回滚，A读取的数据无效，不符合一致性要求。
```

- 不可重复读(Non-Repeated Reads)

```
一个事务在读取某些数据后的某个事件，再次读取以前的数据，却发现其读出的数据已经发生了改变、或某些数据已经被删除了。
事务A读取到了事务B已经提交的修改数据，不符合隔离性。
```

- 幻读(Phantom Reads)

```txt
一个事务按相同的查询条件重新读取以前检索过的数据，却发现其它事务插入了满足其查询条件的新数据。
事务A读取到了事务B提交的新增数据，不符合隔离性。
```

### 事务隔离级别

- 未提交读(Read Uncommitted)

  ```
  在当前事务中读到其他事务修改且未提交的值.
  ```

- 已提交读(Read Committed)

  ```
  在当前事务中读到其他事务修改且已提交的值.
  ```

- 可重复读(Repeatable Read)(默认)

  ```
  在当前事务中读取不到其他事务修改的值,当前事务的值只受当前事务的影响.
  通过MVCC解决不可重复读, 通过间隙锁解决幻读.
  ```

- 可序列化(Serializable) 

  ```
  事务排序执行
  ```

![image-20190818210431770](http://ww3.sinaimg.cn/large/006tNc79gy1g644uhi6x6j31r40ogka1.jpg)

### 事务隔离机制的实现

- 可重复读隔离级别实现机制:  将一个值从1改成2,3,4过程生成的undo log.不同的时刻启动的事务会关联上不同的read-view.事务T启动时候关联上了额read-view A,即使其它事务改变了1的值.事务T始终读取到的是read-view的值.

<img src="https://s1.ax1x.com/2020/04/21/J8qkKU.png" alt="J8qkKU.png" style="zoom: 50%;" />

### 锁机制

行锁 表锁 页锁

- 共享锁(读锁) : 若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。

  ```sql
  select … lock in share mode;
  ```

- 排它锁(写锁) : 若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。这保证了其他事务在T释放A上的锁之前不能再读取和修改A。

  ```sql
  select ... for update
  ```

> InnoDB中行锁是作用在索引列,没有索引的列依旧是表锁

- 间隙锁 : RR级别下加上间隙锁防止幻读.

### 主从同步

```
Master -> change event -> binlog
slavec -> send dump command -> master
master -> push binlog -> slave
slave -> copy to relay log -> resolve byte stream
```

## 性能优化

```SPARQL
slow_query_log  -->  explain  -->  show profile  --> optimise
```

### B+ tree

![](http://ww2.sinaimg.cn/large/006tNc79gy1g62yd1knyzj30io08vtby.jpg)

```txt
Why not BST?  数据集特殊时,有可能退化为链表,树高不稳定.
Why not AVL?  为了维持绝对平衡需要大量旋转
Why not RBT？  RBT出度为2，树太高
Why not B Tree？ B Tree不利于区间搜索,需要回溯
Why not hash？  只能= 不能 < > in, 不能组合索引

B+ Tree : 本质多路平衡查找树+叶节点链表
出度更大，树高更低。
叶子节点有指针，无需回溯即可区间查找
可以一次加载进去树的一部分到内存

叶节点存值, 非叶节点存索引
结构比较扁平，高度低（一般不超过4层），随机寻道次数少
叶子节点形成有序链表，范围查询转化为顺序读，效率高。
相对而言B树必须通过中序遍历才能支持范围查询,需要回溯
```

### 索引

- 数据结构

  Full-Text : 全文索引; Hash : 范围排序时候不友好; B+ Tree : 范围+组合索引;BitMap:适合列有限取值(gender)

- 常见索引

  普通索引;  唯一索引:特殊的普通索引不允许重复,可以为空; 

  主键索引:特殊的唯一索引,不允许为空; 组合索引:最左前缀原则
  
- 聚簇索引 : 如主键索引.叶子节点存储行记录.

- 非聚簇索引 : 叶子节点存储主键的值, 需要再次根据主键的值再查找主键索引找到对应行记录

### Explain

- `id`

  标识执行顺序,表的读取顺序; 一个 select 一个id; id相同执行顺序由上向下; id不同id大的先执行; 
  \<derived2>表示该table有id=2的执行结果衍生出来

  优化策略 : 查看执行顺序, 让小表驱动大表, 先读小表再读大表

- select_type

  - SIMPLE : 查询中不含UNION或子查询的简单查询(JOIN查询也算SIMPLE)
  - PRIMARY : 包含UNION或者子查询的最外层查询的select_type为PRIMARY
  - SUBQUERY : 子查询,会创建临时表
  - DERIVED : 该查询会衍生出临时表

- `type`

  ```SPARQL
  system -> const -> eq_ref -> ref -> range -> index -> ALL
  ```

  - system : const 特例表示表中只有一条记录

  - const : 查询唯一索引,主键索引列时会出现. 只匹配到唯一的结果. 普通索引匹配出来的可能多个结果此时就不是const了.

    ```sql
    select * from `user` where id=1
    ```

  - eq_ref : 两表连接时当前表只有一条记录与前表匹配. `PRIMARY KEY` or `UNIQUE NOT NULL` index

    ```sql
    select `user`.* from `user`,`consultant` where `user`.`id`=`consultant`.`user_id`
    ```

  - ref : 多条记录匹配, 查找和扫描的混合体

    ```sql
    select `user_role` from `user_role` where `user_role`.`user_id`=3
    ```

  - range : 索引范围查找

    ```sql
    select * from `user` where id>3
    ```

  - index : 遍历索引树

    ```sql
    select id from `user`    -- id 为主键索引
    ```

  - ALL : 遍历文件

    ```sql
    select name from `user`  -- name 未建索引
    ```

- `key` : 实际用到的索引列

- `ref` : type为ref时实际的ref使用情况  const, col

- `rows` : 扫描匹配的行数

- extra 

  - `Using filesort` : 无法利用索引排序的使用文件排序, 未按照索引顺序进行`order by`
  - `Using temporary` : 借助临时表完成去重排序等功能时, 会产生临时表(distinct order by group by)
  - `Using index` 

### 索引失效

- `最左前缀原则`: 查询不跳过索引
- 索引列上使用 `计算` `函数` `自动或手动类型转换` 会使索引失效
- 联合索引中 某个索引列使用范围查找,则其后的索引列索引失效
- 使用 `!=` 索引失效
- `IN ` `NOT IN` 使索引失效
- `IS NULL` `IS NOT NULL` 索引失效
- `LIKE` 可能导致索引失效
- `Order By` 多个索引列升降不同
- `Group By` 先排序后分组,失效情况和 `Order By` 相同

![image-20190818162437256](http://ww4.sinaimg.cn/large/006tNc79gy1g63wrdaxmej31lg0mq49f.jpg)

## 扩展

[ProcessOn MySQL高级](https://www.processon.com/view/5a3efcb3e4b0bf89b85681d0#map)

[ProcessOn MySQL高级](https://www.processon.com/view/5d41bbb1e4b0d11c89182772#map)


