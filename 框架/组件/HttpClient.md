# 在Springboot中使用HttpClient实现HTTP请求



HttpClient是Apache J阿卡Common 下的子项目，用来提供搞笑的、最新的、功能丰富的支持HTTP协议的客户端编程工具包，并且它支持Http协议最新的版本和建议

[HttpClient官网]([Apache HttpComponents – HttpClient Overview](https://hc.apache.org/httpcomponents-client-5.2.x/index.html))

> HttpClientpom依赖

```xml
<dependency>
	<groupId>org.apache.httpcomponents</groupId>
	<artifactId>httpclient</artifactId>
	<version>4.5.6</version>
</dependency>
```

### 1. 获取客户端

![image-20230811195758496](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308111957583.png)

在使用时，首先需要 获取 HttpClient 对象，有如下几种获取方式

1. **默认创建方式**

   ```java
   // 获取默认配置的 HttpClient 
   CloseableHttpClient httpClient = HttpClients.createDefault();
   ```

2. **根据系统配置创建**

   ```java
   // 此种方式是通过 根据 系统配置创建 HttpClient
   CloseableHttpClient httpClient = HttpClients.createSystem();
   ```

   在项目启动时可以通过设置如下JVM启动参数：

   - http.agent  配置 userAgent
   - http.keepAlive  配置 keepAlive 数据

3. **自定义创建**

   ```java
   // 此种方式可以在创建时 设置一些默认值
   CloseableHttpClient  httpClient = HttpClients.custom()
                        .setDefaultHeaders(Collections.emptyList())   // 设置默认请求头
                        .setDefaultRequestConfig(RequestConfig.DEFAULT)  // 设置默认配置
                        .build();
   ```

### 2. 配置参数

HttpClient可以通过创建HttpClient对象时就设置全局配置，也可以为单个请求设置请求配置

1. 创建配置对象

   ```java
     //  创建请求配置信息
   	RequestConfig  requestConfig = RequestConfig.custom()
        // 设置连接超时时间
       .setConnectTimeout(Timeout.of(3000, TimeUnit.MILLISECONDS))
       // 设置响应超时时间
       .setResponseTimeout(3000, TimeUnit.MILLISECONDS) 
       // 设置从连接池获取链接的超时时间
       .setConnectionRequestTimeout(3000, TimeUnit.MILLISECONDS)
       .build();
   
   
   
   ```

2. 设置全局配置

   ```java
   // 此种方式可以在创建时 设置一些默认值
   CloseableHttpClient  httpClient = HttpClients.custom()
                        .setDefaultHeaders(Collections.emptyList())   // 设置默认请求头
                        .setDefaultRequestConfig(requestConfig)  // 设置默认配置
                        .build();
   
   ```

3. 单个请求设置配置

   ```java
   // 创建 GET 请求对象
   HttpGet httpGet = new HttpGet(uri);
   
   // 设置请求参数
   httpGet.setConfig(requestConfig);
   
   ```

### 3. 设置请求头信息

在请求时，会遇到设置自定义请求头或者更改Conent-Type的值，可以通过一下两种方式设置：

1. 设置公共请求头

   ```java
   List<Header> headers = new ArrayList<>();
   headers.add(new BasicHeader(HttpHeaders.CONTENT_TYPE, ContentType.APPLICATION_JSON));
   headers.add(new BasicHeader(HttpHeaders.ACCEPT_ENCODING, "gzip, x-gzip, deflate"));
   headers.add(new BasicHeader(HttpHeaders.CONNECTION, "keep-alive"));
   
   // 创建 一个默认的 httpClient
   CloseableHttpClient  httpClient = HttpClients.custom()
       .setDefaultHeaders(headers)   // 设置默认请求头
       .build()
   
   ```

2. 单个请求设置请求头

   ```java
   // 创建 POST 请求
   HttpPost httpPost = new HttpPost(uri);
   // 添加 Content-Type 请求头
   httpPost.addHeader(HttpHeaders.CONTENT_TYPE, ContentType.APPLICATION_FORM_URLENCODED);
   // 添加 accept 请求头
   httpPost.addHeader(new BasicHeader(HttpHeaders.ACCEPT, "*/*"));
   ```

###  4.发送请求

#### GET请求

GET请求的所有参数都是拼接在URL后面的，在HttpClient中，有两种方式实现：

1. 请求参数直接拼接在请求路径后面

   ```JAVA
           //设置url
           String url="https://p3-passport.byteacctimg.com/img/user-avatar/6aed7928847e099390ad80f4e57688a8~100x100.awebp";
           //创建GET请求对象
           HttpGet httpGet = new HttpGet(url);
   //        创建HttpClient对象
           HttpClient httpClient= HttpClients.createDefault();
   //        使用HttpClient执行GET请求，得到response
           HttpResponse response = httpClient.execute(httpGet);
   ```

2. 通过URIBuilder构建请求路径

```java

// 构建请求路径，及参数
URL url = new URL("http://localhost:10010/user/params");
URI uri = new URIBuilder()
    .setScheme(url.getProtocol())
    .setHost(url.getHost())
    .setPort(url.getPort())
    .setPath(url.getPath())
    // 构建参数
    .setParameters(
        new BasicNameValuePair("name", "张三"),
        new BasicNameValuePair("age", "20")
     ).build();

// 创建 GET 请求对象
HttpGet httpGet = new HttpGet(uri);
// 调用 HttpClient 的 execute 方法执行请求
CloseableHttpResponse response = httpClient.execute(httpGet);
// 获取请求状态
int code = response.getCode();
// 如果请求成功
if(code == HttpStatus.SC_OK){
    LOGGER.info("响应结果为：{}", EntityUtils.toString(response.getEntity()));
}
```

#### POST请求

HTTP中POST请求的数据是包含在请求体中。在HttpClient中POST请求发送是通过调用HttpPost类中的SetEntity（HttpEntity entity）方法设置消息内容。

1. 发送JSON数据

   HttpClient 中发送 JSON 数据可以使用 StringHttpEntity 类实现，如下所示：

   ```java
   // 请求参数
   String url = "http://localhost:10010/user/body";
   // 创建 GET 请求对象
   HttpPost httpPost = new HttpPost(url);
   // 构建对象
   User user = new User();
   user.setName("张三")
       .setAge(20)
       .setAddress(new Address()
                   .setCounty("中国")
                   .setCity("北京"))
       .setAihao(Arrays.asList("跑步", "爬山", "看书"));
   
   // 创建 字符串实体对象
   HttpEntity httpEntity = new StringEntity(JSON.toJSONString(user));
   httpPost.setEntity(httpEntity);
   
   // 发送 POST 请求
   httpClient.execute(httpPost);
   ```

2. 模拟form表单数据

   在实际使用中，可以存在 需要模拟form表单的情况进行请求数据，在HttpClient中也可以很方便的实现form的提交功能

   - 修改contentType

     ```java
     // 创建 ContentType 对象为 form 表单模式 
     ContentType contentType = ContentType.create("application/x-www-form-urlencoded", StandardCharsets.UTF_8);
     // 添加到 HttpPost 头中
     httpPost.setHeader(HttpHeaders.CONTENT_TYPE, contentType);
     ```

   - 创建请求数据HttpEntity

     ```java
     // 方式一、自己拼接请求数据，并且创建 StringEntity 对象
     String query = "name="+ URLEncoder.encode("张三", "utf-8") +"&age=20";
     HttpEntity httpEntity = new StringEntity(query);
     
     // 方式二、通过UrlEncodedFormEntity 创建 HttpEntity
     HttpEntity httpEntity = new UrlEncodedFormEntity(
         Arrays.asList(new BasicNameValuePair("name", "张三"),
                       new BasicNameValuePair("age", "20")),
         StandardCharsets.UTF_8
     );
     
     // 把 HttpEntity 设置到 HttpPost 中
     httpPost.setEntity(httpEntity);
     ```

### 5.上传下载

#### 上传

在HttpClient中实现上传功能，可以通过`MultipartEntityBuilder`直接构建HttpClient即可

```java
//要上传的文件
File file = new File("F:/20150703212056_Yxi4L.jpeg");

// 创建对象
MultipartEntityBuilder builder = MultipartEntityBuilder.create();

// 添加二进制消息体
builder.addBinaryBody("file", file);

// 也可以添加文本消息
ContentType contentType = ContentType.TEXT_PLAIN.withCharset(StandardCharsets.UTF_8);
builder.addTextBody("name", "张三", contentType);

// 通过 MultipartEntityBuilder 构建消息体
HttpEntity httpEntity = builder.build();
HttpPost httpPost = new HttpPost("http://localhost:10010/user/upload");
httpPost.setEntity(httpEntity);
CloseableHttpResponse response = httpClient.execute(httpPost);
// 获取请求状态
int code = response.getCode();
// 如果请求成功
if(code == HttpStatus.SC_OK){
    LOGGER.info("响应结果为：{}", EntityUtils.toString(response.getEntity()));
}
```

#### 下载

HttpClient中的下载只需发送请求，读取输入流，然后保存到相应的路径下即可

```java
// 请求下载路径
HttpGet httpGet = new HttpGet("http://localhost:10010/user/downLoad");
CloseableHttpResponse response = httpClient.execute(httpGet);

// 如果请求成功
if (response.getCode() == HttpStatus.SC_OK){

    // 获取下载文件的文件名，此处的 File-Name 头信息，需要在服务端进行自定义
    Header header = response.getFirstHeader("File-Name");
    String value = header.getValue();

    // 读取数据
    byte[] bytes = EntityUtils.toByteArray(response.getEntity());
    try (OutputStream outputStream = new FileOutputStream("F:/" + value);){
        outputStream.write(bytes);
        outputStream.flush();
    }
}
```

### 6.响应处理

在HttpClient中把响应封装成`CloseableHttpResponse`对像，

此对象有如下接口：![image-20230811205246037](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308112052112.png)

HttpClient提供了`EntityUtils`工具类，可以把HttpEntity转换为字节数组或字符串![image-20230811205411540](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202308112054605.png)

此外可以在调用HttpClient的`execute（）`方法时传入`响应处理器`,返回自定义的数据类型。

```java
       @Data
        @Accessors(chain =true)
        public class Response {
            private int code;
            private String msg;
            private String body;
        }


	HttpResponse response = httpClient.execute(httpGet,res->{
            return new Response().setCode(res.getCode())
                    .setMsg(res.getReasonPhrase())
                    .setBody(EntityDetails.toString(res.getEntity(), StandardCharsets.UTF_8));
        });
```

### 7.会话保持

可能会有需要保持会话的情况（需要登录后才能进行先关的操作），HttpClient提供了`HttpClientContext`,可以实现会话保持功能

```java
// 创建 HttpClientContext对象
HttpContext httpContext = new BasicHttpContext();
httpContext.setAttribute("name", "zhangsan");
HttpClientContext httpClientContext = HttpClientContext.adapt(httpContext);

// 登录
httpClient.execute(new HttpPost(""), httpClientContext);

// 获取数据
httpClient.execute(new HttpGet(""), httpClientContext);

```

## 总结

在使用上，主要可以把HttpClient对象当做用户这个对象，来通过HttpGet、HttpPost、HttpClientContext来实现相应的功能，而HttpGet则亏当做`请求`这个对象，通过设置HttpGet等来完善相应的需求，通过多态来提高接口通用性。



参考博客：

[Apache HttpClient 5 使用详细教程 - 掘金 (juejin.cn)](https://juejin.cn/post/7134500832979976205#heading-16)

[Apache HttpClient 详解 - 掘金 (juejin.cn)](https://juejin.cn/post/7052900785381703694#heading-11)