# 1.概述

## 1.1 Netty是什么

Netty是一个异步的、基于事件驱动网络应用框架，用于快速开发可维护性、高性能的网络服务器和客户端

### 1.2 Metty的地位

Netty在java网络应用框架中的地位就好比：Spring框架在javaEE开发中的地位

以下框架都使用了Netty，因为他们都有网络通信需求

- RocketMQ -ali开源的消息队列
- Hadoop -大数据分布式存储架构
- gRPC -rpc框架
- Dubbo -rpc框架
- Spring 5.x -flux api 完全抛弃了tomcat，使用netty作为服务器端
- Zookeeper -分布式协调框架

### 1.3 Netty的优势

- Netty vs NIO

  - 基于NIO
  - 减少工作量和bug

  - 解决TCP传输问题，如粘包、半包

  - 解决epoll空论导致CPU100%

  - 对API进行增强，使之更易用

# 2. Hello Word

### 服务器端

```java
public class HelloServer {
    public static void main(String[] args) {
        //1. 启动器，负责组装 netty 组件，启动服务器
        new ServerBootstrap()
                // 2. group组
                .group(new NioEventLoopGroup())
                // 3. 选择服务器的ServerSocketChannel实现
                .channel(NioServerSocketChannel.class) //OIO BIO
                // 4. 与nio类似，boss负责处理连接 worker（child）负责处理读写，决定了worker（child）能执行那些操作（handler）
                .childHandler(
                        //5. channel 嗲表和客户端进行数据读写操作的通道Initializer 初始化，负责添加别的handler
                        new ChannelInitializer<NioSocketChannel>() {
                            @Override
                            protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
                                // 6. 添加具体的 handler
                                nioSocketChannel.pipeline().addLast(new StringDecoder()); // 将ByteBuf转换为字符串
                                nioSocketChannel.pipeline().addLast(new ChannelInboundHandlerAdapter() { //自定义handler
                                    @Override
                                    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                        // 打印上一步转换好的字符串
                                        System.out.println(msg);
                                    }
                                });
                            }
                        })
                //6. 绑定监听端口
                .bind(8080);
    }
}
```

### 客户端

```java
public class helloClient {
    public static void main(String[] args) throws InterruptedException {
        //创建启动器类
        new Bootstrap()
                //2. 添加 EventLoop
                .group(new NioEventLoopGroup())
                //3. 选择客户端 channel实现
                .channel(NioSocketChannel.class)
                //4. 添加处理器
                .handler(new ChannelInitializer<NioSocketChannel>() {
                    @Override //建立连接后被调用
                    protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
                        nioSocketChannel.pipeline().addLast(new StringEncoder());
                    }
                })
                //5. 连接到服务器
                .connect(new InetSocketAddress("localhost",8080))
                .sync()
                .channel()
                //向服务器发送信息
                .writeAndFlush("hello,word");
    }
}
```

# 3. 组件

## 3.1 EventLoop

**事件循环对象**

Eventloop本质是一个单线程执行器（同时维护了一个Selector），里面有run方法处理Channel上源源不断的io事件。

他的继承关系比较复杂

- 一条线是继承自ScheduleExecutorService，因此包含了线程池中所有的方法
- 另一条是继承自netty自己的OrderedEventExecutor
  - 提供了boolean intEventloop（Threa thread）方法判断一个线程是否属于此Eventloop
  - 提供了parent方法来看自己属于那个EventloopGroup

**事件循环组**

EventLoopGroup是一组Eventloop，Channel一般会调用EventloopGroup 的register方法来绑定其中一个Eventloop，后续这个Channel 上的io事件都由此Eventloop 来处理（保证了io事件处理时的线程安全）

- 继承自Iterable接口提供遍历Eventloop的能力
- 另有一个next方法获取集合中下一个Eventloop

EventLoopGroup的使用：

