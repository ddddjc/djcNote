# Http工作原理
HTTP协议是浏览器与服务器之间的数据传送协议。作为应用层协议，HTTP是基于TCP/IP
协议来传递数据的（HTML文件、图片、查询结果等），HTTP协议不涉及数据包
（Packet）传输，主要规定了客户端和服务器之间的通信格式。
![](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/20230712103325.png)
Http请求处理过程：
1. 用户通过浏览器进行一个操作，浏览器获取了事件
2. 浏览器向服务器发送TCP连接请求
3. 服务器接受浏览器的连接请求，经过TCP三次握手建立连接
4. 浏览器将请求数据打包成一个HTTP协议格式的数据包
5. 浏览器将该数据包推入网络，数据包经过网络传输，最终达到端服务程序。
6. 服务器拿到数据包后，以HTTP格式进行解包，获取服务端信息
7. 得到信息后进行处理
8. 服务器将响应结果按照HTTP协议格式打包
9. 服务器将响应数据推入网络，数据经网络传输最终达到浏览器
10. 浏览器拿到数据包后，以HTTP协议格式进行解包，然后解析数据
11. 浏览器将数据展示。
Tomcat主要的工作：主要是接受连接、解析请求数据、处理请求和发送响应这几个步骤。
# Tomcat整体架构
## 1. HTTP服务器请求处理
浏览器发给服务器端的是一个HTTP格式的请求，HTTP服务器收到后，会调用服务端程序（自己写的ava类)来处理。一般来说不同的请求需要不同的java类来吃处理。
1. 若是Http直接调用具体的业务类，会造成较高的耦合。![image.png](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/20230712105009.png)
2. 而图二：HTTP服务器不直接调用业务类，而是把请求交给容器来处理，容器通过Servlet接口调用业务类。因此，Servlet接口和Servlet容器的出现，达到了HTTP服务器与业务解耦合的目的。而Servlet接口和Servlet容器这一整套规范叫作Servlet规范。Tomcat按照Servlet规范的要求实现了Servlet容器，同时它们也具有HTTP服务器的功能。作为Java程序员，如果我们要实现新的业务功能，只需要实现一个Servlet，并把它注册到Tomcat（Servlet容器）中，剩下的事情就由Tomcat帮我们处理了
## 2. Servlet容器工作流程
为了解耦合，HTTP容器不直接调用Servlert，而是把请求交给Servlet容器来处理。
当客户端请求某个自愿时，HTTP服务器会把信息封装成一个ServletRequest对象，然后调用Servlet容器的Service方法。Servlet容器拿到请求后，根据URL和Servlet的映射关系，找到相应的Servlet（参考web项目的web.xml配置），如果Servlet没有被加载，则用反射机制创建一个Servlet调用init方法来完成初始化，后调用Servlet的·Service方法，处理请求。把ServletResponse对象返回给Http服务器，HTTP服务器会把相应发给客户端。
![image.png](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/20230712110405.png)

```xml
<servlet>  
	<servlet-name>UseRunResearchEvaluation</servlet-name>  
	<servlet-class>edu.isi.karma.webserver.temporarily.UseRunResearchEvaluationServlet</servlet-class>  
</servlet>  
<servlet-mapping>  
	<servlet-name>UseRunResearchEvaluation</servlet-name>  
	<url-pattern>rice/useRunResearchEvaluation</url-pattern>  
</servlet-mapping>
```
## 3.Tomcat整体架构
Tomcat两个核心功能：
- 处理Socket链接，负责网络字节流与Request和Response对象的转化
- 加载和管理Servlet，以及处理Request请求。
于是Tomcat设计了两个核心组件连接器（Connector）和容器（Container）来分别做这两件事情，连接器负责对外交流，容器负责内部处理。
![image.png](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/20230712110925.png)


