# 一、会话(Session）
**会比较绕，一般记住中文 会话 来指一次有始有终的会话，会话机制来指保持状态的方案，session来指解决方案的存储结构以及语言里的内容**
- Session，中文翻译为会话，[会话 (计算机科学) - 维基百科](https://zh.wikipedia.org/wiki/%E4%BC%9A%E8%AF%9D_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))
- 原本含义：有始有终的一系列动作/消息。例如：见面聊天时从相遇到分手这一系列过程可以称之为会话（session）,又如浏览器窗口打开到关闭的过程 。
- 当session一词与网络协议相关联时，往往隐含了“面向连接和”和/或“保持状态”这样两个含义。**”面向连接“** 指在通讯双方通讯之前要先建立一个通讯的渠道，比如打电话，只有对方接了通讯才开始。**"保持状态"** 指通讯一方可以把一系列消息关联起来，使消息之间可以相互依赖（识别消息是谁发送的如一个TCP session即一个TCP socket）。
- 到了web服务器蓬勃发展时代，session又有了新的拓展，指客户端与服务器端保持状态的解决方案。有时候（中文语境下更常用）session也指这种解决方案的存储结构。如“把xxx保存在session里。”  由于各种用于web开发的语言在一定程度上都提供了对这种解决方案的支持，所以在某种特定语言的语境下，session也被用来指代该语言的解决方案，比如经常把Java里提供的javax.servlet.http.HttpSession简称为session。
# 二、 会话机制
- 前提：HTTP协议本身是无状态的，这与HTTP协议本来的目的是相符的，并不会记录过去的行为，每次请求都是独立的。[[基础/计算机网络/HTTP 协议详解|HTTP 协议详解]]
- **会话机制** 是Web程序中常用的技术，用来跟踪用户的这个会话。常用的会话跟踪技术是Cookie和Session。Cookie通过在客户端记录信息确定用户身份，Session通过在服务器端记录信息确定用户身份。
	- Cookie和Session的区别：
		- Cookie相当于客户的凭证，而Session相当于服务端的数据。
		- 只用Cookie：相当于集字卡，集够足够的章时，可以换取奖品，消费一次，客户端给卡盖一个章（客户端修改Cookie），但是可能会出现自己盖假章（修改token）、转让、被盗卡（Cookie劫持）的情况；
		- 两者结合，相当于Cookie保存一个标识，客户端通过标识来在Session中访问用户信息。
- 由于HTTP协议是无状态的，而出于种种考虑也不希望使之成为有状态的，因此这两种方案就成为现实的选择。具体来说cookie机制采用的是在客户端保持状态的方案，而session机制采用的是在服务器端保持状态的方案。同时我们也看到，由于采用服务器端保持状态的方案在客户端也需要保存一个标识，所以session机制可能需要借助于cookie机制来达到保存标识的目的，但实际上它还有其他选择。
# 三、Cookie
1. Cookie的发送：正统的Cookie是通过拓展HTTP协议来实现的，服务器通过在HTTP的响应头中加上一行特殊的指示以提示浏览器按照指示生成相应的cookie。然而纯粹的客户端脚本如JavaScript或者VBScript也可以生成cookie。
```java
Cookie cookie = new Cookie(key,value);　　//以键值对的方式存放内容，

　　　　　　　　　　response.addCookie(cookie);　　//发送回浏览器端

　　注意：一旦cookie创建好了，就不能在往其中增加别的键值对，但是可以修改其中的内容，

　　　　　　　　　　cookie.setValue();　　//将key对应的value值修改
```
2. Cookie的使用：浏览器按照一定的原则发送给服务器。浏览器检查所有Cookie，如果cookie的范围大于所要访问资源的范围，则把cookie加入到请求头中访问服务器。相当于一个品牌的会员卡，可以在所有连锁店中使用。
![image.png](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture/202307060904609.png)

3. cookie的内容主要包括：名字，值，过期时间，路径和域。
	其中域可以指定某一个域比如.google.com，相当于总店招牌，比如宝洁公司，也可以指定一个域下的具体某台机器比如www.google.com或者froogle.google.com，可以用飘柔来做比。  
	路径就是跟在域名后面的URL路径，比如/或者/foo等等，可以用某飘柔专柜做比。  
	路径与域合在一起就构成了cookie的作用范围。  
	如果不设置过期时间，则表示这个cookie的生命期为浏览器会话期间，只要关闭浏览器窗口，cookie就消失了。这种生命期为浏览器会话期的cookie被称为会话cookie。会话cookie一般不存储在硬盘上而是保存在内存里，当然这种行为并不是规范规定的。如果设置了过期时间，浏览器就会把cookie保存到硬盘上，关闭后再次打开浏览器，这些cookie仍然有效直到超过设定的过期时间。存储在硬盘上的cookie可以在不同的浏览器进程间共享.
	下面就是一个goolge设置cookie的响应头的例子
	```java
curl --location --request GET 'http://localhost:8080/question/6' \

--header 'Cookie: cookie_key1=cookie_value1;cookie_key2=cookie_value2' \

--header 'token: eyJhbGciOiJIUzUxMiJ9.eyJ1c2VySWQiOjExMTExLCJyb2xlIjoiMSIsImV4cCI6MTY4ODg5MDc1OX0.sTAQjtxauCkT4dgQ967lfApgDnFgS2wjmwUC667s6mmp2quIlUGRc2759GRG4IqtlfuU_orRnAoc4VVTyohNWQ' \

--header 'User-Agent: Apifox/1.0.0 (https://www.apifox.cn)' \

--header 'Accept: */*' \

--header 'Host: localhost:8080' \

--header 'Connection: keep-alive'
    ```
# 四、Session
1.session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。
浏览器请求web服务器时，当程序需要创建Sesson时，服务器会先检查这个请求是否包含一个Session标识，称为SessionId，如果包含，则说明用户创建过Session，服务器会根据SessionId检索出来使用（这个SessionId应该是不重复的且不易被找到规律以仿照的字符串）；如果没有，服务器则创建一个Session，并创建一个与之相联的SessionId，在响应的时候返回给客户端保存，保存的方式可以是cookie。在交互过程中，客户端可以把这个SessionId发给服务器，服务器再根据这个SessionId获取Session。
```java
//获取Session
request.getSession();　　//如果没有将创建一个新的，等效getSession(true);
```
- 需要注意的是，`getSession()`方法是定义在`HttpServletRequest`接口中的抽象方法，request.getSession()的实现是由Servlet容器（tomcat，jetty）提供的，容器会根据规范实现会话管理，并在需要时创建和管理会话对象。具体看[[基础/tomcat/tomcat架构|tomcat架构]]
2.session的生命周期：只要浏览器不发送指令去删除session，session会一直存在。关闭浏览器后可能会删除内存中的cookie，导致获取不到SessionId，后来访问后重新登录，这样看起来会**像**是关闭浏览器就删除了session。正是因为浏览器不会让服务器删除session，迫使session设置了一个失效时间。当超过失效时间，会默认用户退出登录，后删除session（先持久化）。
- 创建：执行request.getSession();
- 销毁：
	1. 超时，默认三十分钟
	2. 执行api：session.invalidate()将session对象销毁、setMaxInactiveInterval(int interval) 设置有效时间，单位秒
	3. 服务器非正常关闭。（非正常关闭不会进行持久化）
3.sessionId的url重写：当Cookie被浏览器禁用时，将不能通过Cookie发送SessionId。可通过url重写发送SessionId

# 参考：
[会话机制，Cookie和Session详解 博客园 (cnblogs.com)](https://www.cnblogs.com/whgk/p/6422391.html)
[session CSDN博客](https://blog.csdn.net/keda8997110/article/details/16922815)
