# JSON

## JSON数据和JavaJava对象的转换

- Fastjson是阿里巴巴提供的一个Java语言编写的高性能功能完善的JSON库，是目前Java语言中最快的JSON库，可以实现Java对象和JSON字符串的相互转换。

- 使用：

    1.  导入坐标

    ```xml
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.62</version>
    </dependency>
    ```

    2. Java对象转JSON

        ```java
         String jsonStr= JSON.toJSONString(obj);
        ```

    3. Json字符串转Java

        ```java
        User user= JSON.parseObject(str,User.class);
        ```

        