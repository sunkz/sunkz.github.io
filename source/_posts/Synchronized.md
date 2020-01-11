title: Synchronized
author: Sunkz
tags:

  - synchronized
  - markword
  - lock
  - monitor

photos:

- https://s2.ax1x.com/2020/01/03/laMmn0.png

categories: []
date: 2020-01-03 09:59:00

---

### Operating System

```
信号量semaphore : 进程自备同步操作，P(S)和V(S)操作大量分散在各个进程中，不易管理，易发生死锁。
```

```
管程monitor : 封装了同步操作，对进程隐蔽了同步细节，简化了同步功能的调用。
```

### ObjectMonitor

```
JVM对管程的C++实现
```

```c++
ObjectMonitor(){
    _owner=NULL;//临界资源
    _count=0;//同一线程可多次进入 ++
    _cxq=NULL;//所有请求锁的线程首先会被放在这个链表中
    _EntryList=NULL;//_cxq或WaitSet中有资格成为候选资源的线程会被移动到该List
    _WaitSet=NULL;//执行wait()方法后进入此Set
    
    void enter(TRAPS);
    void exit(TRAPS);
    void wait(jlong millis,bool interruptable,TRAPS);
    void notify(TRAPS);//从WaitSet头取出一个线程根据不同的策略放入EntryList或者_cxq
    void notifyAll(TRAPS);
}
```

<div align=center><img src="https://cdn.shenlanbao.com/consultants/167428545_0200103143438.png" style="zoom: 40%;" /></div>


### Synchronized

```java
synchronized (this){ //CurrThread进入_EntryList
    ...   //_owner=CurrtThread , count++
    this.notify();//不释放锁 _owner=CurrThread.WaitSet队头Thread进入_cxq或者EntryList.
    this.wait();//将_owner指向NULL,CurrTread进入WaitSet,EntryList中的线程开始TryLock.
    ...  //将_owner指向NULL,EntryList中的线程开始TryLock.
}
```

```
线程执行到带synchronized会自动的在该方法前后添加monitorenter和monitorexit指令(monitor指令的隐式调用).ObjectMonitor中的成员方法对临界资源操作时,涉及到用户态和内核态的切换,造成Synchronized同步效率低下,在JDK1.6只后,在JVM层面对其做了优化.
```

### Synchronized Optimization

#### Mark Word

<img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4ymys7tlvj30w00gq412.jpg" style="zoom:50%;" />

#### 锁升级

```
无锁 偏向锁 轻量级锁 重量级锁
```

![](https://cdn.shenlanbao.com/consultants/479139810_p2368988626.jpg)

```
开启偏向锁  -XX:+UseBiasedLocking   jdk6时增加偏向锁和轻量级锁,jdk5之前Synchronized一直为重量级锁
无竞争时,消除同步操作. 再次存在竞争,则偏向锁取消. 并发较小时,开启偏向锁将大大降低,锁的竞争时间.

线程再次请求锁时,只需检查对象投中的线程ID标记位,就知道当前线程是否拥有该对象的锁,而无需进入到Monitor中检查_owner=线程ID.

JDK1.7之后自旋锁默认开启.自旋次数由JVM控制.(高并发情况下,自旋非常消耗CPU资源)
```

- 重入锁 : Synchronized(内核态实现 count字段) ReentryLock(用户态实现) 都是重入锁

- 自旋锁 : 线程自旋性能消耗小于线程挂起唤醒性能消耗时使用. (轻量级锁)

