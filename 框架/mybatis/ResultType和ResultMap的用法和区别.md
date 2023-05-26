---
alias:
tags: mybatis
from: https://blog.csdn.net/xushiyu1996818/article/details/89075069
---
# 综述

MyBatis中在查询进行select映射的时候，返回类型可以用resultType，也可以用resultMap，resultType是直接表示返回类型的，而resultMap则是对外部ResultMap的引用，但是resultType跟resultMap不能同时存在。
在MyBatis进行查询映射时，其实查询出来的每一个属性都是放在一个对应的Map里面的，其中键是属性名，值则是其对应的值。
①当提供的返回类型属性是resultType时，MyBatis会将Map里面的键值对取出赋给resultType所指定的对象对应的属性。所以其实MyBatis的每一个查询映射的返回类型都是ResultMap，只是当提供的返回类型属性是resultType的时候，MyBatis对自动的给把对应的值赋给resultType所指定对象的属性。
②当提供的返回类型是resultMap时，因为Map不能很好表示领域模型，就需要自己再进一步的把它转化为对应的对象，这常常在复杂查询中很有作用。

# ResultType

resultType可以直接返回给出的返回值类型，比如String、int、Map，等等，其中返回List也是将返回类型定义为Map，然后mybatis会自动将这些map放在一个List中，resultType还可以是一个对象，举例如下：

## 返回常见类型：（类似于int或者Integer）

```XML
　　<select id="getLogCount" resultType="int">
　　　　select COUNT(*) from AttLog where attTime = #{attTime} and userId = #{userId};
　　</select>

```

## 返回Map

```xml
<select id="getDeviceInfoByDeviceId" resultType="Map">
　　select userCount as usercount,
　　fingerCount as fingercount,
　　faceCount as facecount,
　　attRecordCount as recordcount,
　　lastOnline,
　　state as status
　　from DeviceInfo where deviceId = #{deviceId} and tb_isDelete = 0;
</select>
```

返回一个对象或者一个list（list里面是resultType的类型）

```sql
<select id="queryAllDeviceInfo" resultType="com.cachee.ilabor.att.clientmodel.DeviceInfo">
select * from deviceInfo where tb_isDelete = 0;
</select>
```

## **返回一个对象**

**对于SQL语句查询出的字段在相应的pojo中必须有和它相同的字段对应。**

但是 ，如果列名没有精确匹配,你可以在列名上使用 select 字句的别名(一个 基本的 SQL 特性)来匹配标签.

```xml
<select id="selectUsers" parameterType="int" resultType="User">
  select
    user_id             as "id",
    user_name           as "userName",
    hashed_password     as "hashedPassword"
  from some_table
  where id = #{id}
</select>
```

resultType对应的是java对象中的属性，大小写不敏感，
resultType如果放的是java.lang.Map,key是查询语句的列名，value是查询的值，大小写敏感
resultMap:指的是定义好了的id的，是定义好的resyltType的引用
注意：用resultType的时候，要保证结果集的列名与java对象的属性相同，而resultMap则不用，而且resultMap可以用typeHander转换
parameterType:参数类型，只能传一个参数，如果有多个参数要封装，如封装成一个类，要写包名加类名，基本数据类型则可以省略
一对1、一对多时，若有表的字段相同必须写别名，不然查询结果无法正常映射，出现某属性为空或者返回的结果与想象中的不同，而这往往是没有报错的。

# ResultMap

## 基本使用

适合使用返回值是自定义实体类的情况

映射实体类的数据类型

id:resultMap的唯一标识

column: 库表的字段名

property： 实体类里的属性名