```java

@Slf4j
public class testEventloop {
    public static void main(String[] args) {
        // 1. 创建事件事件循环组
        NioEventLoopGroup group = new NioEventLoopGroup(4);
        //2. 获取下一个事情循环对象
        System.out.println(group.next());
        System.out.println(group.next());
        //3. 执行普通任务
//        group.next().submit(()->{
//            try {
//                Thread.sleep(1000);
//            } catch (InterruptedException e) {
//                e.printStackTrace();
//            }
//            log.info("ok");
//        });
        //4. 执行定时任务
//        group.next().scheduleAtFixedRate(()->{
//            log.info("ok");
//        },0,1, TimeUnit.SECONDS);

        log.info("main");
    }
}


@Slf4j
public class EventloopServer {
    public static void main(String[] args) {
        //细分2：创建一个独立的EventLoopGroup
        DefaultEventLoopGroup group = new DefaultEventLoopGroup();
        new ServerBootstrap()
                //把Eventloop进一步划分 （boss和worker      如果某一个线程执行时间比较长可以独立出一个EventLoopGroup，让一个新的handler处理
                //细分1 boss只负责 ServerSocketChannel上accept事件 work值负责socketChannel事件
                .group(new NioEventLoopGroup(), new NioEventLoopGroup())
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
                        nioSocketChannel.pipeline().addLast("handler1",new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                ByteBuf buf = (ByteBuf) msg;
                                log.info(buf.toString(StandardCharsets.UTF_8));
                                ctx.fireChannelRead(msg); //让消息传递给下一个handler
                            }
                        }).addLast(group, "handler2", new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                ByteBuf buf = (ByteBuf) msg;
                                log.info(buf.toString(StandardCharsets.UTF_8));
                            }
                        });
                    }
                })
                .bind(8080);
    }
}
```



![image-20230826172321080](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308261723200.png)

![image-20230826174240098](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308261742231.png)

相当于每个Channel与Eventloop绑定后，就不会变了，后面每次调用Channel，都使用固定的Eventloop

**handler 如何执行切换线程**

如果两个handler绑定的是同一个线程，那么就直接调用，否则，把要调用的的代码封装为一个任务对象，由下一个handler的线程来调用

AbstractChannelHandlerContext.class:

~~~java
    static void invokeChannelRead(final AbstractChannelHandlerContext next, Object msg) {
        final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
        //判断下一个handler的事件循环是否与当前时间循环时是一线程
        EventExecutor executor = next.executor(); //返回下一个handler的Eventloop
        //是则直接调用
        if (executor.inEventLoop()) {
            next.invokeChannelRead(m);
        } 
        //不是则将要执行的代码作为任务交给下一个事件处理
        else {
            executor.execute(new Runnable() {
                public void run() {
                    next.invokeChannelRead(m);
                }
            });
        }
    }
~~~

## 3.2 Channel

### channel的主要作用

- close（）可以用来关闭channel
- closeFuture（）可以用来处理channel的关闭
  - sync方法作用是同步等待channel关闭
  - addListener（）方法时异步等待channel关闭
- pipline（）方法添加处理器
- write（）方法将数据写入
- writeAndFlush（）将数据写入并退出

### **connect的问题**

connect链接是异步链接的，不是在主线程，大约需要1s，而主线程会接着往下执行，如果直接获取channel的话，会获取到一个未连接的channel。

有两种解决方法：

1. 使用sync方法同步结果，阻塞主线程执行，直到连接成功
2. 使用addListener方法异步处理，连接建立好后会调用该方法。

```java
public static void main(String[] args) throws InterruptedException {
    //带有Future，Promise的类型都是和异步方法配套使用来处理结果    
    ChannelFuture channelFuture = new Bootstrap()
                .group(new NioEventLoopGroup())
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<NioSocketChannel>() {
                    @Override //建立连接后被调用
                    protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
                        nioSocketChannel.pipeline().addLast(new StringEncoder());
                    }
                })
                //5. 连接到服务器
                //异步非阻塞，当先调用connect方法的是main。发起了调用，真正执行connect连接的是nio线程（NioEventLoopGroup）
                .connect(new InetSocketAddress("localhost", 8080));//可能是会需要 1s后链接成功
        //如果没有sync，会无阻塞向下执行，会拿到还没建立好连接的channel。（测试把·sync（）删除，会报错，而sleep 3s，则可以成功。
    	//2.1使用cync 方法同步结果。
        channelFuture.sync();//阻塞当前线程，直到nio线程连接建立完毕
        Channel channel = channelFuture.channel();
        channel.writeAndFlush("sssssssssss");
    }
		//2.2 使用addListener（回调对象）方法异步处理结果
        channelFuture.addListener(new ChannelFutureListener() {
            @Override
            // 在nio线程连接建立好以后，会调用operationComplete
            public void operationComplete(ChannelFuture channelFuture) throws Exception {
                // 在nio线程建立好之后，会调用operationComplete
                Channel channel = channelFuture.channel();
                channel.writeAndFlush("hello word");
            }
        });
```

### close的问题

