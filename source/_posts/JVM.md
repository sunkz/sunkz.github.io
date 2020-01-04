title: JVM
author: Sunkz
tags:

  - jvm

photos:

- https://s2.ax1x.com/2019/12/31/l1Kp6J.png

categories: []
date: 2019-12-31 11:42:00

---
## 整体架构

<div align=center><img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4ymxwegnyj31ay0u0td5.jpg" alt="jvm.png" style="zoom:40%;" /></div>

![jmm.png](http://tva1.sinaimg.cn/large/0060lm7Tly1g4ymybolb1j311v0u0wqg.jpg)

- 程序计数器 : 记录程序执行的行号,此区域Java虚拟机规范中没有规定任何OutOfMemoryError

- 虚拟机栈 : 描述Java方法执行的动态内存模型.

  - 栈帧 : 每个方法执行都会创建一个栈帧.存储局部变量表,操作数栈,动态链接,方法出口等.栈帧伴随着方法的执行和结束而创建和销毁.

    ```
    StackOverFlowError : 方法的栈帧进栈(虚拟机栈),但不出栈(通常递归调用时),虚拟机栈将被栈帧填满,栈内存溢出.
    ```

    - 局部变量表 : 存放编译期可知的各种基本数据类型,引用类型.局部变量表的内存空间在编译期完成分配,当进入一个方法,这个方法在帧中分配的内存大小是固定的,方法运行期间不改变局部变量表的大小.局部变量表只存对象的引用,对象实际存储在堆中.
    - 操作数栈 : 指令
    - 动态链接 : 指向多态
    - 返回地址 : returnAddress

- 本地方法栈 : 执行native方法的栈

- 堆 : 存放对象实例

- 方法区(永久代) : 存放类信息、常量、静态变量,及时编译期编译后的代码等数据

  - 类信息 : 类的版本,字段,方法,接口
  - 常量池 : StringTable - HashSet 结构
  
- 直接内存 : 可将对象分配到对外内存,不受Java堆内存大小限制.

## JVM内存区域

### 对象的创建

- new 类名

- 根据new的参数在常量池中定位一个类的符号引用

- 如果没有找到这个符号引用,说明类还没有被加载,则进行类的加载解析和初始化.

- 虚拟机为对象分配内存(位于堆中)

  - 内存分配策略

    - 指针碰撞 : 将设Java堆中的内存分配是绝对规整的,指针作为已分配内存和未分配内存的分界线,给变量分配内存的过程就是将作为分界线的指针向未分配的内存空间挪动一个变量的大小.	
    - 空闲列表 : 如果Java堆中的内存分配不是规整的,虚拟机就会维护一个空闲列表,用来记录剩余的可用的内存空间.每次为变量分配内存后动态的去维护这个列表.

  - 内存分配过程中需处理线程安全问题

    - 多线程环境下操作指针(碰撞)或者空闲列表时加锁.

    - 在堆中为每个线程开辟本地线程分配缓冲 Thread Local Allocation Buffer

      ```
      本地线程分配缓冲Thread Local Allocation Buffer(TLAB) : 每个线程划分自己的内存空间,分配对象只在自己的内存空间.TLAB仅作用于新生代的Eden Space,但如果对象过大的话则仍然是直接使用堆空间分配.虽然总体来说堆是线程共享的，但是在堆的年轻代中的Eden区可以分配给专属于线程的局部缓存区TLAB，也可以用来存放对象。相当于线程私有的对象。
      ```

- 将分配的内存初始化为零值(不包括对象头)

- 调用对象的\<init\>方法(构造方法,静态代码块)

### 对象的结构

- Header

  - 自身运行时的数据(Mark Word) : 长度32位或者64位 

    Hash值,GC分代年龄,锁状态标志,线程持有的锁,偏向线程ID,偏向时间戳

    <div align=center><img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4ymys7tlvj30w00gq412.jpg" alt="Mark Word" style="zoom:50%;" /></div>

  - 类型指针 : JVM通过该指针确定这个对象是哪个类的实例

- InstanceData

  - 真正存在存储对象的数据

- Padding

  - 填充位,保持整个对象的大小是8的整数倍

### 对象的访问定位

```
reference类型的两种访问方式
```

- 使用句柄
  - 栈空间的指针指向堆中的一块区域(句柄池),句柄池中保存了堆中对象的实际地址

    堆中的对象移动位置,被GC等,栈中的指针始终不变指向句柄池,变的指示句柄池中对象的实际地址

- 直接指针
  
  - 栈空间的指针直接指向堆中的对象.好处,直接定位,性能非常高(HotSpot采用此方式)

<div align=center><img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4ymz44v82j31140e4ju3.jpg" style="zoom: 40%;" /></div>

## 垃圾回收

### 垃圾判定

- 引用计数法 :在对象中添加一个引用计数器,当对象被引用时就+1,当引用失效时就-1. 缺点,当对象存在循环引用时候,就不能判定是垃圾了.

  ```
  打印垃圾回收信息 -verbose:gc --xx:+PrintGCDetail
  ```

  - 强引用 : new 内存区域不足也不会回收强引用的对象,会抛出OutOfMemoryError. 通过将对象指定为null,使其引用弱化,被回收.
  - 软引用 : 只有当内存空间不足是才会回收该引用对象的内存.
  - 弱引用 : 发生GC时,垃圾收集器发现有弱引用的对象就会回收.
  - 虚引用 : 虚引用关联的对象被GC时,会收到系统通知.

- 可达性分析

  - 可以作为GC Roots 的对象 : 栈帧中的局部变量表,方法区类属性所引用的对象,方法区中常量所引用的对象.本地方法栈中引用的对象.以上这些可以作为GC Roots 的对象向下(堆)查找,找不到的对象,都可作为垃圾.

<div align=center><img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4ymzhfpewj31140gw76c.jpg" style="zoom:33%;" /></div>

### 如何回收

#### 回收算法

- 标记清除 : 先使用可达性分析法查找并标记出垃圾,在清除

  缺点 : 堆中会出现很多不连续的空间小区域, 新的大对象将没办法分配空间.如果JVM找不到可用分配区,将再次触发垃圾回收

<div align=center><img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4ymzu3jijj312e0ikdh3.jpg" style="zoom:33%;" /></div>

- 复制算法
  - 堆 : 新生代 ,老年代   缺点 :  新生代区域只能利用50%空间
  - 优化 : 新生代细分((Eden,Survivor,Survivor: 8-1-1) : 因为大部分都是垃圾只有10%能存活
    - 新对象存放在 Eden, GC 之后将 存活的对象移到 Survivor1. 再次新对象扔到Eden,GC之后,Eden和Survivor1中存活的对象都移到Survivor2.这样将有80%空间得到利用.
  - 分配担保 : 如果另一块Survivor空间没有足够空间存放上一次新生代收集下来的存货对象时,这些对象将直接通过分配担保机制进入老年代(方法区).

<div align=center><img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4yn06rhk3j312e0ikt9x.jpg" style="zoom:33%;" /></div>

- 标记整理 : 针对老年代, **老年代不适合复制算法,因为每次都有大量存活,复制效率低**
  - 标记清除之后再整理

<div align=center><img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4yn0iywxlj31ag0ly0u3.jpg" style="zoom:33%;" /></div>

- 分代收集

  针对不同区域采用不同的收集算法 : 新生代-复制算法,老年代-标记整理
  
  Young(1/3) : Eden(* 8/10)  2 * Survivor(1/10) [  from(* 1/10) to (* 1/10) ]  ;   Old(2/3) 
  
  Young -> Old : 经理过一定Minor次数之后依然存活的对象;Survivor区存放不下的对象

#### 回收分类

- Minor GC(Young)

  ```
  触发条件 : Eden空间不足
  ```

- Full GC(Old)

  ```
  触发条件:
  	创建一个大对象如果Eden空间内存不足,则直接保存Old,如果Old也不足则触发Full GC
  	Minor GC时候Survivor放不下,则放在Old,如果Old也放不下则触发Full GC
  	Minor GC时候Survivor可以放下,但是存活到Survivor的平均大小大于Old区剩余空间,则触发FullGC
  	程序中调用 System.gc()可能触发FullGC	
  ```

#### 回收过程

- Stop-the-World

  ```
  为防止GC Roots枚举时对象引用不断变化,GC进行时必须停顿所有用户线程.
  大多数GC优化都是通过减少Stop-the-World的时间
  ```
  
- SafePoint

  ```
  程序并非在所有地方都能暂停开始GC,需要找到一个合适的时间进行GC.
  SafePoint一般在:方法调用,循环跳转,异常跳转的时候.
  一旦GC要发生让所有线程都跑到安全点再停顿下来,再开始GC.
  ```

- SafeRegion

  ```
  线程Block,Sleep时并不能走到Safepoint,所以这类线程不能及时跑到安全点.
  SafeRegion是指一段代码中引用关系不会发生变化,在这个Region的任何地方开始GC都是安全的.
  线程执行SafeRegion代码时,首先会标识自己进入SafeRegion,这样GC时就不用管该线程的状态了.
  SafeRegion可以看做是SafePoint的扩展.
  ```

- Throughput

  ```
  吞吐量=运行用户代码时间/(运行用户代码时间+垃圾收集时间)
  ```

#### 垃圾收集器

- Serial  : 复制算法,单线程,进行垃圾收集时,必须暂停所有工作线程.(Young+Old)

<div align=center><img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g6s59ahb9kj315i09y0ub.jpg" alt="image-20190908153217262" style="zoom: 40%;" /></div>

- ParNew  :  复制算法,多线程收集(Young+Old)

<div align=center><img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g6s5apy0x6j315e0am408.jpg" alt="image-20190908153342556" style="zoom:40%;" /></div>

- CMS Concurrent Mark Swap  并发标记清除  (老年代)

  - 初始标记 : 需要Stop-The-World, 只标记GC Roots能直接关联到的对象.

  - 并发标记 : 执行 GC Roots Tracing 过程, 并不停止用户线程.

  - 重新标记 : 需要Stop-The-World,修正并发标记期间用户下城继续运作而导致标记产生变动的那一部分对象的标记记录,这个阶段的停顿时间会比初始标记阶段稍长一些,但远比并发标记的时间短.

  - 并发清除 : 不会停止用户线程.

    ```
    最耗时的并发标记和并发清除过程,收集器线程和用户线程可以一起工作,所以性能极高
    ```

<div align=center><img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4yn0zqlc7j313809kjsw.jpg" style="zoom:40%;" /></div>

- G1
  - 利用多CPU多核环境减少Stop-The-World时间,基于标记整理,可预测停顿时间
  - 初始标记 : 需要Stop-The-World, 只标记GC Roots能直接关联到的对象.
  - 并发标记 : 执行 GC Roots Tracing 可达性分析过程, 并不停止用户线程.
  - 最终标记 : 修正并发标记.
  - 筛选回收 : 对各个Region的回收价值和成本进行排序,根据用户所期望的GC停顿时间制定回收计划.

<div align=center><img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4yn1d0sptj312s07uabq.jpg" style="zoom:40%;" /></div>

```
Hotspot 虚拟机包含所有收集器
```

<div align=center><img src="https://s2.ax1x.com/2019/12/26/lAm1iT.png" alt="lAm1iT.png" style="zoom:40%;" /></div>

- ZGC : JDK11

## 内存分配

```
优先分配到Eden
大对象可能直接分配到老年代 : 防止大对象在Eden来回复制移动
长期存活的对象分配到老年代
空间分配担保 : 新生代内存不够向老年代借用
逃逸分析以及栈上分配
```

## 虚拟机工具

- jps : 显示虚拟机进程
- jstat : 收集虚拟机运行数据
- jinfo : 显示虚拟机配置信息
- jstack : 显示虚拟机线程快照
- jconsole : GUI

## Class文件结构

```
以字节为基础单位的二进制流,对应着唯一的一个类或接口的定义信息.
```

![](https://s2.ax1x.com/2019/12/28/le4Gv9.png)

```
魔数.主次版本号.常量池的数量.访问标志.类索引.父类索引.变量池...
```

## 类加载子系统

### 类加载过程

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4zeva8db5j31xy0qa7cn.jpg)

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4zexi46f5j320k0jstiz.jpg)

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4zkk3sbmaj30r4150n8p.jpg)

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4zkkojw65j30w20u0156.jpg)

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4zkl3ksj1j30tm098dgt.jpg)

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4zkm2fnebj30xu0u0gth.jpg)

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4zkncevauj30u016hwo7.jpg)

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4zkqmav96j30u00xr11t.jpg)

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4zkqxtswdj30u00zpgt9.jpg)

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4zkrcesugj31780ro12c.jpg)