配置映射文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace:当前库表映射文件的命名空间，唯一的不能重复 -->
<mapper namespace="com.hao947.sql.mapper.PersonMapper">
  <!-- type:映射实体类的数据类型 id：resultMap的唯一标识 -->
  <resultMap type="person" id="BaseResultMap">
    <!-- column:库表的字段名 property:实体类里的属性名 -->
    <id column="person_id" property="personId" />
    <result column="name" property="name" />
    <result column="gender" property="gender" />
    <result column="person_addr" property="personAddr" />
    <result column="birthday" property="birthday" />
  </resultMap>
  <!--id:当前sql的唯一标识
     parameterType：输入参数的数据类型 
     resultType：返回值的数据类型 
     #{}:用来接受参数的，如果是传递一个参数#{id}内容任意，如果是多个参数就有一定的规则,采用的是预编译的形式select 
    * from person p where p.id = ? ，安全性很高 -->
 
  <!-- sql语句返回值类型使用resultMap -->
  <select id="selectPersonById" parameterType="java.lang.Integer"
    resultMap="BaseResultMap">
    select * from person p where p.person_id = #{id}
  </select>
  <!-- resultMap:适合使用返回值是自定义实体类的情况 
  resultType：适合使用返回值的数据类型是非自定义的，即jdk的提供的类型 -->
  <select id="selectPersonCount" resultType="java.lang.Integer">
    select count(*) from
    person
  </select>
 
  <select id="selectPersonByIdWithMap" parameterType="java.lang.Integer"
    resultType="java.util.Map">
    select * from person p where p.person_id= #{id}
    </select>
 
</mapper>
```

## id和result
```
<id property="id" column="post_id"/>

<result property="subject" column="post_subject"/>
```

这是最基本的结果集映射。*id* 和*result* 将列映射到属性或简单的数据类型字段(String, int, double, Date等)。

这两者唯一不同的是，在比较对象实例时**id 作为结果集的标识属性**。这有助于提高总体性能，**特别是应用缓存和嵌套结果映射的时候。**

Id、result属性如下：
![](https://back.tasdasdasdas.tk/202212111958380.png)


## 高级使用

MyBatis的创建基于这样一个思想：数据库并不是您想怎样就怎样的。虽然我们希望所有的数据库遵守第三范式或BCNF（修正的第三范式），但它们不是。如果有一个数据库能够完美映射到所有应用程序，也将是非常棒的，但也没有。结果集映射就是MyBatis为解决这些问题而提供的解决方案。

示例

```xml
<!-- 超复杂的 Result Map -->
<resultMap id="detailedBlogResultMap" type="Blog">
  <constructor>
    <idArg column="blog_id" javaType="int"/>
  </constructor>
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
    <result property="favouriteSection" column="author_favourite_section"/>
  </association>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <association property="author" javaType="Author"/>
    <collection property="comments" ofType="Comment">
      <id property="id" column="comment_id"/>
    </collection>
    <collection property="tags" ofType="Tag" >
      <id property="id" column="tag_id"/>
    </collection>
    <discriminator javaType="int" column="draft">
      <case value="1" resultType="DraftPost"/>
    </discriminator>
  </collection>
</resultMap>
```

**resultMap**

**constructor**–实例化的时候通过构造器将结果集注入到类中

**idArg,arg**–注入构造器的结果集

**id**–结果集ID，将结果集标记为ID，以方便全局调用

**result**–注入一个字段或者javabean属性的结果

**association**–复杂类型联合;许多查询结果合成这个类型

*嵌套结果映射*– associations能引用自身,或者从其它地方引用,

**collection**–复杂类型集合

*嵌套结果映射*– collections能引用自身,或者从其它地方引用

**discriminator**–使用一个结果值以决定使用哪个resultMap

**case**–基于不同值的结果映射

*嵌套结果映射*–case也能引用它自身, 所以也能包含这些同样的元素。它也可以从外部引用resultMap

注意：

```html
public class A{
    private B b1;
    private List<B> b2;
}
在映射b1属性时用association标签, 映射b2时用collection标签，分别是一对一，一对多的关系
```

## Constructor元素

**通常情况下， java实体类的属性都有get和set方法，但是在有的不变类中，没有get和set方法，只能在构造器中注入属性，这个时候就要constructor元素**

```delphi
<constructor>
<idArg column="id" javaType="int"/>
<arg column=”username” javaType=”String”/>
</constructor>
```

当属性与DTO，或者与您自己的域模型一起工作的时候，许多场合要用到不变类。通常，包含引用，或者查找的数据很少或者数据不会改变的的表，适合映射到不变类中。构造器注入允许您在类实例化后给类设值，这不需要通过public方法。MyBatis同样也支持private属性和JavaBeans的私有属性达到这一点，但是一些用户可能更喜欢使用构造器注入。构造器元素可以做到这点。

```XML
public class User {
   //...
   public User(Integer id, String username, int age) {
     //...
  }
//...
}
```

为了将结果注入构造方法，MyBatis需要通过某种方式定位相应的构造方法。
在下面的例子中，MyBatis搜索一个声明了三个形参的的构造方法，以 java.lang.Integer, java.lang.String and int 的顺序排列。

```xml
<constructor>
   <idArg column="id" javaType="int"/>
   <arg column="username" javaType="String"/>
   <arg column="age" javaType="_int"/>
