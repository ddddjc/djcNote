# NIO基础

non-blocking io 非阻塞IO

## 1. 三大组件

### 1.1 Channel & Buffer

channel有点类似于stream，他就说读写数据的**双向通道**，可以从channel将数据读入buffer，也可以将buffer的数据写入channel，而之前的stream要么是输入，要么是输出，channel比stream更为底层

常见的channel有：

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

buffer则用来缓冲读写数据，常见的buffer有

- ByteBuffer
  - MapperByteBuffer
  - DirectByteBuffer
  - HeapByteBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer
- CharBuffer

### 1.2 Selector

单从字面意思不好理解，需要结合服务器的实际演化来理解他的用途

#### 多线程版设计

![image-20230825085102267](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308250851329.png)

**多线程版缺点：**

- 内存占用高
- 线程上下文切换成本高
- 只适合连接数少的场景

#### 线程池版设计

![image-20230825085654505](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308250856594.png)

**线程池版缺点**

- 阻塞模式下，线程仅能处理一个socket连接
- 仅适合短连接场景

#### selector版设计

selector的作用是配合一个线程来管理多个channel，获取这些channel上发送的事情，这些channel工作在非阻塞模式下，不会让线程吊死在一个channel上，适合连接数特别多，但流量低的场景（low traffic）

![image-20230825090143975](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308250901056.png)

调用selector的select（）会阻塞channel发生了读写就绪事件，这些时间的发生，select方法就会返回这些时间交给thread来处理

## 2. ByteBuffer

### 2.1 ByteBuffer使用

1. 向buffer写入数据，例如调用channel.read(buffer)
2. 调用flip（）切换至读模式
3. 从buffer读取数据，例如调用buffer.get()
4. 调用chear（）或者compact（）切换至写模式
5. 重复1-4

### 2.2 ByteBuffer结构

ByteBuffer有以下重要属性

- capacity   容量
- position    读写指针
- limit           限制

一开始是空的，指针在前面

![image-20230825092953959](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308250929018.png)

写入模式下，position是写入位置，limit等于容量，

![image-20230825093109072](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308250931149.png)

flip动作发生后，position切换为读取位置，limit切换为读取限制

![image-20230825093246503](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308250932560.png)

读取四个字节后，状态

 ![image-20230825093421259](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308250934323.png)

clear动作发生后，状态（从头开始写）

![image-20230825093513709](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308250935777.png)

compact方法，是把未读完的部分向前压缩，然后切换至写模式

![image-20230825093637340](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308250936413.png)

### 2.3 常见方法

**分配空间**

可以使用allocate为ByteBuffer分配空间，其他类也有该方法

```java
        System.out.println(ByteBuffer.allocate(16).getClass());
        System.out.println(ByteBuffer.allocateDirect(16).getClass());
        /**
         * class java.nio.HeapByteBuffer          -java堆内存，读写效率较低，受到GC影响
         * class java.nio.DirectByteBuffer        -直接内存，读写效率高（少一次拷贝），不会收到GC影响，分配效率低
         */
```

**向buffer写入数据**

- 调用channel的read方法
- 调用buffer自己的put方法

```java
int readBytes=channel.read(buffer);
buffer.put((bute) 127)
```

**从buffer读取数据**

- 调用channel的write方法
- 调用buffer自己的get方法

get方法会让position读指针向后走，如果想重复读取数据

- 可以调用rewind方法将position重置为0
- 或者调用get（i）方法获取索引i的内容，不会移动指针

**mark和reset**

mark： 做一个标记，记录position位置

reset： 将position 重置到mark位置

**字符串与ByteBuffer转换**

~~~java
//字符串到buffer
//1
ByteBuffer buffer=ByteBuffer.allocate(16);
buffer.put(s.getBytes());
//2
ByteBuffer encode = StandardCharsets.UTF_8.encode(s);
//3 
ByteBuffer wrap = ByteBuffer.wrap(s.getBytes());
//buffer到字符串
String string = StandardCharsets.UTF_8.decode(buffer).toString();
~~~

**集中写**

```java
ByteBuffer buffer1= StandardCharsets.UTF_8.encode("hwllo");
        ByteBuffer buffer2= StandardCharsets.UTF_8.encode("world");
        ByteBuffer buffer3= StandardCharsets.UTF_8.encode("你好");
        try {
            FileChannel channel = new RandomAccessFile("wprds.txt", "rw").getChannel();
            channel.write(new ByteBuffer[]{buffer1,buffer2,buffer3});
        } catch (IOException e) {
        }
```

**分散读**