# 连接器-Coyote
## 1. 架构介绍
Coyote是Tomcat服务器提供的供客户端访问的外部接口，客户端通过Coyote与服务器建立连接、发送请求并接收响应。
Coyote 封装了底层的网络通信（Socket 请求及响应处理），为Catalina 容器提供了统一的接口，使Catalina 容器与具体的请求协议及IO操作方式完全解耦。Coyote 将Socket 输入转换封装为 Request 对象，交由Catalina 容器进行处理，处理请求完成后, Catalina 通过Coyote 提供的Response 对象将结果写入输出流 。
Coyote主要负责具体协议的解析与IO的相关操作。与Servlet无直接关系。他的Request和Response也未实现Servlet规范对应的接口，而是在Catalina中将他们封装为ServletRequest和ServletResponse。
![image.png](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/20230712113651.png)
## 2.  Coyote支持的IO模型与协议
IO模型：(参考[[基础/小知识点/网络IO|网络IO]])

- NIO  ：非阻塞I/O，采用JAVA NIO类库实现
- NIO2：异步I/O，采用JDK 7 最新NIO3类库实现
- APR ：采用Apache可移植库实现，是c/c++编写的本地库。如果采用该方案，需要单独按照APR库。
应用层协议：
- HTTP/1.1：这是大部分web已用常用的访问协议
- AJP：用于web服务器集成（如Apache），实现对静态资源的优化以及集群部署，当前支持AJP/1.3.
- HTTP /2：HTTP 2.0大幅度提升了Web性能，下一代HTTP协议，自8.5以及9.0版本之后支持。
![image.png](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/20230712120148.png)
## 3. 连接器组件
![image.png](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307121442379.png)

连接器各个组件中的作用：

### EndPoint

1. EndPoint : Coyote通信端点，即通信监听的接口，是具体Socket接收和发送处理器，是对传输层的抽象，因此EndPoint用来实现TCP/IP协议。
2. Tomcat 并没有EndPoint 接口，而是提供了一个抽象类AbstractEndpoint ， 里面定义了两个内部类：Acceptor和SocketProcessor。Acceptor用于监听Socket连接请求。SocketProcessor用于处理接收到的Socket请求，它实现Runnable接口，在Run方法里调用协议处理组件Processor进行处理。为了提高处理能力，SocketProcessor被提交到线程池来执行。而这个线程池叫作执行器（Executor)

### Processor

1. 读取EndPoint传来的Socket字节流，并解析为Request对象，把Request和Response对象通过Adapter将其提交到容器处理。Processor是对应用层协议的抽象

### ProtocolHandler

ProtocolHandler : Coyote 协议接口，通过Endpoint和Processor，实现对具体协议的处理能力。。Tomcat 按照协议和I/O 提供了6个实现类 ： AjpNioProtocol ，AjpAprProtocol， AjpNio2Protocol ， Http11NioProtocol ，Http11Nio2Protocol ，Http11AprProtocol。我们在配置tomcat/conf/server.xml 时 ， 至少要指定具体的ProtocolHandler , 当然也可以指定协议名称 ， 如 ： HTTP/1.1 ，如果安装了APR，那么将使用Http11AprProtocol ， 否则使用 Http11NioProtocol 。

### Adapter

由于客户端发过来的请求信息不尽相同，Tomcat定义了自己的Request类来“存放”这些请求信息ProtocolHandler接口负责解析请求并生成Tomcat Request类，也就意味着，不能用TomcatRequest作为参数来调用容器。Tomcat设计者的解决方案是引入CoyoteAdapter，这是适配器模式的经典运用，连接器调用CoyoteAdapter的Sevice方法，传入的是TomcatRequest对象，CoyoteAdapter负责将Tomcat Request转成ServletRequest，再调用容器的Service方法。

# 容器 - Catalina

Catalina是Servlet的容器实现，包含了之前的所有容器组件，以及安全、会话、集群、管理等Servlet容器架构。通过松耦合的方式集成Coyote，以完成按照请求协议进行数据读写。

## 1. Catalina地位

![image-20230712150942774](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307121509855.png)

Tomcat本质上是一款Servlet容器，因此Catalina才是Tomcat的核心，其他模块都是为Catalina提供支撑服务的。比如：

通过Coyote模块提供链接通信，Jasper模块提供Jsp引擎，Naming提供JNDI服务，Juli提供日志服务。

## 整体结构

![image-20230712154029315](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307121540407.png)

Catalina负责管理Server，而Server表示着整个服务器。Server下面有多个Service，每个服务都包含着多个连接器组件Connector（Copyote 实现）和一个容器组件Cpontainer。在Tomcat启动的时候，会初始化一个Catalina的实例