</constructor>
```

当你在处理一个带有多个形参的构造方法时，很容易在保证 arg 元素的正确顺序上出错。 从版本 3.4.3 开始，可以在指定参数名称的前提下，以任意顺序编写 arg 元素。 **为了通过名称来引用构造方法参数，你可以添加 @Param 注解，或者使用 ‘-parameters’ 编译选项并启用 useActualParamName 选项（默认开启）来编译项目。** 下面的例子对于同一个构造方法依然是有效的，尽管第二和第三个形参顺序与构造方法中声明的顺序不匹配。

```xml
<constructor>
   <idArg column="id" javaType="int" name="id" />
   <arg column="age" javaType="_int" name="age" />
   <arg column="username" javaType="String" name="username" />
</constructor>
```

如果类中存在名称和类型相同的属性，那么可以省略 javaType 。
剩余的属性和规则和普通的 id 和 result 元素是一样的。

constructor - 用于在实例化类时，注入结果到构造方法中
idArg - ID 参数;标记出作为 ID 的结果可以帮助提高整体性能
arg - 将被注入到构造方法的一个普通结果

```xml
<constructor>
<idArg column="id" javaType="int"/>
<arg column=”username” javaType=”String”/>
</constructor>
```

其它的属性与规则与id、result元素的一样。

![](https://back.tasdasdasdas.tk/202212112000797.png)


## Association

关联

```xml
<association property="author" column="blog_author_id" javaType="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
</association>
```

**关联元素处理“有一个”类型的关系**。比如,在我们的示例中,一个博客有一个用户。
关联映射就工作于这种结果之上。你指定了目标属性,来获取值的列,属性的 java 类型(很多情况下 MyBatis 可以自己算出来),如果需要的话还有 jdbc 类型,如果你想覆盖或获取的结果值还需要类型控制器。
关联中不同的是你需要告诉 MyBatis 如何加载关联。MyBatis 在这方面会有两种不同的
方式:

·**Nested Select:**通过执行另一个返回预期复杂类型的映射SQL语句（即引用外部定义好的SQL语句块）。

·**Nested Results:**通过嵌套结果映射（nested result mappings）来处理联接结果集（joined results）的重复子集。

它不同于普通只有select和resultMap属性的结果映射。

![](https://back.tasdasdasdas.tk/202212112002765.png)


联合嵌套选择（Nested Select for Association）
select
通过这个属性，通过ID引用另一个加载复杂类型的映射语句。从指定列属性中返回的值，将作为参数设置给目标select 语句。表格下方将有一个例子。注意：在处理组合键时，您可以使用column=”{prop1=col1,prop2=col2}”这样的语法，设置多个列名传入到嵌套语句。这就会把prop1和prop2设置到目标嵌套语句的参数对象中。


```xml
<resultMap id=”blogResult” type=”Blog”>  
<association property="author" column="blog_author_id" javaType="Author"  
select=”selectAuthor”/>  
</resultMap>  
   
<select id=”selectBlog” parameterType=”int” resultMap=”blogResult”>  
SELECT * FROM BLOG WHERE ID = #{id}  
</select>  
   
