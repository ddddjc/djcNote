# CAS与原子操作

## 1.乐观锁与悲观锁的概念

锁可以从不同的角度分类。其中，乐观锁和悲观锁是一种分类方式

**悲观锁：**

悲观锁就是我们常说的锁。对于悲观锁来说，它总是认为每次访问共享资源时会发生冲突，所以必须对每次数据操作加上锁，以保证临界区的程序同一时间只能有·一个线程在执行。

**乐观锁：**

乐观锁有称为“无锁”，乐观锁总是假设对共享资源的访问没有冲突，线程可以不听地执行，无需加锁也无需等待。而一旦多个线程发生冲突，乐观锁通常是使用一种称为CAS的技术来保证线程执行的安全性。

由于无锁操作中没有锁的存在，因此不可能会出现死锁的情况，也就是说**乐观锁天生免疫死锁**

乐观锁多用于**“读多写少”**的环境，避免频繁加锁影响性能；而悲观锁多用于**“写多读少”**的环境，避免频繁失败和重试影响性能。

## 2.CAS概念

CAS的全称是：比较并交换（Compare And Swap）。在CAS中， 有这样三个值：

- V：要更新的变量（var）
- E：预期值（expected）
- N：新值（new）

比较并交换过程如下：

判断V是否等于E，如果等于，将V的值设为N；如果不等于，说明已经有其他线程更新了V，则当前线程放弃更新，什么也不做。

所以这里的**预期值E本质上指的是“旧值”**

我们以一个简单的例子来解释这个过程：

1. 如果有一个多线程共享的变量i原本大于5，现在在线程A中想把他设为6；
2. 使用CAS来做这件事
3. 实现会用i与5对比，发现他大于5，说明没有被其他线程改过，就把他设为新的值6，注册CAS成功，i的值被设置为6
4. 如果不等于5，说明i被其他线程改过了，那就什么也不做，CAS失败，i仍未2

在这个例子中，i就是V，5就是E，6就是N。

> 不会出现判断了i为5后，正准备更新他的新值是，它被其他线程更爱了
>
> 因为CAS是一种原子操作，它是一种系统原语，上一条CPU的原指令，从CPU层面保证了他的原子性

**当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余操作均会失败，但失败的线程并不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作**

## 3. Java实现CAS的原理-Unsafe类

前面提到的，CAS是一种原子操作，那么Java是这样来使用CAS的》

我们知道，在Java中，如果一个方法是native的，那Java就不负责具体实现它，而是交给底层的JVM使用c或者c++去实现。

在Java中，有一个Unsafe类，他在sun.misc包中。他里面是一些native方法，其中就有几个关于CAS的：

```java
boolean compareAndSwapObject(Object o, long offset,Object expected, Object x);
boolean compareAndSwapInt(Object o, long offset,int expected,int x);
boolean compareAndSwapLong(Object o, long offset,long expected,long x);
```

都是public native的。

Unsafe中对CAS的实现主要是c++写的，它的具体实现和操作系统、CPU都有关系。

Linux的X86下主要是通过cmpxchgl这个指令在CPU级完成CAS操作的，但在多处理器情况下必须使用lock指令加锁来完成。当然，不同的操作系统和处理器的实现会有所不同。

Unsafe类中还有其他方法用于不同的用途。比如支持线程挂起和回复的park和unpark，LockSupport底层就是调用了这两种方法，还有支持反射操作的allocateInstance（）方法。

## 4. 原子操作-AtomicInteger类源码分析

jdk提供了一些用于原子操作的类，在`java.util.concurrent.atomic`包下面。