close操作同样是异步执行的，如果紧贴后面，可能会在close结束前执行其他操作，解决方法：

- 可以通过channel.closeFuture()来获取closeFuture
- 后续两种操作与上面类似

```java
public static void main(String[] args) throws InterruptedException {
    	//在主线程新建，可以在后续中使用、关闭
        NioEventLoopGroup group = new NioEventLoopGroup();
        ChannelFuture channelFuture = new Bootstrap()
                .group(group)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<NioSocketChannel>() {
                    @Override //建立连接后被调用
                    protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
                        nioSocketChannel.pipeline().addLast(new StringEncoder());
                    }
                })
                //5. 连接到服务器
                //异步非阻塞，当先调用connect方法的是main。发起了调用，真正执行connect连接的是nio线程（NioEventLoopGroup）
                .connect(new InetSocketAddress("localhost", 8080));//可能是会需要 1s后链接成功
        Channel channel = channelFuture.sync().channel();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while(true){
                    Scanner scanner=new Scanner(System.in);
                    String s=scanner.nextLine();
                    if(s.equals("q")){
                        channel.close(); //close是异步操作，如果直接在后面打印关闭，可能在它关闭之前关闭
                        log.info("紧随其后");
                        break;
                    }
                    channel.writeAndFlush(s);
                }
            }
        },"input").start();
        ChannelFuture closeFuture = channel.closeFuture();
//        closeFuture.sync();
//        log.info("处理关闭后的操作");
        closeFuture.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture channelFuture) throws Exception {
                log.info("处理关闭后的操作");//此时channel关闭了，但线程并没有真正的关闭，可以用一个可见的NioEventLoopGroup对象作为建立ChannelFuture时的参数，后自动关闭
                group.shutdownGracefully();
            }
        });
        log.info("addListener后面");
    }
```

![image-20230828091436039](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308280914150.png)

输出为这个，可见addListener不是立即执行的，而是等channel结束后再回过来调用，是在nioEventloopFroup-2-1线程内执行

### netty为什么异步

类似于医生的分责任

 ![image-20230828092532235](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308280925302.png)

每个线程负责部分的任务，其他线程负责处理其他的任务，这样可以充分利用每个线程，（提升了吞吐量而不是响应时间）。

如果是每个线程负责一个完整的任务，则吞吐量由线程数量决定，这时不能介绍的（访问数量可能很高，而线程远远小于访问数）

要点

- 单线程没法异步提高效率，必须配合多线程，多核cpu才能发挥异步的优势
- 异步并没有缩短响应时间，反而有所增加，但提高了吞吐量
- 合理进行任务拆分，也是利用异步的关键（若某一个线程负载过大，其他线程闲着，反而会降低效率

## 3.3 Future & Promise

在异步处理时，经常用到这两个接口

> netty中的Future与jdk中的Future同名，但是是两个接口，netty的Future继承自jdk的Future，而Promise又对netty Future进行了拓展
>
> - jdk Future只能同步等待任务结束（或成功，或失败）才能得到结果
> - netty Future 可以等待任务结束得到结果，也可以异步方式得到结果，但都是要等任务结束
> - netty Promise 不仅有netty Future的功能，而且脱离了任务独立存在，只作为两个线程间传递结果的容器

## 3.4 Handler & Pipline

ChannelHandler用来处理Channel上各种事件，分入站、出站两种。所有ChannelHandler被练成一串，就是Pipline

- 入站处理器通常是ChannelInboundhandlerAdapter的子类，主要用来杜取客户端数据，写回结果
- 出站处理器通常是ChannelOutboundHandlerAdapter的子类，主要对写回结果进行加工

> 比喻：每个Channel是一个产品的加工车间，Pipline是车间的流水线，ChannelHandler就是流水线上的各道工序，而后面的ByteBuf是原材料，经过很多工序加工：先经过一道道入站工序，再经过一道道出站工序最终变成产品