<select id=”selectAuthor” parameterType=”int” resultType="Author">  
SELECT * FROM AUTHOR WHERE ID = #{id}  
</select>  
<wbr>  
```

我们使用两个select语句：一个用来加载Blog，另一个用来加载Author。Blog的resultMap 描述了使用“selectAuthor”语句来加载author的属性。

如果列名和属性名称相匹配的话，所有匹配的属性都会自动加载。

上面的例子，首先执行`<select id=“selectBlog”>`，执行结果存放到`<resultMap id=“blogResult”>`结果映射中。“blogResult”是一个Blog类型，从`<select id=“selectBlog”>`查出的数据都会自动赋值给”blogResult”的与列名匹配的属性，这时blog_id，title等就被赋值了。同时“blogResult”还有一个关联属性"Author"，执行嵌套查询select=”selectAuthor”后，Author对象的属性id，username，password，email，bio也被赋于数据库匹配的值。

```txt
Blog{
 
blog_id;
 
title;
 
Author author{
	 
	id;
	 
	username;
	 
	password;
	 
	email;
	 
	bio;
	}
 
}
```

虽然这个方法简单，但是对于大数据集或列表查询，就不尽如人意了。**这个问题被称为“N+1 选择问题”（N+1 Selects Problem）。概括地说，N+1选择问题是这样产生的：**

**您执行单条SQL语句去获取一个列表的记录( “+1”)。**

**对列表中的每一条记录，再执行一个联合select 语句来加载每条记录更加详细的信息(“N”)。**

**这个问题会导致成千上万的SQL语句的执行，因此并非总是可取的。**

上面的例子，MyBatis可以使用延迟加载这些查询，因此这些查询立马可节省开销。然而，如果您加载一个列表后立即迭代访问嵌套的数据，这将会调用所有的延迟加载，因此性能会变得非常糟糕。

鉴于此，这有另外一种方式。

联合嵌套结果集（Nested Results for Association）
resultMap
一个可以映射联合嵌套结果集到一个适合的对象视图上的ResultMap 。这是一个替代的方式去调用另一个select 语句。它允许您去联合多个表到一个结果集里。这样的结果集可能包括冗余的、重复的需要分解和正确映射到一个嵌套对象视图的数据组。简言之，MyBatis 让您把结果映射‘链接’到一起，用来处理嵌套结果。举个例子会更好理解，例子在表格下方。

您已经在上面看到了一个非常复杂的嵌套联合的例子，接下的演示的例子会更简单一些。我们把Blog和Author表联接起来查询，而不是执行分开的查询语句：

```xml
<select id="selectBlog" parameterType="int" resultMap="blogResult">  
select  
B.id as blog_id,  
B.title as blog_title,  
B.author_id as blog_author_id,  
A.id as author_id,  
A.username as author_username,  
A.password as author_password,  
A.email as author_email,  
A.bio as author_bio  
from Blog B left outer join Author A on B.author_id = A.id  
where B.id = #{id}  
</select>  
```

注意到这个连接（join），**要确保所有的别名都是唯一且无歧义的**。（**为什么用别名呢？用别名的目的就是确保唯一无歧义。今天写了一段代码，没有返回List类型的结果集，检查了很久也没报错，最后加上别名，唯一区别开来就好了。估计是两个表中有个别字段歧义**）这使映射容易多了，现在我们来映射结果集：

```xml
<resultMap id="blogResult" type="Blog">  
<id property=”blog_id” column="id" />  
<result property="title" column="blog_title"/>  
<association property="author" column="blog_author_id" javaType="Author"  
resultMap=”authorResult”/>  
</resultMap>  
   