~~~java
try {
            FileChannel channel = new RandomAccessFile("words.txt", "r").getChannel();
            ByteBuffer allocate = ByteBuffer.allocate(5);
            ByteBuffer allocate1 = ByteBuffer.allocate(4);
            ByteBuffer allocate2 = ByteBuffer.allocate(6);
            channel.read(new ByteBuffer[]{allocate,allocate1,allocate2});
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        }
~~~

## 3. 文件编程

### 3.1 FileChannel

> FileChannel 只能工作在阻塞模式下

#### 获取：

不能直接打开FileChannel，必须通过FileInputStream、FileOutputStream或者RandomAccessFile来获取FileChannel，它们都有getChannel方法

- 通过FileInputStream获取的channel只能读
- 通过FileOutputStream获取的channel只能写
- 通过RandomAccessFile是否能读写根据构造RandomAccessFile是的读写模式决定

#### 读取

从channel读取数据填充ByteBuffer，返回值表示读到多少字节，-1表示到达了文件末尾

~~~java
int read = channel.read(buffer);
~~~

#### 写入

例如SocketChannel

```java
ByteBuffer buffer=...;
buffer.put(...) ;//数据
buffer.flip();  //切换模式

while(buffer.hasRemaining()){
    channel.write(buffer);
}
```

在while中调用channel.write是因为write方法并不能保证一次将buffer中的内容全部写入channel

#### 关闭

channel必须关闭，不过调用了FileOutputStream、FileInputStream或者RandomAccessFile的close方法会间接调用channel的close方法

#### 强制携入

操作系统处于性能的考虑，会将数据缓存，不是立即写入磁盘。可以调用force（true）方法将文件内容和元数据（文件的权限等信息）立刻写入磁盘

> channel.transferTo
>
> 会利用操作系统的零拷贝进行优化，效率高

### 3.2 Path

JDK 7引入了Path和Paths类

- Path 用来表示文件路径
- Paths是工具类，用来获取Path实例

```java
Path path = Paths.get("");
```

### 3.3 Files

![image-20230825143918063](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308251439157.png)

## 4网络编程

### 阻塞与非阻塞区别：

#### 阻塞：

- 阻塞模式下，相关方法都会导致线程暂停
  - ServerSocketChannel.accept会在没有连接时让线程暂停
  - SocketChannel.read会在没有数据可读时让线程暂停
  - 阻塞的表现其实时线程暂停了，展厅期间不会占用cpu，但线程相当于闲置
- 单线程模式下，阻塞方法之间相互影响，几乎不能正常工作，需要多线程支持
- 但多线程下，有新的问题，体现在一下方面
  - 32为jvm 一个线程 320k，64为jvm 一个线程1024k，如果链接数过多，必然导致OOM，并且线程太多，胆儿会因为频繁上下文切换导致性能降低
  - 可以采用线程池技术来减少线程数和线程上下文切换，但治标不治本，如果有很多连接建立，但长时间inactive，会阻塞线程池中所有线程，因此不适合长连接，只适合短连接。

#### 非阻塞：

- 非阻塞模式下，相关方法都不会让线程暂停
  - 在ServerSocketChannel.accept在没有连接建立时，会返回null，继续运行
  - SocketChannel.read在没有数据可读时，会返回0，但线程不必阻塞，可以去执行其他SocktChannel的Read或是去执行ServerSocketChannel.accept
  - 写数据时，线程只是等待数据写入Channel即可，无需等Channel通过网络把数据发送出去
- 但非阻塞模式下，即使没有连接建立，和可读数据，线程仍然在不断运行，拜拜浪费了cpu
- 数据复制过程中，线程实际还是阻塞的（AIO改进的地方）

#### 多路复用：

单线程可以配合Selector完成对多个Channel可读事件的监控，这称为多路复用

- 多路复用仅针对网络IO，普通文件IO没法利用多路复用
- 如果不用Selector的非阻塞模式，线程大部分时间都在做无用功，而Selector能够保证
  - 有可连接事件时才会去连接
  - 有可读事件才会读取
  - 有可写事件才会写入
    - 限于网络传输能力，Channel未必写时可用，会触发Selector的可写事件

```java
public class Server {
    public static void main(String[] args) throws IOException {
        //使用nio来理解阻塞模式，单线程
        //0. ByteBuffer
        ByteBuffer buffer=ByteBuffer.allocate(16);
        //1.创建服务器
        ServerSocketChannel ssc=ServerSocketChannel.open();
        ssc.configureBlocking(false); //默认true，是否阻塞
        //2. 绑定监听端口
        ssc.bind(new InetSocketAddress(8080));
        //3.连接集合
        List<SocketChannel> channels=new ArrayList<>();
        while(true){
            //4.accept建立与客户端连接，SocketChannel用来与客户端之间通信
            SocketChannel sc=ssc.accept();//非阻塞，线程还会继续运行，如果阻塞，则等待
            sc.configureBlocking(false);//对sc阻塞进行设置
            if(sc!=null){
                channels.add(sc);
                //code
            }
            for(SocketChannel channel:channels){
                int read=channel.read(buffer);//若非阻塞，线程仍会继续执行，若没有读到数据，read=-1；
                if(read>0){
                    //code
                }
            }
        }
    }
}
```