~~~java
@Slf4j
public class TestPipline {
    public static void main(String[] args) {
        new ServerBootstrap()
                .group(new NioEventLoopGroup())
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
                        //1. 通过channel拿到pipline
                        ChannelPipeline pipeline = nioSocketChannel.pipeline();
                        //2. 添加处理器 head -> h1 -> h2 -> h3 -> h4 -> h5 -> h6 -. tail
                        pipeline.addLast("h1", new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
                                log.info("1");
                                super.channelReadComplete(ctx);
                            }
                        });
                        pipeline.addLast("h2", new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
                                log.info("2");
                                super.channelReadComplete(ctx);
                            }
                        });
                        pipeline.addLast("h3", new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
                                log.info("3");
                                super.channelReadComplete(ctx);
                                nioSocketChannel.writeAndFlush(ctx.alloc().buffer().writeBytes("server...".getBytes()));
                            }
                        });
                         pipeline.addLast("h4",new ChannelOutboundHandlerAdapter(){
                             @Override
                             public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                                 log.debug("4");
                                 super.write(ctx, msg, promise);
                             }
                         });
                         pipeline.addLast("h5",new ChannelOutboundHandlerAdapter(){
                             @Override
                             public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                                 log.debug("5");
                                 super.write(ctx, msg, promise);
                             }
                         });
                         pipeline.addLast("h6",new ChannelOutboundHandlerAdapter(){
                             @Override
                             public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                                 log.debug("6");
                                 super.write(ctx, msg, promise);
                             }
                         });
                    }
                })
                .bind(8080);
    }
}
~~~

进展按照顺序，出站反序

![image-20230830093335280](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308300933340.png)

进站出站顺序：

![image-20230830094527000](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308300945094.png)

## 3.5 ByteBuf

对字节数据的封装

创建：

~~~java
ByteBuf buf=ByteBufAllocator.DEFAULT.buffer();
~~~

**直接内存 vs 堆内存**

默认情况直接内存

~~~java
// 创建池化基于堆的ByteBuf
ByteBuf bytebuf=ByteBufAllocator.DEFAULT.heapBuffer();
// 创建池化基于直接内存的ByteBuf
ByteBuf byteBuf = ByteBufAllocator.DEFAULT.directBuffer();
~~~

- 直接内存创建和销毁的代价昂贵，但读写性能高（少一次内存负责），适合配合池化功能一起用
- 直接内存对GC压力小，因为这部分内存不受JVM垃圾回收的管理，但也要注意即时主动释放

**池化 vs 非池化**

池化的最大意义在于可以重用ByteBuf，优点有：

- 没有池化，则每次都得创建新的ByteBuf实例，这个操作对直接内存代价昂贵，就算是堆内存，也会增加GC压力
- 有了池化，则可以重用池中的ByteBuf实例，并采用了与jemalloc类似的内存分配算法提升分配效率
- 并发时，池化功能更节约内存，减少内存溢出的可能

池化功能是否开启，可以通过设置系统环境变量来设置

-Dio.netty.allocator.type={unpooled|pooled}

- 4.1以后，非Android平台默认启用池化实现，Android平台气筒非池化实现
- 4.1以前，池化功能还不成熟，默认是非池化实现



**组成**

ByteBuf由四部分组成

![image-20230830102752926](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308301027979.png)

最开始**读写指针**都在0位置



**扩容**

如果如果量不够，会引发扩容

扩容规则：

- 如过写入后数据大小未超过512，则选择下一个16的整数倍，例如写入后大小为12，则扩容后是16
- 如果写入后数据大小超过512，则选择下一个2^n，例如写入后大小为523，则扩容后是1024
- 扩容不能超过max capacity，否则会报错

**读取**

读过的内容就属于废弃部分了，再读只能读那些尚未读取的部分

要重复读取，可以在read前做个标记 mark，后要重复读取的话，重置到标记位置reset

或者使用getapi

**回收**

由于netty中有堆外内存的ByteBuf实现，堆外内存最好是手动来释放，而不是等GC垃圾回收

- UnpooledHeapByteBuf 使用的是JVM内存，只需等待GC回收内存即可
- UnpooledDirectByteBuf使用的是直接内存，需要特色的方法来回收内存
- PooledByteBuf和他的子类使用了池化机制需要更复杂的规则来回收内存

Netty 这里采用了引用计数法来控制回收内存，每个ByteBuf都实现了 ReferenceCounted接口

- 每个ByteBuf对象的初始计数为1
- 调用release方法计数减1，如果计数为0，ByteBuf内存被回收
- 调用retain方法计数加1，表示调用者没用完之前，其他Handler即使调用了release也不会造成回收
- 当计数为0时，底层内存会被回收，这时即使ByteBuf对象还在，其他各个方法均无法正常使用

由最后拿到ByteBuf的Handler来释放内存

**slice**

【零拷贝】的体现之一，对原始ByteBuf进行切片成多个ByteBuf，切片后的ByteBuf并没有发生内存复制，还是使用原始的ByteBuf的内存，切片后的ByteBuf维护独立的read，write指针