<resultMap id="authorResult" type="Author">  
<id property="id" column="author_id"/>  
<result property="username" column="author_username"/>  
<result property="password" column="author_password"/>  
<result property="email" column="author_email"/>  
<result property="bio" column="author_bio"/>  
</resultMap>  
```

在上面的例子中，您会看到Blog的作者（“author”）联合一个“authorResult”结果映射来加载Author实例。

**重点提示:*id*元素在嵌套结果映射中扮演了非常重要的角色，您应该总是指定一个或多个属性来唯一标识这个结果集**。事实上，如果您没有那样做，MyBatis也会工作，但是会导致严重性能开销。选择尽量少的属性来唯一标识结果，而使用主键是最明显的选择（即使是复合主键）。

上面的例子使用一个扩展的resultMap 元素来联合映射。这可使Author结果映射可重复使用。然后，**如果您不需要重用它，您可以直接嵌套这个联合结果映**射。下面例子就是使用这样的方式：

```xml
<resultMap id="blogResult" type="Blog">  
<id property=”blog_id” column="id" />  
<result property="title" column="blog_title"/>  
<association property="author" column="blog_author_id" javaType="Author">  
<id property="id" column="author_id"/>  
<result property="username" column="author_username"/>  
<result property="password" column="author_password"/>  
<result property="email" column="author_email"/>  
<result property="bio" column="author_bio"/>  
</association>  
</resultMap> 
```

在上面的例子中您已经看到如果处理“一对一”（“has one”）类型的联合查询。

## Collection

```xml
<collection property="posts" ofType="domain.blog.Post">  
<id property="id" column="post_id"/>  
<result property="subject" column="post_subject"/>  
<result property="body" column="post_body"/>  
</collection> 
```

*collection*元素的作用差不多和association元素的作用一样。事实上，它们非常相似，以至于再对相似点进行描述会显得冗余，因此我们只关注它们的不同点。

继续我们上面的例子，一个Blog只有一个Author。但一个Blog有许多帖子（文章）。在Blog类中，会像下面这样定义相应属性：

`private List<Post> posts`;

映射一个嵌套结果集到一个列表，我们使用*collection*元素。就像*association* 元素那样，我们使用嵌套查询，或者从连接中嵌套结果集。

**集合嵌套选择（Nested Select for Collection）**

首先我们使用嵌套选择来加载Blog的文章。

```xml
<resultMap id=”blogResult” type=”Blog”>  
<collection property="posts" javaType=”ArrayList” column="blog_id"  
ofType="Post" select=”selectPostsForBlog”/>  
</resultMap>  
   
<select id=”selectBlog” parameterType=”int” resultMap=”blogResult”>  
SELECT * FROM BLOG WHERE ID = #{id}  
</select>  
   
<select id=”selectPostsForBlog” parameterType=”int” resultType="Author">  
SELECT * FROM POST WHERE BLOG_ID = #{id}  
</select>  
```

一看上去这有许多东西需要注意，但大部分看起与我们在*associa*tion元素中学过的相似。首先，您会注意到我们使用了collection元素，然后会注意到一个新的属性“ofType”。这个元素是用来区别JavaBean属性（或者字段）类型和集合所包括的类型。因此您会读到下面这段代码。

```xml
<collection property="posts" javaType=”ArrayList” column="blog_id"
 
ofType="Post" select=”selectPostsForBlog”/>
```

理解为:“一个名为posts，类型为Post的ArrayList集合（A collection of posts in an ArrayList of type Post）” 。

javaType属性不是必须的，通常MyBatis 会自动识别，所以您通常可以简略地写成：

```xml
<collection property="posts" column="blog_id" ofType="Post"
 
select=”selectPostsForBlog”/>
```

**集合的嵌套结果集（Nested Results for Collection）**

这时候，您可能已经猜出嵌套结果集是怎样工作的了，因为它与association非常相似，只不过多了一个属性***“ofType”****。*

```csharp
<select id="selectBlog" parameterType="int" resultMap="blogResult">  
select  
B.id as blog_id,  
B.title as blog_title,  
B.author_id as blog_author_id,  
P.id as post_id,  
P.subject as post_subject,  
P.body as post_body,  
from Blog B  
left outer join Post P on B.id = P.blog_id  
where B.id = #{id}  
</select>
```

同样，我们把Blog和Post两张表连接在一起，并**且也保证列标签名在映射的时候是唯一且无歧义的**。现在将Blog和Post的集合映射在一起是多么简单：

```xml
<resultMap id="blogResult" type="Blog">  
<id property=”id” column="blog_id" />  
<result property="title" column="blog_title"/>  
<collection property="posts" ofType="Post">  
<id property="id" column="post_id"/>  
<result property="subject" column="post_subject"/>  
<result property="body" column="post_body"/>  
</collection>  
</resultMap>  
```

再次强调一下，*id* 元素是非常重要的。如果您忘了或者不知道*id* 元素的作用，请先读一下上面*association*一节。

如果希望结果映射有更好的可重用性，您可以使用下面的方式：

```xml
<resultMap id="blogResult" type="Blog">  
<id property=”id” column="blog_id" />  
<result property="title" column="blog_title"/>  
<collection property="posts" ofType="Post" resultMap=”blogPostResult”/>  
</resultMap>  
   
