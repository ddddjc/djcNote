# volatile

## 1. 几个基本概念

### 1.1. 内存可见性

内存可见性指线程之间的可见性，当一个线程修改了共享变量时，另一个线程可以读取到这个修改后的值。

### 1.2. 重排序

为优化程序性能，对原有的指令执行顺序优化重新排序。重排序可能发生在多个阶段，比如编译重排序、CPU重排序等

### 1.3. happens-before规则

是一个重新许愿使用的规则，只要程序员在写代码时遵循happens-before规则，JVM就能保证指令在多线程之间的顺序性符合程序员的预期

## 2. volatile的内存语义

在java中，volatile关键字有特色的内存语义。volatile主要有以下两个功能：

- 保证变量的**内存可见性**
- 禁止volatile变量与普通变量重排序 

### 2.1内存可见性

示例代码：

~~~java
public class VolatileExample {
    int a = 0;
    volatile boolean flag = false;

    public void writer() {
        a = 1; // step 1
        flag = true; // step 2
    }

    public void reader() {
        if (flag) { // step 3
            System.out.println(a); // step 4
        }
    }
}
~~~

在这里，使用了volatile关键字修饰了一个boolean类型的变量flag

所谓内存可见性，指当一个线程对volatile修饰的变量进行写操作（如step2）使，JMM会立即把该线程对应的本地内存中的共享变量的值刷新到主内存中；当一个线程对volatile修饰的变量进行读操作（如step3）时，JMM立即把该线程对应的本地内存设置为无效，从主内存中读取共享变量

> 在这一点上，volatile与锁具有相同的内存效果，volatile变量的写和锁的释放具有相同的内存语义，volatile变量的读和锁的获取具有相同的内存语义。

假设在时间线上，线程A线执行writer方法，线程B后执行reader方法。那必然会有下图：

![volatile内存示意图](http://concurrent.redspider.group/article/02/imgs/volatile%E5%86%85%E5%AD%98%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg)

而如果flag变量没有用volatile修饰，在step2，线程A的本地内存里面的变量就不会立即跟新到主内存，那随后线程B也同样不会去主内存拿最新值，仍然使用线程B本地内存缓存的变量值a=0，flag=false

### 2.2 禁止重排序

在JSR-133以前的java内存模型中，是允许volatile变量与普通变量重排序的。上面的代码就可能重排序为：

1. 线程A写volatile变量，step2,设置flag为true，**但此时a未写到公共内存，或者整个操作被重排序到后面**
2. 线程B读同一个volatile，step3，读取到flag为true；
3. 线程读普通变量，step4，读取到a=0；
4. 线程A修改普通变量，step1，设置a=1；

可见，如果volatile变量与普通变量发生了重排序，虽然volatile变量能保证内存可见性，也可能导致普通变量读取错误。

所以在就内存模型中，volatile的写-读操作就不能与锁释放-获取具有相同的内存语义了·。为了提供一种比锁更轻量级的线程间的通信机制，JSR-133专家决定增强volatile的内存语义：严格限制编译器和处理器对volatile变量与普通变量的重排序。

编译器还好说，JVM是如何限制处理器的重排序呢？它是通过**内存屏障**来实现

**什么是内存屏障？**

硬件层面，内存屏障分为两种：读屏障和写屏障。内存屏障有两个作用：

1. 阻止屏障两侧的指令重排序
2. 强制把写缓冲区/高速缓冲区中的脏数据等写会主内存，或者让缓存中相应的数据生效

> 这里的缓存主要指的是CPU缓存（cache）

编译器在生`字节码时`，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。编译器选择了一个比较保守的JMM内存屏障插入策略，这样可以保证在任何处理器平台，任何程序中都能得到正确的volatile内存语义。这个策略是：

- 在每个volatile写操作前插入一个StoresStores屏障；
- 在每个volatile写操作或插入一个StoreLoad屏障；
- 在每个volatile读操作后插入一个LoadLoad屏障
- 在每个volatile读操作后再插入一个LoadStore屏障

![内存屏障](http://concurrent.redspider.group/article/02/imgs/%E5%86%85%E5%AD%98%E5%B1%8F%E9%9A%9C.png)

> 对这几个屏障的解释：
>
> Load代表读操作，Store代表写操作
>
> **LoadLoad屏障**：对于这样的语句Load1；LoadLoad；Load2，在Load2及后续操作要读取数据被访问前，保证Load1要读取的数据被读取完。
>
> **StoreStore屏障**：对于这样的语句Store1；StoreStore；Store2，在Store2及后续操作写入操作执行前，这个屏障会把Store1强制刷新到内存，保证Store1的写入操作对其他处理器可见。
>
> **LoadStore屏障**：对于这样的语句Load1; LoadStore; Store2，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。
> **StoreLoad屏障**：对于这样的语句Store1; StoreLoad; Load2，在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。它的开销是四种屏障中最大的（冲刷写缓冲器，清空无效化队列）。在大多数处理器的实现中，这个屏障是个万能屏障，兼具其它三种内存屏障的功能

对与连续多个volatile变量读或者连续多个volatile写，编译器做了一定的优化来提高性能，比如：

- 第一个volatile读
- LoadLoad屏障
- 第二个volatile读
- LoadStore屏障

再介绍一些volatile与普通变量的重排序规则：

1. 如果第一个操作时volatile读，那么无论第二个操作是什么，都不能重排序；
2. 如果第二个操作是volatile写，那么无论第一个操作是什么，都不能重排序；
3. 如果第一个操作是volatile写，第二个操作是volatile读，那么不能重排序。

## 3. volatile的用途

从volatile的内存语义来说，volatile可以保证内存可见性且禁止重排序。

在保证内存可见性这一点是，volatile有着与锁相同的内存语义，所以可以作为一个“轻量级”的锁来使用。但由于volatile仅仅保证单个volatile变量的读、写操作具有原子性，而锁可以保证整个临界区代码的执行具有原子性。所以**功能上，锁比volatile更强大；在性能上，volatile更有优势**

禁止重排序这一点上，volatile也是非常有用。比如我们熟悉的单例模式。其中有一种实现方法是“双重锁检查”，比如

```java
public class Singleton {

    private static Singleton instance; // 不使用volatile关键字

    // 双重锁检验
    public static Singleton getInstance() {
        if (instance == null) { // 第7行
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton(); // 第10行
                }
            }
        }
        return instance;
    }
}
```

这如果这里的变量不使用volatile关键字，是可能会发生错误的。他可能重排序：

~~~java
instance = new Singleton(); // 第10行

// 可以分解为以下三个步骤
1 memory=allocate();// 分配内存 相当于c的malloc
2 ctorInstanc(memory) //初始化对象
3 s=memory //设置s指向刚分配的地址

// 上述三个步骤可能会被重排序为 1-3-2，也就是：
1 memory=allocate();// 分配内存 相当于c的malloc
3 s=memory //设置s指向刚分配的地址
2 ctorInstanc(memory) //初始化对象
~~~

而一旦发生了这样的重排序，比如线程A在第10行执行了步骤1和步骤3，但是步骤2还没有执行完。这个时候另一个线程B执行到第七行，他会判定instance不为空，然后直接返回了一个未初始化完成的instance。