### 四种类加载器

![](http://tva1.sinaimg.cn/large/0060lm7Tly1g4zl1ygrrkj31c80oogs3.jpg)

### 双亲委派模型

<div align=center><img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4zoa7vfggj312r0u0jw0.jpg" style="zoom:40%;" /></div>

<img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4zo9g1w9dj31tk0igjvx.jpg"  />

```
自底向上询问是否加载,自顶向下尝试加载 : 防止核心类被自定义类加载.
```

```
隐式加载 : new , ClassLoader自动去找 new 所指的.class文件 
显示加载 : loadClass, forName 
```

```
双亲委托机制是为了保证一个 Java 类在 JVM 中是唯一的，假如你不小心写了一个与 JRE 核心类同名的类，比如 Object 类，双亲委托机制能保证加载的是 JRE 里的那个 Object 类，而不是你写的 Object 类。这是因为 AppClassLoader 在加载你的 Object 类时，会委托给 ExtClassLoader 去加载，而 ExtClassLoader 又会委托给 BootstrapClassLoader，BootstrapClassLoader 发现自己已经加载过了 Object 类，会直接返回，不会去加载你写的 Object 类。
```

```
自定义类加载器需要继承 ClassLoader 抽象类，再重写 findClass 和 loadClass 方法即可，如果你要打破双亲委托机制，就需要重写 loadClass 方法，因为 loadClass 的默认实现就是双亲委托机制。
```

## 运行时栈帧结构

<div align=center><img src="http://tva1.sinaimg.cn/large/0060lm7Tly1g4zos1i0xrj30ra0q2dli.jpg" style="zoom:50%;" /></div>

## 扩展

### 常用参数

```
-Xss : 每个线程虚拟机栈的大小
-Xms : 堆初始大小
-Xmx : 堆扩容的最大值
-XX : SurvivorRadio : Eden 和 Survivor 比例 默认 8:1
-XX : NewRadio : Young 和 Old 比例 
-XX : MaxTenuringThreshold : 从Young->Old经过GC最大阈值
-XX : +UseSerialGC 使用Serial垃圾收集器
```

### Happens-Before

```
happens-before 指两个操作之间的执行顺序.提供跨线程的内存可见性.
如果一个操作的执行结果需要对另一操作可见,那么这两个操作之间必然存在happens-before关系
(前一操作对后一操作可见不代表前一操作先于后一操作)
```

- 程序顺序规则

  ```
  单线程的每个操作,	总是前一个操作happens-before与该线程中的任意后续操作
  ```

- 监视器规则

  ```
  对一个锁的解锁,总是happens-before对这个锁的加锁
  ```

- volatile变量规则

  ```
  对一个volatile域的写,总是happens-before与任意后续对这个volatile域的读
  ```

### 重排序

```
编译器和处理器为了提高程序执行性能,对指令重新排序
```

- 数据依赖性(as-if-serial)

  ```
  写后读 读后写 写后写
  ```

- 重排序分类

  ```
  编译器重排序 : 编译器在编译过程中重排序
  处理器重排序 : 没有数据依赖性的两个操作,处理器也可能对这两个操作重排序
  ```

- 内存屏障

  ```
  LoadLoad  StoreStore  LoadStore  StoreLoad
  ```