<resultMap id="blogPostResult" type="Post">  
<id property="id" column="post_id"/>  
<result property="subject" column="post_subject"/>  
<result property="body" column="post_body"/>  
</resultMap>  
```

在您的映射中没有深度、宽度、联合和集合数目的限制。但应该谨记，在进行映射的时候也要考虑性能的因素。应用程序的单元测试和性能测试帮助您发现最好的方式可能要花很长时间。但幸运的是，MyBatis允许您以后可以修改您的想法，这时只需要修改少量代码就行了。

## discriminator

```xml
<discriminator javaType="int" column="draft">
  <case value="1" resultType="DraftPost"/>
</discriminator>
```

**有时一个单独的数据库查询也许返回很多不同(但是希望有些关联)数据类型的结果集。
鉴别器元素就是被设计来处理这个情况的,还有包括类的继承层次结构。鉴别器非常容易理解,因为它的表现很像 Java 语言中的 switch 语句。 **
定义鉴别器指定了 column 和 javaType 属性。
（1）column 是 MyBatis 查找比较值的地方。
（2）JavaType 是需要被用来保证等价测试的合适类型(尽管字符串在很多情形下都会有用)。比如:

```sql
<resultMap id="vehicleResult" type="Vehicle">
  <id property="id" column="id" />
  <result property="vin" column="vin"/>
  <result property="year" column="year"/>
  <result property="make" column="make"/>
  <result property="model" column="model"/>
  <result property="color" column="color"/>
  <discriminator javaType="int" column="vehicle_type">
    <case value="1" resultMap="carResult"/>
    <case value="2" resultMap="truckResult"/>
    <case value="3" resultMap="vanResult"/>
    <case value="4" resultMap="suvResult"/>
  </discriminator>
</resultMap>
```

在这个示例中,MyBatis 会从结果集中得到每条记录,然后比较它的 vehicle 类型的值。
如果它匹配任何一个鉴别器的实例,那么就使用这个实例指定的结果映射。换句话说,这样做完全是剩余的结果映射被忽略(除非它被扩展,这在第二个示例中讨论)。如果没有任何一个实例相匹配,那么 MyBatis 仅仅使用鉴别器块外定义的结果映射。所以,如果 carResult按如下声明:

```xml
<resultMap id="carResult" type="Car">
  <result property="doorCount" column="door_count" />
</resultMap>
```

那么只有 doorCount 属性会被加载。这步完成后完整地允许鉴别器实例的独立组,尽管
和父结果映射可能没有什么关系。这种情况下,我们当然知道 cars 和 vehicles 之间有关系,如 Car 是一个 Vehicle 实例。因此,我们想要剩余的属性也被加载。我们设置的结果映射的简单改变如下。

```xml
<resultMap id="carResult" type="Car" extends="vehicleResult">  <result property="doorCount" column="door_count" /></resultMap>
```

现在 vehicleResult 和 carResult 的属性都会被加载了。
尽管曾经有些人会发现这个外部映射定义会多少有一些令人厌烦之处。
因此还有另外一种语法来做简洁的映射风格。比如:

```xml
<resultMap id="vehicleResult" type="Vehicle">
  <id property="id" column="id" />
  <result property="vin" column="vin"/>
  <result property="year" column="year"/>
  <result property="make" column="make"/>
  <result property="model" column="model"/>
  <result property="color" column="color"/>
  <discriminator javaType="int" column="vehicle_type">
    <case value="1" resultType="carResult">
      <result property="doorCount" column="door_count" />
    </case>
    <case value="2" resultType="truckResult">
      <result property="boxSize" column="box_size" />
      <result property="extendedCab" column="extended_cab" />
    </case>
    <case value="3" resultType="vanResult">
      <result property="powerSlidingDoor" column="power_sliding_door" />
    </case>
    <case value="4" resultType="suvResult">
      <result property="allWheelDrive" column="all_wheel_drive" />
    </case>
  </discriminator>
</resultMap>
```

要记得这些都是结果映射,如果你不指定任何结果,那么 MyBatis 将会为你自动匹配列
和属性。所以这些例子中的大部分是很冗长的,而其实是不需要的。也就是说,很多数据库是很复杂的,我们不太可能对所有示例都能依靠它。