![原子类](http://concurrent.redspider.group/article/02/imgs/%E5%8E%9F%E5%AD%90%E7%B1%BB.jpg)

从名字可以看出这些类的大致用途：

- 原子更新基本类型
- 原子引用数组
- 原子更新引用
- 原子更新字段（属性）

这里我们以`AtomicInteger`类的`getAndAdd(int delta)`方法为例，来看看Java是如何实现原子操作的。

先看看这个方法的源码：

```java
public final int getAndAdd(int delta) {
    return U.getAndAddInt(this, VALUE, delta);
}
```

这里的U其实就是一个`Unsafe`对象：

```java
private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
```

所以其实`AtomicInteger`类的`getAndAdd(int delta)`方法是调用`Unsafe`类的方法来实现的：

```java
@HotSpotIntrinsicCandidate
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, offset, v, v + delta));
    return v;
}
```

> 注：这个方法是在JDK 1.8才新增的。在JDK1.8之前，`AtomicInteger`源码实现有所不同，是基于for死循环的，有兴趣的读者可以自行了解一下。

我们来一步步解析这段源码。首先，对象`o`是`this`，也就是一个`AtomicInteger`对象。然后`offset`是一个常量`VALUE`。这个常量是在`AtomicInteger`类中声明的：

```java
private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");
```

同样是调用的`Unsafe`的方法。从方法名字上来看，是得到了一个对象字段偏移量。

> 用于获取某个字段相对Java对象的“起始地址”的偏移量。
>
> 一个java对象可以看成是一段内存，各个字段都得按照一定的顺序放在这段内存里，同时考虑到对齐要求，可能这些字段不是连续放置的，
>
> 用这个方法能准确地告诉你某个字段相对于对象的起始内存地址的字节偏移量，因为是相对偏移量，所以它其实跟某个具体对象又没什么太大关系，跟class的定义和虚拟机的内存模型的实现细节更相关。

继续看源码。前面我们讲到，CAS是“无锁”的基础，它允许更新失败。所以经常会与while循环搭配，在失败后不断去重试。

这里声明了一个v，也就是要返回的值。从`getAndAddInt`来看，它返回的应该是原来的值，而新的值的`v + delta`。

这里使用的是**do-while循环**。这种循环不多见，它的目的是**保证循环体内的语句至少会被执行一遍**。这样才能保证return 的值`v`是我们期望的值。

循环体的条件是一个CAS方法：

```java
public final boolean weakCompareAndSetInt(Object o, long offset,
                                          int expected,
                                          int x) {
    return compareAndSetInt(o, offset, expected, x);
}

public final native boolean compareAndSetInt(Object o, long offset,
                                             int expected,
                                             int x);
```

可以看到，最终其实是调用的我们之前说到了CAS `native`方法。那为什么要经过一层`weakCompareAndSetInt`呢？从JDK源码上看不出来什么。在JDK 8及之前的版本，这两个方法是一样的。

> 而在JDK 9开始，这两个方法上面增加了@HotSpotIntrinsicCandidate注解。这个注解允许HotSpot VM自己来写汇编或IR编译器来实现该方法以提供性能。也就是说虽然外面看到的在JDK9中weakCompareAndSet和compareAndSet底层依旧是调用了一样的代码，但是不排除HotSpot VM会手动来实现weakCompareAndSet真正含义的功能的可能性。

根据本文第一篇参考文章（文末链接），它跟`volatile`有关。

简单来说，`weakCompareAndSet`操作仅保留了`volatile`自身变量的特性，而除去了happens-before规则带来的内存语义。也就是说，`weakCompareAndSet`**无法保证处理操作目标的volatile变量外的其他变量的执行顺序( 编译器和处理器为了优化程序性能而对指令序列进行重新排序 )，同时也无法保证这些变量的可见性。**这在一定程度上可以提高性能。

再回到循环条件上来，可以看到它是在不断尝试去用CAS更新。如果更新失败，就继续重试。那为什么要把获取“旧值”v的操作放到循环体内呢？其实这也很好理解。前面我们说了，CAS如果旧值V不等于预期值E，它就会更新失败。说明旧的值发生了变化。那我们当然需要返回的是被其他线程改变之后的旧值了，因此放在了do循环体内。

## 5. CAS实现原子性操作的三大问题

### 5.1 ABA问题

ABA问题：一个值原来是A，变成了B，又变成了A。这个时候使用CAS是检查不出来变化的，但实际上却被更新了两次。

ABA问题的解决思路是在变量前追加上**版本号或者时间戳**。从JDK1.5开始，JDK的atomic包中提供了一个类`AtomicStampedReference`类来解决ABA问题

这个类的compareAndSet方法的·作用是实现检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果二者都相等，才使用CAS设置为新的值和标志。

```java
public boolean compareAndSet(V   expectedReference,
                             V   newReference,
                             int expectedStamp,
                             int newStamp) {
    Pair<V> current = pair;
    return
        expectedReference == current.reference &&
        expectedStamp == current.stamp &&
        ((newReference == current.reference &&
          newStamp == current.stamp) ||
         casPair(current, Pair.of(newReference, newStamp)));
}
```

### 5.2 循环时间长开销大

CAS多与自旋组合，如果自旋CAS长时间不成功，会占用大量的CPU资源。

解决思路是让JVM支持处理器提供的**pause指令**

pause指令能让自旋失败是cpu睡眠一小段时间在继续自旋，从而使得读操作的频率低很多，为解决内存顺序冲突而导致的CPU流水线重排的代价也会小很多。

### 5.3 只能保证一个共享变量的原子操作

有两种解决方案：

1. 使用JDK 1.5开始就提供的`AtomicReference`类保证对象之间的原子性，把多个变量放到一个对象里面进行CAS操作；
2. 使用锁。锁内的临界区代码可以保证只有当前线程能操作。