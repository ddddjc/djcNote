# Netty进阶

## 1. 粘包与半包

- 粘包：发送方连续发送多个小数据包时，接收方可能会将他们合并成一个大的数据包
- 半包：发送方发送的数据长度大于接收方的缓冲区长度时，接收方无法完整接收数据包，导致数据的接收不完整

**TCP层面：**

**问题**

- TCP以一个段为单位，每发送一个段就需要进行一次应答处理，但如果这么做，缺点是包的往返时间越长，性能越差

**解决**

- 进入了窗口概念，窗口大小即决定了无需等待应答而可以继续发送的数据最大值
- 窗口十几件IU起到了一个缓冲区的作用，同时也能起到流量控制作用
  - 窗口内的数据允许被发送
  - 当有一个应答到达，窗口向下滑动，可以发生别的数据
  - 应答未到达之前，窗口停止滑动

滑动窗口可能会导致粘包和半包问题。

- 窗口内数据一起发送，导致粘包
- 窗口容量不足，导致半包

### 现象分析：

粘包：

- 现象： 发送 abc      def ，接收abcdef
- 原因
  - 应用层：接收方ByteBuf设置太大（Netty默认1024）
  - 滑动窗口：假设发送方256bytes 表示一个完整的报文，但由于接收方处理不及时且窗口大小足够大，这256bytes字节就会缓冲在接收方的滑动窗口中，当滑动窗口中缓冲了多个报文就会粘包
  - Nagle算法：会造成粘包

半包

- 现象，发送 abcdef ，接收abc   def
- 原因
  - 应用层：接收方ByteBuf小于实际发送数据量
  - 滑动窗口：假设接收方的窗口只剩了128bytes，发送方的报文大小是256bytes，这时放不下了，只能先发送前128bytes，等待ack（回应）后才能发送剩余部分，这就造成了半包
  - MSS限制：当发送的数据超过MSS限制后，会将数据切分发送，就会造成半包

本质是因为TCP是流式协议，消息无边界

### 解决：

#### 1. 短连接

发送完一次就断开，不会造成粘包。

#### 2. 定长

使用解码器：FixedLengthFrameDecoder

#### 3. 换行符

使用解码器：LineBasedFrameDecoder（以换行符作为分割，有最大程度限制）、DelimiterBasedFrameDecoder（可以自定义分隔符和最大长度）

#### 4. 

#### LengthFieldBasedFrameDecoder

## 2. Netty提供的协议

#### http

~~~java
@Slf4j
public class TestHttpServer {
    public static void main(String[] args) {
        NioEventLoopGroup boss = new NioEventLoopGroup();
        NioEventLoopGroup worker = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.channel(NioServerSocketChannel.class);
            serverBootstrap.group(boss, worker);
            serverBootstrap.childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                protected void initChannel(SocketChannel ch)  {
                    ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                    ch.pipeline().addLast(new HttpServerCodec());          //codec 包含编码和解码
                    ch.pipeline().addLast(new SimpleChannelInboundHandler<HttpRequest>() {   //选择解码后的类型
                        @Override
                        protected void channelRead0(ChannelHandlerContext channelHandlerContext, HttpRequest httpRequest) throws Exception {
                            log.debug(httpRequest.uri());
                            DefaultFullHttpResponse response = new DefaultFullHttpResponse(httpRequest.protocolVersion(), HttpResponseStatus.OK);
                            byte[] bytes = "<h1>hello word</h1>".getBytes();
                            response.content().writeBytes(bytes);
                            response.headers().setInt(CONTENT_LENGTH,bytes.length);  //告诉浏览器长度，否则会一直等待·接收
                            channelHandlerContext.writeAndFlush(response);
                        }
                    });
                }
            });
            ChannelFuture channelFuture = serverBootstrap.bind(8080).sync();
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            boss.shutdownGracefully();
            worker.shutdownGracefully();
        }
    }
}
~~~

### 自定义协议

#### 要素

- 魔数，用来在第一时间判定是否是无效数据包
- 版本号，可以支持协议的升级
- 序列化算法，消息正文到底采用哪种序列化反序列化方式，可以由此拓展，例如：json、protobuf、hessian，jdk
- 指令类型，是登录，注册，单聊，群聊。。。跟业务相关
- 请求序号，为了双工通信，提供异步功能
- 正文长度
- 消息正文

~~~java
public class MessageCodec extends ByteToMessageCodec<Message> {
    @Override
    protected void encode(ChannelHandlerContext ctx, Message message, ByteBuf out) throws Exception {
        //1. 4字节魔数
        out.writeBytes(new byte[]{1,2,3,4});
        //2. 1字节的版本
        out.writeByte(1);
        //3。 1字节的序列化方式 jdk 0，json 1
        out.writeByte(0);
        //4. 1字节的指令类型
        out.writeByte(message.getMessageType());
        //5. 4个字节请求序号
        out.writeByte(message.getSequenceId());
        // 对齐填充用，无意义
        out.writeByte(0xff);
        //6. 获取内容的字节数组
        ByteArrayOutputStream bos=new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(message);
        byte[] bytes=bos.toByteArray();
        //7. 长度
        int len=bytes.length;
        out.writeInt(len);
        out.writeBytes(bytes);
    }

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        int magicNum = in.readInt();
        byte version = in.readByte();
        byte serializerType = in.readByte();
        byte messageType = in.readByte();
        int sequenceId = in.readInt();
        in.readByte();
        int length = in.readInt();
        byte[] bytes=new byte[length];
        in.readBytes(bytes,0,length);
        if (serializerType==0){
            ObjectInputStream ois=new ObjectInputStream(new ByteArrayInputStream(bytes));
            Message message = (Message) ois.readObject();
        }
    }
}
~~~



# 优化与源码

## 1.优化

### backlog

serverBootstrap.option(*ChannelOption*.SO_BACKLOG,size);

## 2.源码

![image-20230901080528734](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202309010805891.png)



