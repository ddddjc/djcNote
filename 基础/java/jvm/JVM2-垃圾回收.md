# 垃圾回收

## 1. 如何判断对象了可以回收

### 1.1引用计数法

- 给每个对象一个计数器，如果被引用一次就+1，少被引用则-1.
- 可能会有对象产生环形相互依赖，这样外面没有被引用，但无法被回收

### 1.2可达性分析算法

- 从GC Roots开始向下搜索，搜索所中国的路径被称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此是可以回收的
- 当对象对当前使用的应用程序变的不可触及的时候，这个对象就可以被回收了



`垃圾回收不会发生在永久代，如果永久代满了或者是超过了临界值，会触发完全垃圾回收（Full GC）。查看垃圾回收器的输出信息，就会发现永久代也是被回收的。这就是为什么正确的永久代大小对避免Full GC是非常重要的原因`

### 1.3 四种引用

`有五种`

1. 强引用
   - 只有所有GC Root对象都不通过【强引用】引用该对象，该对象才能被垃圾回收
2. 软引用（SoftReference）
   - 仅有软引用引用该对象时，在垃圾回收后，内存仍不足时会再次触发垃圾回收，回收软引用对象
   - 可以配合引用队列来释放引用自身
3. 弱引用（WeakReference）
   - 仅有弱引用引用该对象时，在垃圾回收时，无论内存是否充足，都会回收弱引用对象
   - 可以配合引用队列来释放弱引用自身
4. 虚引用（PhantomReference）
   - 必须配合引用队列使用，主要配合ByteBuffer使用，被引用对象回收时，会将虚引用入队，由Reference Handler 线程调用虚引用相管方法释放直接内存
5. 终接器引用（FinalReference）
   - 无需手动编码，但内部配合引用队列使用，在垃圾回收时，终接器应用入队（被引用对象暂时没有被回收），再由Finalizer线程通过终接器引用找到被引用对象并调整他的finalize方法，第二层GC时才能回收被引用对象。

## 2. 垃圾回收

### 2.1 标记-清除算法

标记无用对象，然后进行清除回收。缺点：效率不高，无法清除垃圾碎片。

- 该算法分为两个阶段，标记和清除。标记阶段标记所有需要回收的对象，清除阶段回收被标记的对象所咱用的空间。该算法最大的问题就是内存碎片严重，后续可能发生对象不能找到利用空间的问题。![标记清除算法](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307192018099.png)

### 2.2 标记-整理算法

标记无用对象，让所有存活对象都向一端移动，然后直接清掉段边界以外的内存

- 标记后不是清理对象，而是将存活对象移向内存的一端，然后清除端边界外的对象

![标记-整理算法示意图](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307192018250.png)

### 2.3 复制算法

按照容量划分两个大小一样的内存区域，当一块用完的时候将活着的对象复制到另一块上，然后再把已使用的内存空间一次清理掉。缺点：内存使用率不高，只有原来的一半。

- 按内存容量将内存划分为等大小的两块。没次只使用一块，当这块内存满后将尚存活的对象复制到另一块上去，把已使用的内存清掉。

![赋值算法示意图](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307192018940.png)

## 3. 分代垃圾回收

普通GC和全局GC

- 普通GC(minor GC)：只针对新生代区域的GC，指发生在新生代的垃圾回收动作，因为大多数Java对象存活率都不高，所以Minor GC非常频繁，一般回收速度也比较快。

- 全局GC(major GC or FullGC) ：指发生在老年代的垃圾收集动作，出现了Major GC，经常会伴随至少一次的MinorGC(不是绝对的)。Major GC的速度一般要比Minor GC慢10倍以上。

***

新生代三个区域占比8:1:1

![image-20230719144959903](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307191449970.png)