SelectionKey 的四种常量：

![image-20230825170015086](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308251700162.png)

- accept - 会在有连接请求时触发
- connect-是客户端，连接建立后触发
- read- 可读事件
- write- 可写时间

使用selector

~~~java
public class Server {
    public static void main(String[] args) throws IOException {
        //创建selector，管理多个chanel
        Selector selector = Selector.open();
        ByteBuffer buffer=ByteBuffer.allocate(16);
        ServerSocketChannel ssc=ServerSocketChannel.open();
        ssc.configureBlocking(false); //默认true，是否阻塞

        //建立selector和·channel的联系（注册
        //SelectionKey就是就将来事件发生后，通过它可以找到事件和哪个channel的事件
        SelectionKey register = ssc.register(selector, 0, null);
        //key只关注accept事件
        register.interestOps(SelectionKey.OP_ACCEPT);
        ssc.bind(new InetSocketAddress(8080));
        while(true){
            //select 方法，没有事件发生，线程阻塞，有事件发生，恢复运行
            //select 在事件为处理事，不会阻塞 (key.cancel()可以取消事件)
           selector.select();
           //处理事件   selectedKyes，包含了所有发生的事件
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            while(iterator.hasNext()){
                SelectionKey key = iterator.next();
                ServerSocketChannel channel =(ServerSocketChannel) key.channel();
                channel.accept();
            }
        }
    }
}
~~~

### 多线程实现