Catalina各组件的职责：

| 组件      | 职责                                                         |
| --------- | ------------------------------------------------------------ |
| Catalina  | 负责解析Tomcat的配置文件，以此来创建服务器Server组件，并根据命令来对其进行管理 |
| Server    | 服务器表示整个Catalina Servlet容器以及其组件，负责组装并启动Servlet引擎，Tomcat连接器。Servlet通过实现Lifecysle接口，提供了一种优雅地启动和关闭整个系统的方式 |
| Service   | 服务是Server内部的组件，一个Server包含多个Service。它将若干个Connector组件绑定到一个Container（Engine）上 |
| Connector | 连接器，处理与客户端的通信，它负责接收客户请求，然后转给相关的容器处理，最后向客户返回响应结果 |
| Container | 容器，负责处理用户的servlet请求，并返回对象给web用户的模块   |
## Container结构
Tomcat设计了4种容器，分别是Engine、Host、Context和Wrapper。这四种容器不是平行关系，而是父子关系。Tomcat通过一种分层的架构，使的Servlet容器具有很好的灵活性。
![image.png](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307121617281.png)
各组件含义：

| 容器    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| Engine  | 表示整个Catalina的Servlet引擎，用来管理多个虚拟站点，一个Service最多只能有一个Engine，但一个引擎可包含多个Host |
| Host    | 代表一个虚拟主机，或者说一个站点，可以给Tomcat配置多个虚拟主机地址，而一个虚拟主机下可包含多个Context |
| Context | 表示一个Web应用程序，一个Web应用可能包含多个Wrapper          |
| Wrapper | 表示一个Servlet，Wrapper作为容器中的最底层，不能包含字容器   |

我们也可以再通过Tomcat的server.xml配置文件来加深对Tomcat容器的理解。Tomcat采用了组件化的设计，它的构成组件都是可配置的，其中最外层的是Server，其他组件按照一定的格式要求配置在这个顶层容器中

```xml
<Server> 
    <Service>
        <Connector/>
        <Connector/>
        <Engine>
        	<Host>
            	<Context></Context>
            </Host>
        </Engine>
    </Service>
</Server>
```

这些容器具有父子关系，形成一个树形结构。Tomcat用组合模式来管理这些容器。具体的实现方法有，每个容器组件都实现Container接口，可以使得用户对单容器对象和组合容器对象的·使用具有一致性。在这里的但容器指最底层的Wrapper，组合容器对象指上面的Context、Host或者Engine。

![image-20230712163615903](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307121636020.png)



# Tomcat启动流程

![image-20230712164713580](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307121647661.png)

#  请求处理流程

![image-20230712173341726](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307121733799.png)

确定到Servlet来处理：通过mapper组件来确定Servlet。

原理：Mapper组件保存了Web应用的配置信息，其实就是容器组件与访问路径的映射关系，比如Host容器里配置的域名、Context容器里的Web应用路径，以及Wrapper容器里Servlet映射的路径。当一个请求到来时，Mapper组件通过解析请求URL里的域名和路径，再到自己的Mapper里查找，就能定位到一个Servlet。如上图。

![image-20230712173757340](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307121737418.png)

从tomcat架构层次分析

1. Connector组件EndPoint中的Acceptor监听客户端套接字连接并接受Socket
2. 将连接交给线程池Executor处理，开始执行请求响应任务
3. Processor组件读取消息报文，解析请求行、请求体、请求头，封装成Request对象。
4. Mapper组件根据请求行的URL值和请求头的Host值匹配由哪个Host容器、Context容器、Wrapper容器处理请求
5. CoyoteAdaptor组件负责将Connector组件和Engine容器关联起来，把生成的Request对象和相应对象Response传递到Engine容器里中，调用Pipeline。
6. Engine容器的管道开始处理，管道中包含若干个Value、每个Value负责部分处理逻辑。执行Value后会执行基础的Value--StandardEngineValue，负责调用Host容器的Pipeline。
7. Host容器的管道开始处理，流程类似，最后执行Context容器的Pipeline
8. Context容器的管道开始处理，流程类似，最后执行Wrapper容器的Pipeline
9. Wrapper容器额的管道开始处理，流程类似，最后执行Wrapper容器对应的Servlet对象的处理方法。