- 创建新的对象时，首先分配到伊甸园区域
- 新生代的伊甸园空间不足时，触发`minor gc`，伊甸园和form存活的对象使用copy复制到to区域中，存活的对象年龄+1并且清空伊甸园和from区域，后**交换from和to**，令from指向幸存区保留对象的内容，而to区作为中间停留区。
- `minor gc` 会引发一次stop the word（stw）。暂停其他用户线程，等垃圾回收结束后，才继续运行(若不停止，可能发生线程不安全问题)
- 当对象寿命超过阈值时，会晋升到老年代，最大寿命是15（保存在对象头中，占4bit）
- 当新生代要晋升时，发现老年代空间不足，会触发`full gc (清理整个堆空间-包括新生代好老年代)`，stw的时间更长（存储对象较多、不那么容易进行垃圾回收，以及算法与新生代不同（标记加整理、标记加清除））

![img](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307191532960.webp)

### 3.1 相关VM参数

![image-20230719153426737](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307191534811.png)

- 大对象在新生代空间不够但老年代空间足够的情况下，直接回晋升到老年代

## 4.垃圾回收器

### 4.1 串行

- 单线程
- 堆内存较小，适合个人电脑

`-XX:+UseSerialGC =Serial + SerialOld`

- Serial：工作在新生代，复制算法
- SerialOld：工作在老年代，标记+整理算法

![image-20230719161012089](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307191610179.png)

工作流程：每个线程都在运行中，若发生垃圾回收，则先让每个线程到达一个安全点后阻塞，后执行垃圾回收线程，垃圾回收执行结束后，开始其他线程。

### 4.2 吞吐量优先 

吞吐量：垃圾回收时间与程序运行时间占比，占比越低，吞吐量越高。

- 多线程
- 堆内存较大，多核cpu
- 单位时间内，stw的时间最短 0.2+0.2=0.4

`-XX:+UseParallelGC ~ -XX:+UseParallelOldGC ` :一个即可

 `-XX:+UseAdaptiveSizePolicy ` ：

 `-XX:GCTimeRatio=ratio ` ：

 `-XX:MaxGCPauseMillis=ms `

 `-XX:ParallelGCThreads=n `

![image-20230719161936031](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307191619100.png)

### 4.3 响应时间优先（CMS）

- 多线程
- 堆内存较大，多核cpu

- 尽可能让stw(stop the word) 的单次时间最短 0.1+0.1+0.1+0.1+0.1=0.5

`-XX:+UseConcMarkSweepGC ~ -XX:+UseParNewGC ~ SerialOld`

`-XX:ParallelGCThreads=n ~ -XX:ConcGCThreads==threads `

`-XX:CMSInitiatingOccupancyFraction=percent`

`-XX:+CMSScavengeBeforeRemark `

![image-20230719162858890](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307191628962.png)

1、初始标记（CMS initial mark） 初始标记仅只是标记一下GC Roots能直接关联到的对象，速度很快，需要“Stop The World”。
2、并发标记（CMS concurrent mark） 并发标记阶段就是进行GC Roots Tracing的过程。
3、重新标记重新标记阶段是为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短，仍然需要“Stop The World”。
4、并发清除（CMS concurrent sweep） 并发清除阶段会清除对象。

优点：并发收集、低停顿、“标记-清除”算法

缺点：

1. CMS会抢占CPU资源。**并发标记阶段虽然不会导致用户线程暂停**，但确需要CPU分出精力去执行多条垃圾收集线程，从而**使得用户线程的执行速度下降**
2. CMS无法处理浮动垃圾（Floating Garbage），可能会出现“Concurrent Mode Failure”而导致另
   一次Full GC：
   并发清理的过程中，由于用户线程还在执行，因此就会继续产生对象和垃圾，这些新的垃圾没有被标记，CMS只能在下一次收集中处理它们。这也导致了CMS不能在老年代几乎完全被填满了再去进行收集，必须预留一部分空间提供给并发收集时程序运作使用。在JDK1.5默认设置下，老年代使用了68%(JDK1.6是92%)的空间后CMS的垃圾收集就会被激活，其实这是一个比较保守的设置，只要应用中老年代增长不是很快，可以适当地调高参数-XX:CMSInitialingOccupancyFraction来提高触发百分比，降低回收的频率来获得更好的性能。如果CMS在收集期间，内存无法满足程序的需要，就会出现“Concurrent Mode Failure”，这时JVM将临时调用单线程的Serial Old收集器来重新进行老年代的垃圾收集，这样的话，CMS原本降低停顿时间的目的不仅没完成，和直接使用Serial Old收集器相比，还增加了前面几个阶段的停顿时间。