```java
public class MultiThreadServer {
    public static void main(String[] args) throws IOException {
        Thread.currentThread().setName("boss");
        ServerSocketChannel ssc = ServerSocketChannel.open();
        ssc.configureBlocking(false);
        Selector boss = Selector.open();
        SelectionKey bossKey = ssc.register(boss, 0, null);
        bossKey.interestOps(SelectionKey.OP_ACCEPT);
        ssc.bind(new InetSocketAddress(8080));
        Work[] works=new Work[10];
        for(int i=0;i< works.length;i++){
            works[i]=new Work("worker-"+i);
        }
        AtomicInteger idex=new AtomicInteger();
        while (true){
            boss.select();
            Iterator<SelectionKey> iterator = boss.selectedKeys().iterator();
            while(iterator.hasNext()){
                SelectionKey key = iterator.next();
                iterator.remove();
                if(key.isAcceptable()){
                    SocketChannel accept = ssc.accept();
                    accept.configureBlocking(false);
                    //round robin
                    works[(idex.getAndIncrement()% works.length)].register(accept);
                    //关联Selector
                    accept.register(works[(idex.getAndIncrement()% works.length)].selector,SelectionKey.OP_READ,null);

                }
            }
        }
    }
    static class Work implements Runnable{
        private Thread thread;
        private Selector selector;
        private String name;
        private boolean start=false;
        private ConcurrentLinkedQueue<Runnable> queue=new ConcurrentLinkedQueue<>();
        public Work(String name) {
            this.name = name;
        }
        public void register(SocketChannel sc) throws IOException {
            if (!start) {
                thread = new Thread(this, name);
                thread.start();
                selector = Selector.open();
                start=true;
            }
            queue.add(()->{
                try {
                    sc.register(selector,SelectionKey.OP_READ,null);
                } catch (ClosedChannelException e) {
                    e.printStackTrace();
                }
            });
        }
        @Override
        public void run() {
            while(true){
                try {
                    selector.select();
                    Runnable poll = queue.poll();
                    if(poll!=null){
                        poll.run();  //执行了注册         sc.register(selector,SelectionKey.OP_READ,null);
                    }
                    Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                    while(iterator.hasNext()){
                        SelectionKey key=iterator.next();
                        iterator.remove();
                        if(key.isReadable()){
                            ByteBuffer buffer=ByteBuffer.allocate(16);
                            SocketChannel channel= (SocketChannel) key.channel();
                            channel.read(buffer);
                            buffer.flip();
                            System.out.println(buffer.toString());
                        }
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

## 5. NIO vs BIO

### 5.1 stream vs channel

- Stream不会自动缓冲数据，channel会利用系统提供的发送缓冲池、接收缓冲池（更为底层）
- stream仅支持阻塞API，channel同时支持阻塞，非阻塞API，网络channel可配合Selector实现多路复用
- 二者均为全双工，即读写可以同时进行

### 5.2 IO模型

同步阻塞、同步非阻塞、同步多路复用、异步阻塞（没有此情况）、异步非阻塞

- 同步：线程自己去获取结果（一个线程）
- 异步：线程自己不去获取结果，而是又其他线程送结果（至少两个线程）

当调用一次channel.read或stream.read后，会切换至操作系统内核态来完成真正的数据读取，而读取又分为两个阶段：

- 等待数据阶段
- 复制数据阶段

![image-20230826110341695](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308261103771.png)

- 阻塞IO
- 非阻塞IO
- 多路复用
- 信号驱动
- 异步IO

###  5.3 零拷贝

传统IO问题

传统IO将一个文件通过Socket写出

内部工作流程：

![image-20230826112045617](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308261120698.png)

1. java本身并不具备IO读写能力，因此read方法调用后，要从java程序的用户态切换至内核态，去调用操作系统（Kernel）的读写能力，将数据读入内核缓冲区。这期间用户线程阻塞，操作系统使用DMA（Direct Memory Access）来实现文件读，期间也不会使用cpu

   > DMA也可以理解为硬件单元，用来释放cpu完成文件IO

2. 从内核态切换回用户态，将数据从内核缓冲区读入用户缓冲区（即 byte[] buf），这期间cpu会参与拷贝，无法利用DMA

3. 调用Write方法，这时将数据从用户缓冲区（byte[] buf）写入socket缓冲区，cpu会参与拷贝

4. 接下来要将网卡写数据，这项能力java又不具备，因此有得从用户态切换至内核态，调用操作系统的写能力，使用DMA将socket缓冲区的数据写入网卡，不会使用cpu



可以看到中间环节较多，java的IO实际不是物理设备的读写，而是缓存的复制，底层的真正读写是操作系统来完成的

- 用户态与内核态的切换发生了3次，这个操作比较重量级
- 数据拷贝了共4次

**NIO优化**

通过DirectBytrBuf

- ByteBuffer.allocate(10) HeapByteBuffer 使用的还是java内存
- ByteBuffer.allocateDirect(10) DirectByteBuffer 使用的是操作系统内存

![image-20230826113421770](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308261134849.png)

大部分步骤与优化前相同。唯有一点，java可以使用DirectByteBuf将堆外内存映射到jvm内存中来终结访问使用

- 这块内存不受jvm垃圾回收影响，因此内存地址固定，有助于IO读写
- java中的DirectByteBuf 对象仅维护了内存的需引用，内存回收分为两步：
  - DirectByteBuf 对象被垃圾回收，将虚引用加入引用队列
  - 通过专门线程访问引用队列，根据虚引用释放堆外内存
- 减少了一次数据拷贝，用户态与内核的切换次数没有减少

进一步优化（底层采用了 linux2.1 后提供的sendFile方法），java中对应着两个channel调用 transfer/transferFrom 方法拷贝数据

![image-20230826114313265](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308261143334.png)

1. java调用transferTo方法后，要从java程序的用户态切换至内核态，使用DMA将数据读入缓冲区，不会使用cpu
2. 数据从内核缓冲区传输到socket缓冲区，cpu会参与拷贝
3. 最后使用DMA将socket缓冲区的数据写入网卡，不糊使用cpu

可以看到

- 只发生了一次用户态与内核态的切换
- 数据拷贝了3次

进一步优化（linux2.4）

![image-20230826114652739](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308261146831.png)

1. java调用transferTo方法后，要从java程序的用户态切换至内核态，使用DMA将数据读入内核缓冲区，不会使用cpu
2. 只会将一些offset和length信息拷入socket缓冲区，几乎无消耗
3. 使用DMA将内核缓冲区的数据写入网卡，不会使用cpu

整个过程仅发生了一次用户态与内核态的切换，数据拷贝了2次。所谓零拷贝，并不是真正的无拷贝，而是在不会拷贝重复数到居民内存中。零拷贝的优点有：

- 更少的用户态与内核态的切换
- 不利用cpu计算，减少cpu缓存伪共享
- 零拷贝适合小文件传输。

### 5.4 AIO

AIO用来解决数据复制阶段的阻塞问题

- 同步意味着，在进行读写操作时，线程需要等待结果，还是相当于闲置
- 异步意味着，在进行读写操作时，线程不必等待结果，而是将来由操作系统来通过回调方式由另外的线程来获得结果

> 异步模型需要底层操作系统（Kernel）提供支持
>
> - windows 系统通过IOCP实现了真正的异步IO
> - Linux系统异步IO在2.6版本引入，但其底层实现还是用多路复用模拟了异步IO，性能没有优势

#### 文件AIO

#### 网络AIO