3. CMS的“标记-清除”算法，会导致大量空间碎片的产生：
   碎片的积累会给分配大对象带来麻烦，往往会出现明明老年代还有很多空间剩余，但是却无法找到连续的空间分配对象的情况，这时候就不得不触发一次Full GC。
   为了解决这个问题。CMS提供了一个-XX:+UseCMSCompactAtFullCollection的开关参数（默认是开启的），用于在CMS收集器进行Full GC时对内存碎片进行合并整理，整理的过程是需要暂停用户线程的，这样碎片虽然没有了，但停顿时间又变长了。
   CMS的设计初衷可是降低停顿，于是又提供了一个参数-XX:CMSFullGCsBeforeCompaction，用于设置执行多少次不压缩碎片的Full GC后，跟着来一次带压缩的Full GC（默认值为0，即每次都会）。

### 4.4 G1

定义：Garbage First

使用场景：

- 同时注重吞吐量和低延迟，默认的暂停目标是200ms
- 超大堆内存，会将堆划分为多个大小相等的Region
- 整体上是标记+整理内存，两个区域之间是复制算法

相关JVM参数

`-XX:+UseG1GC `

`-XX:G1HeapRegionSize==size`

` -XX:MaxGCPauseMillis=time`

#### 1. 垃圾回收阶段



## 5.垃圾回收调优

### 5.1 调优领域

- 内存
- 锁竞争
- cpu占用
- io

### 5.2 确定目标

- 【低延迟】还是高吞吐量，选择合适的回收器
- CMS，G1，ZGC
- ParallelGC
- Zing

### 5.3 最快的GC是不发生GC

- 查看FullGC前后的内存占用，考虑：
  - 数据是不是太多？
  - resultSet = statment.executeQuery（“select * from 大表 limit n”）
- 数据表示是否太臃肿
  - 对象图
  - 对象大小 16 Integer 24 int 4  能用基本数据类型尽量用基本数据类型
- 是否存在内存泄漏
  - static Map map=   容易造成泄漏
  - 软引用
  - 弱引用
  - 第三方缓存实现

###  5.4 新生代调优

- 新生代特点
  - 所有的new 操作的内存分配非常廉价
    - TLAB thread-local allocation buffer
  - 死亡对象的回收代价是0
  - 大部分对象用过即死
  - Minor GC 的时间远远低于Full GC

- 新生代过大的影响
  - 可以减少stw的频率
  - 设置较大的新生代会减少老年代的大小，垃圾回收频率减少，但如果要进行垃圾回收，就很可能会会触发Full GC,这样会占用更长的时间进行垃圾回收
  - 吞吐量和新生代大小的关系是一个先上升后下降的曲线
- 新生代建议大小为堆大小的25%到50%
- 幸存区大到能够保留【当前活跃对象+需要晋升对象】
  - 如果幸存区较小，jvm可能会把不需要晋升的对象晋升到老年代
  - 晋升阈值配置得当，让长时间存活对象尽快晋升
    - `-XX:MaxTenuringThreshold=threshold `
    - `-XX:+PrintTenuringDistribution`

### 5.5 老年代调优

以CMS为例

- CMS的老年代内存越大越好
- 先尝试不做调优，如果没有Full GC 那么已经可以了，否者先尝试调优新生代
- 观察发现Full FC时老年代内存占用，将老年代内存与设调大1/4~1/3
  - `-XX:CMSInitiatingOccupancyFraction=percent`
