#  Spring   （构造）

### hello spring

- maven依赖

- ```xml
  <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-webmvc</artifactId>
     <version>5.1.10.RELEASE</version>
  </dependency>
  
  ```

- beans.xml

- ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd">
  
     <!--bean就是java对象 , 由Spring创建和管理-->
     <bean id="hello" class="com.kuang.pojo.Hello">
         <property name="name" value="Spring"/>
     </bean>
  
  </beans>
  
  ```

- 测试

- ```java
  @Test
  public void test(){
     //解析beans.xml文件 , 生成管理相应的Bean对象
     ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
     //getBean : 参数即为spring配置文件中bean的id .
     Hello hello = (Hello) context.getBean("hello");
     hello.show();
  }
  
  ```

- 小结：

  - 在生成Bean对象时自动调用无参构造函数，用set方法赋值。**<property name="  "** 的name表示**setName**函数里的**Name**

### 通过有参构造方法来创建

- beans.xml有三种方式编写

- ```xml
  <!-- 第一种根据index参数下标设置 -->
  <bean id="userT" class="com.kuang.pojo.UserT">
     <!-- index指构造方法 , 下标从0开始 -->
     <constructor-arg index="0" value="kuangshen2"/>
  </bean>
  <!-- 第二种根据参数名字设置 -->
  <bean id="userT" class="com.kuang.pojo.UserT">
     <!-- name指参数名 -->
     <constructor-arg name="name" value="kuangshen2"/>
  </bean>
  <!-- 第三种根据参数类型设置 -->
  <bean id="userT" class="com.kuang.pojo.UserT">
     <constructor-arg type="java.lang.String" value="kuangshen2"/>
  </bean>
  
  ```

### Spring配置

#### 1.别名：

- alias 设置别名 , 为bean设置别名 , 可以设置多个别名

- ```xml
  <!--设置别名：在获取Bean的时候可以使用别名获取-->
  <alias name="原名" alias="别名"/>
  ```

#### 2.Bean配置

- ```xml
  <!--bean就是java对象,由Spring创建和管理-->
  
  <!--
     id 是bean的标识符,要唯一,如果没有配置id,name就是默认标识符
     如果配置id,又配置了name,那么name是别名
     name可以设置多个别名,可以用逗号,分号,空格隔开
     如果不配置id和name,可以根据applicationContext.getBean(.class)获取对象;
  
  class是bean的全限定名=包名+类名
  -->
  <bean id="hello" name="hello2 h2,h3;h4" class="com.kuang.pojo.Hello">
     <property name="name" value="Spring"/>
  </bean>
  ```

#### 3.import

- applicationContext.xml

- ```xml
  <import resource="{path}/beans.xml"/>
  ```

### 依赖注入

- 概念:
  - 依赖注入（Dependency Injection,DI）。
  - 依赖 : 指Bean对象的创建依赖于容器 . Bean对象的依赖资源 .
  - 注入 : 指Bean对象所依赖的资源 , 由容器来设置和装配 .

#### 1.构造器注入：

- 前面的

#### 2.set注入：

- 要求被注入的属性 , 必须有set方法 , set方法的方法名由set + 属性首字母大写 , 如果属性是boolean类型 , 没有set方法 , 是 is .

#### 3.拓展注入：

##### 1.常量注入：

- ```xml
   <bean id="student" class="com.kuang.pojo.Student">
       <property name="name" value="小明"/>
   </bean>
  ```

#####  2.bean注入：

- 注意点：这里的值是一个引用，ref

- ```xml
   <bean id="addr" class="com.kuang.pojo.Address">
       <property name="address" value="重庆"/>
   </bean>
   
   <bean id="student" class="com.kuang.pojo.Student">
       <property name="name" value="小明"/>
       <property name="address" ref="addr"/>
   </bean>
  ```

##### 3.数组注入

- ```xml
   <bean id="student" class="com.kuang.pojo.Student">
       <property name="name" value="小明"/>
       <property name="address" ref="addr"/>
       <property name="books">
           <array>
               <value>西游记</value>
               <value>红楼梦</value>
               <value>水浒传</value>
           </array>
       </property>
   </bean>
  ```

##### 4.List注入

```xml
 <property name="hobbys">
     <list>
         <value>听歌</value>
         <value>看电影</value>
         <value>爬山</value>
     </list>
 </property>
```

##### 5.Map注入

```xml
 <property name="card">
     <map>
         <entry key="中国邮政" value="456456456465456"/>
         <entry key="建设" value="1456682255511"/>
     </map>
 </property>
```

##### 6.set注入

```xml
 <property name="games">
     <set>
         <value>LOL</value>
         <value>BOB</value>
         <value>COC</value>
     </set>
 </property>
```

##### 7.null注入

```xml
 <property name="wife"><null/></property>
```

##### 8.Properites注入

```xml
 <property name="info">
     <props>
         <prop key="学号">20190604</prop>
         <prop key="性别">男</prop>
         <prop key="姓名">小明</prop>
     </props>
 </property>
```

##### 9、p命名和c命名注入

- 1、P命名空间注入 : 需要在头文件中加入约束文件

  ```xml
   导入约束 : xmlns:p="http://www.springframework.org/schema/p"
   
   <!--P(属性: properties)命名空间 , 直接注入属性-->
   <bean id="user" class="com.kuang.pojo.User" p:name="狂神" p:age="18"/>
  ```

  

- 2、c 命名空间注入 : 需要在头文件中加入约束文件

  ```xml
   导入约束 : xmlns:c="http://www.springframework.org/schema/c"
   <!--C(构造: Constructor)命名空间 , 使用构造器注入-->
   <bean id="user" class="com.kuang.pojo.User" c:name="狂神" c:age="18"/>
  ```

## bean的作用域

在Spring中，那些组成应用程序的主体及由Spring IoC容器所管理的对象，被称之为bean。简单地讲，bean就是由IoC容器初始化、装配及管理的对象 .

| 类别      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| singleton | 在spring Ioc 容器中仅存在一个bean实例，bean以单例方式存在，默认值 |
| prototype | 每次从容器中调用bean时，都返回一个新的实例，即每次调用getBean（）时，相当于执行new XxxBean |
| request   | 每次Http请求都会创建一个新的Bean，该作用域仅适用于WebApplicationContext环境 |
| session   | 同一个Http Session 共享一个Bean，不同Session使用不同Bean,仅适用于WebApplicationContext环境 |

几种作用域中，request、session作用域仅在基于web的应用中使用（不必关心你所采用的是什么web应用框架），只能用在基于web的Spring ApplicationContext环境。

#### Singleton(单例模式)

当一个bean的作用域为Singleton，那么Spring IoC容器中只会存在一个共享的bean实例，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的同一实例。Singleton是单例类型，就是在创建起容器时就同时自动创建了一个bean的对象，不管你是否使用，他都存在了，每次获取到的对象都是同一个对象。注意，Singleton作用域是Spring中的缺省作用域。要在XML中将bean定义成singleton，可以这样配置：

```xml
 <bean id="ServiceImpl" class="cn.csdn.service.ServiceImpl" scope="singleton">
```

测试：

```java
 @Test
 public void test03(){
     ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
     User user = (User) context.getBean("user");
     User user2 = (User) context.getBean("user");
     System.out.println(user==user2);
 }
```

#### Prototype(原型模式)

当一个bean的作用域为Prototype，表示一个bean定义对应多个对象实例。Prototype作用域的bean会导致在每次对该bean请求（将其注入到另一个bean中，或者以程序的方式调用容器的getBean()方法）时都会创建一个新的bean实例。Prototype是原型类型，它在我们创建容器的时候并没有实例化，而是当我们获取bean的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象。根据经验，对有状态的bean应该使用prototype作用域，而对无状态的bean则应该使用singleton作用域。在XML中将bean定义成prototype，可以这样配置：

```xml
 <bean id="account" class="com.foo.DefaultAccount" scope="prototype"/>  
  或者
 <bean id="account" class="com.foo.DefaultAccount" singleton="false"/>
```

#### Request

当一个bean的作用域为Request，表示在一次HTTP请求中，一个bean定义对应一个实例；即每个HTTP请求都会有各自的bean实例，它们依据某个bean定义创建而成。该作用域仅在基于web的Spring ApplicationContext情形下有效。考虑下面bean定义：

```xml
 <bean id="loginAction" class=cn.csdn.LoginAction" scope="request"/>
```

针对每次HTTP请求，Spring容器会根据loginAction bean的定义创建一个全新的LoginAction bean实例，且该loginAction bean实例仅在当前HTTP request内有效，因此可以根据需要放心的更改所建实例的内部状态，而其他请求中根据loginAction bean定义创建的实例，将不会看到这些特定于某个请求的状态变化。当处理请求结束，request作用域的bean实例将被销毁。

#### Session

当一个bean的作用域为Session，表示在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。考虑下面bean定义：

```xml
 <bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
```

针对某个HTTP Session，Spring容器会根据userPreferences bean定义创建一个全新的userPreferences bean实例，且该userPreferences bean仅在当前HTTP Session内有效。与request作用域一样，可以根据需要放心的更改所创建实例的内部状态，而别的HTTP Session中根据userPreferences创建的实例，将不会看到这些特定于某个HTTP Session的状态变化。当HTTP Session最终被废弃的时候，在该HTTP Session作用域内的bean也会被废弃掉。

## Bean的自动装配

- 自动装配是使用spring满足bean依赖的一种方法
- spring会在应用上下文中为某个bean寻找其依赖的bean。

#### spring中bean有三种自动装配机制：

1. 在xml中显式配置
2. 在java中显示配置
3. 隐式的bean发现机制和自动装配

#### 第三种：自动化的装配bean

Spring的自动装配需要从两个角度来实现，或者说是两个操作：

1. 组件扫描(component scanning)：spring会自动发现应用上下文中所创建的bean；
2. 自动装配(autowiring)：spring自动满足bean之间的依赖，也就是我们说的IoC/DI；

组件扫描和自动装配组合发挥巨大威力，使得显示的配置降低到最少。

**推荐不使用自动装配xml配置 , 而使用注解 .**

##### 1.byName：

​     **autowire byname（按名称自动装配）**

- ```xml
  <bean id="user" class="com.kuang.pojo.User" autowire="byName">
     <property name="str" value="qinjiang"/>
  </bean>
  ```

  **小结：**

- 当一个bean节点带有 autowire byName的属性时。
  1. 将查找其类中所有的set方法名，例如setCat，获得将set去掉并且首字母小写的字符串，即cat。
  2. 去spring容器中寻找是否有此字符串名称id的对象。
  3. 如果有，就取出注入；如果没有，就报空指针异常。

#####  2.btType

- 使用autowire byType首先需要保证：同一类型的对象，在spring容器中唯一。如果不唯一，会报不唯一的异常。

- ```xml
  <bean id="user" class="com.kuang.pojo.User" autowire="byType">
  ```

##### 3.使用注解

- jdk1.5开始支持注解，spring2.5开始全面支持注解。

  准备工作：利用1、在spring配置文件中引入context文件头

  ```xml
  xmlns:context="http://www.springframework.org/schema/context"
  
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context.xsd
  ```

  2、开启属性注解支持！

  ```xml
  <context:annotation-config/>
  ```

  注解的方式注入属性。

###### @Autowired

- @Autoaired是按类型自动转配的，不支持id匹配

- 需要导入spring-aop的包

- 示例

- ```java
  public class User {
     @Autowired
     private Cat cat;
     @Autowired
     private Dog dog;
     private String str;
  
     public Cat getCat() {
         return cat;
    }
     public Dog getDog() {
         return dog;
    }
     public String getStr() {
         return str;
    }
  }
  ```

- ```xml
  <context:annotation-config/>
  
  <bean id="dog" class="com.kuang.pojo.Dog"/>
  <bean id="cat" class="com.kuang.pojo.Cat"/>
  <bean id="user" class="com.kuang.pojo.User"/>
  ```

- @Autowired(required=false) 说明：false，对象可以为null；true，对象必须存对象，不能为null。

###### @Qualifier

- @Autowired是根据类型自动装配的，加上@Qualifier则可以根据byName的方式自动装配
- @Qualifier不能单独使用。

###### @Resource

- @Resource如有指定的name属性，先按该属性进行byName方式查找装配；
- 其次再进行默认的byName方式进行装配；
- 如果以上都不成功，则按byType的方式自动装配。
- 都不成功，则报异常。

###### 小结

   @Autowired与@Resource异同：

1、@Autowired与@Resource都可以用来装配bean。都可以写在字段上，或写在setter方法上。

2、@Autowired默认按类型装配（属于spring规范），默认情况下必须要求依赖对象必须存在，如果要允许null 值，可以设置它的required属性为false，如：@Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualifier注解进行使用

3、@Resource（属于J2EE复返），默认按照名称进行装配，名称可以通过name属性进行指定。如果没有指定name属性，当注解写在字段上时，默认取字段名进行按照名称查找，如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。

它们的作用相同都是用注解方式注入对象，但执行顺序不同。@Autowired先byType，@Resource先byName。

## 使用注解开发

**在spring4之后，想要使用注解形式，必须得要引入aop的包**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200613182026555.png)

在配置文件当中，还得要引入一个context约束

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

</beans>
```

### 1.bean的实现

我们之前都是使用 bean 的标签进行bean注入，但是实际开发中，我们一般都会使用注解！

1. 配置扫描哪些包下的注解

```xml
<!--指定注解扫描包-->
<context:component-scan base-package="com.kuang.pojo"/>
```

2. 在指定包下编写类，增加注解

```java
@Component("user")
// 相当于配置文件中 <bean id="user" class="当前注解的类"/>
public class User {
   public String name = "秦疆";
```

3. 测试

   ```java
   @Test
   public void test(){
      ApplicationContext applicationContext =
          new ClassPathXmlApplicationContext("beans.xml");
      User user = (User) applicationContext.getBean("user");
      System.out.println(user.name);
   }
   ```

### 2.属性注入

- 使用注解注入属性

  1. 可以不提供set方法，直接在名上添加@value（“值”）

  ```java
  @Component("user")
  // 相当于配置文件中 <bean id="user" class="当前注解的类"/>
  public class User {
     @Value("秦疆")
     // 相当于配置文件中 <property name="name" value="秦疆"/>
     public String name;
  }
  ```

  2. 如果提供了set方法，在set方法上添加@value(“值”);

  ```java
  @Component("user")
  public class User {
  
     public String name;
  
     @Value("秦疆")
     public void setName(String name) {
         this.name = name;
    }
  }
  ```

### 3.衍生注解

- 我们这些注解，就是替代了在配置文件当中配置步骤而已！更加的方便快捷！

- **@Component三个衍生注解**

- 为了更好的进行分层，Spring可以使用其它三个注解，功能一样，目前使用哪一个功能都一样。

  - @Controller：controller层
  - @Service：service层
  - @Repository：dao层

  写上这些注解，就相当于将这个类交给Spring管理装配了！

### 4.作用域：

@scope

- singleton：默认的，Spring会采用单例模式创建这个对象。关闭工厂 ，所有的对象都会销毁。
- prototype：多例模式。关闭工厂 ，所有的对象不会销毁。内部的垃圾回收机制会回收

```java
@Controller("user")
@Scope("prototype")
public class User {
   @Value("秦疆")
   public String name;
}
```

### 5小结

**XML与注解比较**

- XML可以适用任何场景 ，结构清晰，维护方便
- 注解不是自己提供的类使用不了，开发简单方便

**xml与注解整合开发** ：推荐最佳实践

- xml管理Bean
- 注解完成属性注入
- 使用过程中， 可以不用扫描，扫描是为了类上的注解

```xml
<context:annotation-config/>  
```

作用：

- 进行注解驱动注册，从而使注解生效
- 用于激活那些已经在spring容器里注册过的bean上面的注解，也就是显示的向Spring注册
- 如果不扫描包，就需要手动配置bean
- 如果不加注解驱动，则注入的值为null！

## 基于Java类进行配置JavaConfig

 原来是 Spring 的一个子项目，它通过 Java 类的方式提供 Bean 的定义信息，在 Spring4 的版本， JavaConfig 已正式成为 Spring4 的核心功能 。

测试：

1、编写一个实体类，Dog

```java
@Component  //将这个类标注为Spring的一个组件，放到容器中！
public class Dog {
   public String name = "dog";
}
```

2、新建一个config配置包，编写一个MyConfig配置类

```java
@Configuration  //代表这是一个配置类
public class MyConfig {

   @Bean //通过方法注册一个bean，这里的返回值就Bean的类型，方法名就是bean的id！
   public Dog dog(){
       return new Dog();
  }

}
```

3、测试

```java
@Test
public void test2(){
   ApplicationContext applicationContext =
           new AnnotationConfigApplicationContext(MyConfig.class);
   Dog dog = (Dog) applicationContext.getBean("dog");
   System.out.println(dog.name);
}
```

4、成功输出结果！

**导入其他配置如何做呢？**

1、我们再编写一个配置类！

```java
@Configuration  //代表这是一个配置类
public class MyConfig2 {
}
```

2、在之前的配置类中我们来选择导入这个配置类

```java
@Configuration
@Import(MyConfig2.class)  //导入合并其他配置类，类似于配置文件中的 inculde 标签
public class MyConfig {

   @Bean
   public Dog dog(){
       return new Dog();
  }

}
```

关于这种Java类的配置方式，我们在之后的SpringBoot 和 SpringCloud中还会大量看到，我们需要知道这些注解的作用即可！

# Aop编程

## 1.静态代理设计模式

### 1.为什么需要代理设计模式：

- JavaEE分层开发中，最重要的是Service层

- service层包含哪些代码？

- ```markdown
  Service层中=核心功能（几十行，上百行代码）+额外功能（附加功能
  1. 核心功能
   - 业务运算
   - DAO调用
  2. 额外功能
   - 不属于核心业务
   - 可有可无
   - 代码量少
  
   事务、日志、性能——
  ```

### 2.代理设计模式

#### 1.1概念

- 通过代理类，为原始类（目标）增加额外的功能
- 好处：利于原始类（目标）的维护

#### 1.2 名词解释

- 1. 目标类 原始类
     - 指业务类（核心功能--》业务运算 DAO调用
  2. 目标方法，原始方法
     - 目标类（原始类）中的方法 就是目标方法（原始）
  3. 额外功能（附加功能）
     - 日志，事务，性能

#### 1.3 代理开发的核心要素

1. 代理类=目标类（原始类）+额外功能+原始类（目标类）实现相同的接口

#### 1.4编码

- 静态代理：为每一个原始类，手工编写一个代理类（.java.class)

#### 1.5静态代理存在的问题

1. 静态文件数量过多，不利于项目管理
2. 额外功能维护性差
   - 代理类中 额外功能修改复杂

## 2.Spring的动态代理开发

### 1.Spring动态代理的概念

1. 概念：通过代理类为原始类（目标类）增加额外功能
2. 好处：利于原始类（目标类）的维护

### 2.搭建开发环境

```xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.2.7.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.5</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>1.8.8</version>
        </dependency>
```

### 3.Spring动态代理的开发步骤

1. 创建原始对象（目标对象）

   - serviceImpl对象

   - beans

     ```xml
     <bean id="userService" class="com.djc.Service.UserServiceImpl"/>
     ```

2. 额外功能

   - MethodBeforeAdvice接口

   - 额外功能书写在接口的实现上

   - ```java
     public class Before implements MethodBeforeAdvice {
         /*
         作用：需要把运行在原始方法执行之前运行的额外功能，书写在before方法中
          */
         @Override
         public void before(Method method, Object[] objects, Object o) throws Throwable {
             System.out.println("--------method before advice log--------");
         }
     }
     ```

   - ```xml
         <bean id="before" class="com.djc.dynamic.Before"/>
     ```

3. 定义切入点

   - 切入点：额外功能加入的位置

   - 目的：由程序员根据自己的需求，决定额外功能加入给那个原始方法

   - ```xml
         <aop:config>
             <aop:pointcut id="pc" expression="execution(**(..))"/>
         </aop:config>
     ```

4. 组装（2,3整合）

   - ```xml
        <aop:config>
     <!--        所有方法，都作为切入点，加入额外功能-->
             <aop:pointcut id="pc" expression="execution(**(..))"/>
     <!--        组装：目的把切入点与额外功能进行整合-->
             <aop:advisor advice-ref="before" pointcut-ref="pc"/>
         </aop:config>
     ```

5. 调用

   - 目的：获得Spring工厂创建的动态代理对象，并进行调用

   - ```java
     ApplicationContext context=new ClassPathXmlApplicationContext("beans.xml");
     ```

   - 注意：

     1. Spring的工厂通过原始对象的id获取得的是代理对象
     2. 获取代理对象后，可以通过声明接口类型，进行对象的存储

   - ```java
     Uservice userService = (Uservice) context.getBean("userService");
     ```

### 4.动态代理细节分析

1. Spring创建的动态代理类在哪里？
   - Spring框架在运行时，通过动态字节码技术，在JVM内部，等程序结束后，会和JVM一起消失
   - 动态字节码技术：通过第三方字节码框架，在JVM中创建对应类的字节码，进而创建对象，当虚拟机结束，动态字节码跟着消失
   - 结论：动态代理不需要定义类文件，都是JVM运行过程中动态创建的，所以不会造成静态代理，类文件过多，影响项目管理的问题。

- ![image-20220817182808976](C:\Users\25694\AppData\Roaming\Typora\typora-user-images\image-20220817182808976.png)

2. 动态代理编程简化代理的开发
   1. 在额外功能不改变的前提下，创建其他目标类（原始类）的代理对象时，只需要指定原始（目标）对象即可。
3. 动态代理额外功能的维护性大大增强

## 3.Spring动态代理详解

### 1.额外功能详解

- MethodBeforeAdvice分析

- ```java
  1. MethodBeforeAdvice接口作用：额外功能运行在原始方法执行之前，进行额外功能操作
  public class Before implements MethodBeforeAdvice {
      /*
      作用：需要把运行在原始方法执行之前运行的额外功能，书写在before方法中
      Method:额外功能所增加给的那个原始方法
            login：方法
            register方法
       Object【】：额外功能所增加给那个原始方法的参数
       Object：。。。。原始对象UserServiceImpl，OrderSrviceImpl
       */
      @Override
      public void before(Method method, Object[] objects, Object o) throws Throwable {
          System.out.println("--------method before advice log--------");
      }
  }
  
  2.before方法的3个参数，在实战中，如何使用
      会根据需要进行使用，不一定都会用到，也有可能不用。   
  ```

- MethodIntercept（方法拦截器）

  - MethodBeforeAdvice--》原始方法执行之前
  - MethodIntercept————》前后都行

  ```java
  public class Arround implements MethodInterceptor {
      /*
         invoke方法的作用：额外功能写在invoke
                         额外功能  原始方法之前
                                  原始方法之后
                                  原始方法执行之前 之后
                确定：原始方法怎么运行
  
                参数：MethodInvocation （Method）：额外功能所增加给的那个原始方法
                     methodInvocation.proceed()————》让原始方法运行
                返回值：Object:原始方法执行后的返回值
       */
      @Override
      public Object invoke(MethodInvocation methodInvocation) throws Throwable {
          System.out.println("-----拦截器——----");
          Object ret=methodInvocation.proceed();//
          System.out.println("-----拦截器------");
          return ret;
      }
  }
  ```

- 什么样的功能在前后都添加

  - 事务。。。

- 额外功能在原始方法抛出异常后执行

  ```java
  try {
      ret=methodInvocation.proceed();//
  }catch (Throwable throwable){
      System.out.println("----抛出异常-----");
      throwable.printStackTrace();
  }
  ```

- MethodIntercept影响原始方法的返回值
  1. 原始方法作为返回值，直接作为invoke返回值，MethodIntercept不会影响原始方法的返回值
  2. MethodIntercept影响原始方法的返回值：
     - Invoke方法的返回值，不要直接返回原始方法的运行结果即可

### 2.切入点详解

- 切入点决定额外功能加入位置（方法）

- ```xml
  <aop:pointcut id="pc" expression="execution(* *(..))"/>
  execution(* *(..))--》匹配所有方法
  
  
  1.exexution()切入点函数
  2.* *(..)     切入点表达式
  第一个  *  返回值
  第二个  *  方法名
  （）       参数表
  ..        对于参数没有要求（参数有没有，参数有几个，参数是什么类型都行
  ```

####  2 .1 切入点表达式

- 定义login方法作为切入点

  - ~~~ xml
    * login(..)
    ~~~

- 定义login方法切login方法有2个字符串类型的参数作为切入点

  - ~~~xml
    * login(String ，String)
    注意
    非java.long包中的类型，要写全限定名称
    * login（String，..）  ..表示任意参数
    ~~~

- 上面所讲的方法切入点表达式不清楚

- 精准方法切入点限定

- ~~~xml
  修饰符 返回值             包.类.方法（参数）
  *                      com.djc.Service.UserviceImpl.login(..)
  ~~~


#### 2.2 类类切入点

- ```xml
    *    com.djc.Service.UserServiceImpl.*(..)
    ```

- ~~~xml
    忽略包（一个*代表一层）
    *    com.djc.*.UserServiceImpl.*(..)
    ~~~

#### 2.3包切入点

- ```xml
    切入点包中的类必须在Service中，而不能是子包
    *    com.djc.Service.*.*(..)
    ```

- ```
    切入点当前包及其子包都生效
    * com.djc.Service..*.*(..)
    ```

#### 2.切入点函数

- 切入点函数：用于执行切入点表达式

1. excution

    - 最重要的切入点函数，功能最全
    - 执行  方法切入点表达式  类切入点表达式 包切入点表达式
    -  
    - 弊端：  书写麻烦
    - 注意：其他切入点函数 简化execution书写复杂度，功能上完全一致

2. aegs

    - 作用：主要用于函数（方法）参数的匹配
    - 切入点：方法参数必须得是两个字符串类型的参数
    - agrs(String,String)==execution(* *(String,String))

3. within:

    - 作用：主要用于进行类，包切入点表达式的匹配

    - 切入点：UserServiceImpl这个类/Service包

    - ```xml
          within(*..UserServiceImpl)==execution(*  *..UserServiceImpl.*(..))
          within(con.djc.Service..*)==execution(* com.djc.Service..*.*.(..))
      ```

4. @annotaion

    - ```xml
        为具有特殊注解的方法加入额外功能
        <aop:pointcut id="pc" expression="@annotation(com.djc.Log)"/>
        ```

    - ```java
        @Target(ElementType.METHOD)
        @Retention(RetentionPolicy.RUNTIME)
        public @interface Log {
        
        }
        ```

5. 切入点函数的逻辑运算
    - 指 综合多个切入点函数一起配合工作，进而完成更为复杂的需求
        - and与操作
        - 例：login同时 参数两个字符串
            1. execution(* login(String,String))
            2. execution(* login(..) and args(String,String))
            3. 注意：与操作不能用于同种类型切入点的操作
        - or或操作
        - 例:
            - xxx or  xxx

## 4.AOP编程

### 1.aop概念

- AOP :  面向切面  ==Spring动态代理开发
    - 以切面为基本单位，通过切面间的彼此协同，相互调用，完成程序的构建
    - 切面=切入点+额外功能
- OOP：面向对象
- POP：面向过程

```markdown
AOP的概念：
       本质是Spring的动态代理开发，通过代理类为原始类增加额外功能
       好处：利于原始类的维护
注意：AOP编程不可能取代OOP,OOP编程有意补充
```

###  2.AOP编程的开发步骤

~~~markdown
1. 元素对象
2. 额外功能
3. 切入点
4. 组装切面（额外功能+切入点）
~~~

### 3.切面的名词解释

~~~markdown
切面=切入点+额外功能

~~~

## 5.AOP底层实现原理

#### 1.核心问题

1. 如何创建动态代理类（动态字节码技术）

看pdf



- JDK动态代理  Proxy.newProxyInstance()   通过接口创建代理的实现类
- Cglib动态代理  Enhancer                              通过继承父类创建的代理

#### 2.Spring工厂如何加工创建代理对象 

## 6.基于注解的AOP编程

### 1.基于注解的AOP编程开发步骤

1. 原始对象

2. 额外功能

3. 切入点

4. 组装切面

    ```java
    //通过切面类 定义了 额外功能 @Around
    public class MyAspect {
        @Around("execution(* login(..))")
        public Object arround(ProceedingJoinPoint joinPoint) throws Throwable {
            System.out.println("---------aspect log-------");
            Object proceed = joinPoint.proceed();
             return proceed;
        }
    }
    ```

    ```xml
        <bean id="service" class="com.djc.Service.UserServiceImpl" />
         <bean id="arround" class="com.djc.Aspect.MyAspect"/>
         <aop:aspectj-autoproxy/>
    ```

### 2.细节

1. 切入点复用

    ```java
    ：在切面类中定义一个函数 上面@Pointcut 注解 通过这种方式，定义切入点表达式，后续更加有利于切入点复用
    public class MyAspect {
        @Pointcut("execution(* login(..))")
        public void aa(){}
        @Around(value = "aa()")
        public Object arround(ProceedingJoinPoint joinPoint) throws Throwable {
            System.out.println("---------aspect log-------");
            Object proceed = joinPoint.proceed();
             return proceed;
        }
        @Around(value = "aa()")
        public Object arround2(ProceedingJoinPoint joinPoint) throws Throwable {
            System.out.println("---------aspect log-------");
            Object proceed = joinPoint.proceed();
            return proceed;
        }
    }
    ```

2. 动态代理创建方式

    ```markdown
    AOP底层实现  2种代理创建方式
    1. JDK    通过实现接口  做新的实现方式   创建代理对象
    2. Cglib  通过继承父类  做新的子类      创建代理对象
    
    默认情况下 AOP编程 底层应用JDK动态代理创建方式
    如切换为Cglib
             1. 基于注解AOP开发（false时JDK）
                  <aop:aspectj-autoproxy proxy-target-class="true"/>
             2. 传统的AOP开发
                      <aop:config proxy-target-class="true">
            <aop:pointcut id="pc" expression="@annotation(com.djc.Log)"/>
            <aop:advisor advice-ref="int" pointcut-ref="pc"/>
                  </aop:config>
    ```

## 7.AOP开发的坑

1. 坑：在同一个业务类中，进行业务方法间的相互调用，只有最外层的方法，才是加入了额外功能（内部的方法，通过普通的方式调用，都调用的是原始方法）。如果想让内层的方法也调用代理对象的方法，就要ApplicationContextAware获得工厂，进而获取代理对象。

    ```java
    public class UserServiceImpl implements UserService, ApplicationContextAware {
        private ApplicationContext context;
        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            this.context=applicationContext;
        }
    
        @Log
        @Override
        public void register(User user) {
            System.out.println("UserviceImpl.register");
            UserService userService= (UserService) context.getBean("userService");
            userService.login("1,","2");
        }
        @Log
        @Override
        public boolean login(String name, String password) {
            System.out.println("UserviceImpl.login");
            return true;
        }
    }
    ```

## 8.AOP阶段知识总结。

![image-20220818165609290](C:\Users\25694\AppData\Roaming\Typora\typora-user-images\image-20220818165609290.png)

# 持久层整合

## 1.持久层整合

1. Spring框架为什么要与持久层技术整合
2. Spring可以与哪些持久层技术进行整合

## 2.Spring与MyBatis整合

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.7.RELEASE</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.18</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.1.14.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.48</version>
</dependency>
```

- 配置文件
- ![image-20220818191427846](E:\tools\Typora\photo\image-20220818191427846.png)

- 编码：

- 整合细节：

    ![image-20220818192543800](E:\tools\Typora\photo\image-20220818192543800.png)

## 3.Spring的事务处理

### 1.什么是事务。

~~~markdown
保证业务操作完整性的一种数据库机制

事务的特点： A C I D
1. A 原子行
2. C 一致性
3. I 隔离性
4. D 持久性
~~~

### 2.如何控制事务

### 3.Spring控制事务的开发

- Spring是通过AOP方式进行事务开发

![image-20220818194621425](E:\tools\Typora\photo\image-20220818194621425.png)

3. 切入点

    ```markdown
    @Trabsectional
    事务的额外功能加入给哪些业务方法
    
    1. 类上：类中所有的方法都会加入事务
    2. 方法上：这个方法会加入事务
    ```

4. 组装切面

    ```markdown
    1. 切入点
    2. 额外功能
    
    <tx:annotation-driven transaction-manager=""/>
    ```

### 4.Spring控制事务的编码

- 搭建开发环境

- ```xml
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.3.22</version>
    ```

- 编码
- ![image-20220818200204473](E:\tools\Typora\photo\image-20220818200204473.png)

- 细节
- ![image-20220818200819334](E:\tools\Typora\photo\image-20220818200819334.png)

## 4.Spring中的事务属性（Transaction Attribute）

### 1. 什么是事务属性

![image-20220818201211484](E:\tools\Typora\photo\image-20220818201211484.png)

### 2.如何添加事务属性

```
@Transactional（isloation=，propagation=，readOnly=，timeout=，rollbackFor=，noRollbackFor=）
```

### 3.事务属性详解

#### 1.隔离属性（ISOLATION）

- 隔离属性的概念

    ```markdown
    概念：描述了事务解决并发问题的特性
    1. 什么是并发
           多个事务在同一时间，访问操作了相同的数据
           
           同一时间：0.000几秒 微小前 微小后
           
    2. 并发会产生什么问题
            1. 脏读
            2. 不可重复度
            3. 幻影读
    3. 并发问题如何解决
            通过隔离属性解决，隔离属性中设置不同的值，解决并发过程中的问题
    ```

- 事务并发产生的问题

    1. 脏读
        - 一个事务，读取了另一个事务中没有提交的数据，会在本事务中产生数据不一致的问题
        - 解决方案：@Transaction（isolation=Isolation.READ_COMMITTED)

    2. 不可重复读
        - 一个事务中：多次读取相同的数据，但是读取结果不一样。会在本事务中产生数据不一致的问题
        - 注意： 1. 不是脏读，2 ，一个事务中
        - 解决方案：@Transaction（isolation=Isolation.REPEATABLE_READ)
        - 本质是一把行锁
    3. 幻影读
        - 一个事务中，多次对整表进行查询统计，但结果不一样，会在本事务中产生数据不一致的问题
        - 解决方案：@Transaction（isolation=Isolation.SERIALIZABLE)
        - 本质：表锁

- 总结
- ![image-20220818204655197](E:\tools\Typora\photo\image-20220818204655197.png)

- 数据库对隔离属性的支持

- | 隔离属性的值     | Mysql | Oracle |
    | ---------------- | ----- | ------ |
    | .READ_COMMITTED  | ok    | ok     |
    | REPEATABLE_READ) | ok    | no     |
    | SERIALIZABLE     | ok    | ok     |

    ![image-20220818205107116](E:\tools\Typora\photo\image-20220818205107116.png)

- 默认隔离属性
    - ISOLATION_DEFAUL：会调用不同数据库所设置的默认隔离属性
    - MySQL  ：REPEATABLE_READ
    - Oracle  ： READ_COMMITED
    - ![image-20220818205441446](E:\tools\Typora\photo\image-20220818205441446.png)

- 隔离属性在实战中的建议
    - 推荐使用Spring指定的ISOLATION_DEFAULT
        1. MySQL  repeatable_read
        2. Oracle  read_commited
    - 未来实战中，并发访问情况很低
    - 如果真遇到并发问题，乐观锁
    - ​         Hibernate（JPA） Version
    - ​        MyBatis                  通过拦截器自定义开发

#### 2.传播属性（PROPAGATION）

- 传播属性的概念

- ```markdown
    概念-  ：描述了事务解决问题的嵌套
    
    什么叫做事务的嵌套：他指的是一个大的事务中，包含了若干个小的事务
    
    问题：大事务中融入了很多小的事务，他们彼此影响，最终会导致外部大的事务，丧失了事务的原则性
    ```

- 默认传播属性

- ```markdown
    REQUIRED是传播属性的默认值
    ```

- 推荐传播属性的使用方式

    1. 增删改   方法：直接使用默认值REQUIRED
    2. 查询      操作：显示指定传播属性的值为SUPPORTS

- ![image-20220819104204269](E:\tools\Typora\photo\image-20220819104204269.png)

#### 3.只读属性

- 针对于只进行查询操作的业务方法，可以加入只读属性提高运行效率
- readOnly  默认为false

#### 4.超时属性

- 指定了事务等待的最长时间
    - 当前事务访问数据时，有可能访问的数据被别的事务进行加锁的处理，那么此时本事务就必须进行等待
    - 等待时间  秒
    - @Transactional（timeout=2）
    - 超时属性的默认值  -1
        - 最终由数据库来指定

#### 5，异常属性

- Spring事务处理过程中
- 默认   对于RuntimeException及其子类  采用的是回滚策略
- 默认   对于Exception及其子类 采用的是提交的策略
-  
- rollbackFor={java.lang.Rxception,xxx,xxx}
- noRollbackFor={java.lang.RuntimrRxception,xxx,xxx}
- ![image-20220819110938334](E:\tools\Typora\photo\image-20220819110938334.png)

### 4.事务属性常见配置总结

![image-20220819111211578](E:\tools\Typora\photo\image-20220819111211578.png)

### 5.基于标签的事务配置方式（事务开发的第二种方式）

- ```xml
    基于注解 @Transaction的事务配置回顾
    <bean id="userService" class="com.baizhiedu.service.UserServiceImpl">
     <property name="userDAO" ref="userDAO"/>
    </bean>
    <!--DataSourceTransactionManager-->
    <bean id="dataSourceTransactionManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManage
    r">
     <property name="dataSource" ref="dataSource"/>
    </bean>
    @Transactional(isolation=,propagation=,...)
    public class UserServiceImpl implements UserService {
     private UserDAO userDAO;
    <tx:annotation-driven transactionmanager="dataSourceTransactionManager"/>
    基于标签的事务配置
    <bean id="userService" class="com.baizhiedu.service.UserServiceImpl">
     <property name="userDAO" ref="userDAO"/>
    </bean>
    <!--DataSourceTransactionManager-->
    <bean id="dataSourceTransactionManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManage
    r">
     <property name="dataSource" ref="dataSource"/>
    </bean>
    事务属性
    <tx:advice id="txAdvice" transacationmanager="dataSourceTransactionManager">
     <tx:attributes>
     <tx:method name="register" isoloation="",propagation="">
    </tx:method>
     <tx:method name="login" .....></tx:method>
     等效于
     @Transactional(isolation=,propagation=,)
     public void register(){
     
     }
     
     </tx:attributes>
    </tx:advice> <aop:config>
     <aop:pointcut id="pc" expression="execution(*
    com.baizhiedu.service.UserServiceImpl.register(..))"></aop:pointcut>
     <aop:advisor advice-ref="txAdvice" pointcut-ref="pc">
    </aop:advisor>
    </aop:config>
    ```

    - 基于标签的事务在实战开发中的应用

    - ```xml
        <bean id="userService"
        class="com.baizhiedu.service.UserServiceImpl">
         <property name="userDAO" ref="userDAO"/>
        </bean>
        <!--DataSourceTransactionManager-->
        <bean id="dataSourceTransactionManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionMa
        nager">
         <property name="dataSource" ref="dataSource"/>
        </bean>
        编程时候 service中负责进⾏增删改操作的⽅法 都以modify开头
         查询操作 命名⽆所谓
        <tx:advice id="txAdvice" transacationmanager="dataSourceTransactionManager">
         <tx:attributes>
         <tx:method name="register"></tx:method>
         <tx:method name="modify*"></tx:method>
         <tx:method name="*" propagation="SUPPORTS" readonly="true"></tx:method>
         </tx:attributes>
        </tx:advice>
        应⽤的过程中，service放置到service包中
        <aop:config>
         <aop:pointcut id="pc" expression="execution(*
        com.baizhiedu.service..*.*(..))"></aop:pointcut>
         <aop:advisor advice-ref="txAdvice" pointcut-ref="pc">
        </aop:advisor>
        </aop:config>
        ```

# MVC框架整合

## 1.搭建Web运行环境

```xml
<dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
  </dependency>
    <!--
    https://mvnrepository.com/artifact/javax.servlet.jsp/javax.servlet.js
    p-api -->
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>javax.servlet.jsp-api</artifactId>
      <version>2.3.1</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.1.14.RELEASE</version>
  </dependency>
    <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.1.14.RELEASE</version>
  </dependency>
    <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>5.1.14.RELEASE</version>
  </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>5.1.14.RELEASE</version>
    </dependency>
    <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.1.14.RELEASE</version>
  </dependency>
    <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.2</version>
  </dependency>
    <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.18</version>
  </dependency>
    <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.48</version>
  </dependency>
    <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
  </dependency>
    <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
  </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/springcontext -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.1.4.RELEASE</version>
    </dependency>
    <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>5.1.14.RELEASE</version>
  </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
```

## 2.为什么要整合MVC框架

![image-20220819144501723](E:\tools\Typora\photo\image-20220819144501723.png)

## 3.Spring可以整合哪些MVC框架

![image-20220819144616787](E:\tools\Typora\photo\image-20220819144616787.png)

## 4.Spring整合MVC框架的核心思路

1. 准备工厂

    ```markdown
    1. Web开发过程中如何创建⼯⼚
     ApplicationContext ctx = new
    ClassPathXmlApplicationContext("/applicationContext.xml");
     WebXmlApplicationContext()
    2. 如何保证⼯⼚唯⼀同时被共⽤
     被共⽤：Web request|session|ServletContext(application)
     ⼯⼚存储在ServletContext这个作⽤域中
    ServletContext.setAttribute("xxxx",ctx);
    唯⼀：ServletContext对象 创建的同时 ---》 ApplicationContext ctx = new
    ClassPathXmlApplicationContext("/applicationContext.xml");
     
     ServletContextListener ---> ApplicationContext ctx = new
    ClassPathXmlApplicationContext("/applicationContext.xml");
     ServletContextListener 在ServletContext对象创建的同时，被调⽤(只会
    被调⽤⼀次) ，把⼯⼚创建的代码，写在ServletContextListener中，也会保证只调⽤
     ⼀次，最终⼯⼚就保证了唯⼀性
    3. 总结
     ServletContextListener(唯⼀)
     ApplicationContext ctx = new
    ClassPathXmlApplicationContext("/applicationContext.xml");
     ServletContext.setAttribute("xxx",ctx) (共⽤)
     
    4. Spring封装了⼀个ContextLoaderListener
     1. 创建⼯⼚
     2. 把⼯⼚存在ServletContext中
    ```

    ```xml
    ContextLoaderListener使⽤⽅式
    web.xml
    <listener>
     <listenerclass>org.springframework.web.context.ContextLoaderListener</listenerclass>
    </listener> <context-param>
     <param-name>contextConfigLocation</param-name>
     <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
    ```

2. 代码整合

    ![image-20220819150331190](E:\tools\Typora\photo\image-20220819150331190.png)

#  注解编程

## 1. 注解基础概念

### 1. 什么是注解编程

- ![image-20220819153424426](E:\tools\Typora\photo\image-20220819153424426.png)

### 2. 为什么要讲注解编程

- ![image-20220819153512928](E:\tools\Typora\photo\image-20220819153512928.png)

### 3.注解的作用

- 替换xml这种配置形式
- ![image-20220819153801789](E:\tools\Typora\photo\image-20220819153801789.png)

- 替换接口，实现双方的契约性
    1. **通过注解的方式，在功能调用者和提供者之间达成约定，进而进行功能的调用。因为注解应用更为方便灵活，所以在现在的开发中，更推荐通过注解的形式**
    2. ![image-20220819160223735](E:\tools\Typora\photo\image-20220819160223735.png)

### 4.Spring注解的发展历程

![image-20220819160408001](E:\tools\Typora\photo\image-20220819160408001.png)

### 5.Spring注解开发的一个问题

1. Spring基于注解进行配置后，还能否解耦合
    - 在Spring框架应用注解时，如果对注解配置的内容不满意，可以通过Spring配置文件进行覆盖。

## 2.Spring的基础注解（Spring2.x）

- 这个阶段的注解，仅仅是简化XML的配置，并不能完全替代XML

### 1.对象创建相关注解

- 搭建开发环境

- ```xml
    <context:component-scan base-backage="com.djc"/>
    作业：让Spring框架在设置包及其子包中扫描对应的注解，使其生效
    ```

- 对象创建相关注解

#### 1.@Component

- 作用：替换原有的Spring配置文件中的<bean标签

- 注意：

    - id属性 component注解 提供了默认的设置方式  ：首单词首字母小写
    - class属性  通过反射获得class内容

- ![image-20220819162400339](E:\tools\Typora\photo\image-20220819162400339.png)

- @Component细节

    - 如何显示指定工厂创建对象的id值

    - @Component（“u”）

    - Spring配置文件覆盖注解配置内容

    - ```xml
        applicationContext.xml
        
        <bean id="u" class="com.djc.bean.User"/>
        id要与@Component的一致
        ```

- @Component的衍生注解

- ```markdown
    @Repository  --->xxxDao
    @Service     ---->
    @Controller  ---->
    
    注意：本质上这些衍生注解就是@Component
             作用，细节，用法完全一致
    目的更加准确的表达一个类型的作用
    
    注意：Spring整合Mybatis开发过程中 不使用@Repository，@Component
    ```

#### 2.@Scope注解

```markdown
作用：控制简单对象的创建次数
注意：不添加@Scope Spring提供默认值 singleton

<bean id="" class="" scope="singleton|prototype"
```

#### 3.@Lazy

```markdown
作用：延迟创建单实例对象
注意：一旦使用了@Lazy注解后，Spring会在使用这个对象的时候，进行这个对象的创建
<bean id="" class="" lazy="false"/>
```

#### 生命周期方法相关注解

```markdown
1. 初始化相关方法 @PostConstruct
   InitializingBean
   <bean init-method=""/>
2. 销毁方法 @preDestrory
   DisposableBean
   <bean destory-method=""/>
   
注意： 1. 上述的两个注解并不是Spring提供的。JSR（JavaEE规范）520
      2. 再次验证，通过注解实现了接口的契约性。
```

### 2，注入相关的注解

- 用户自定义类型@Autowired

    ![image-20220819174130554](E:\tools\Typora\photo\image-20220819174130554.png)

    ```markdown
    @Autowired 细节
    1. Autowired 基于类型的注入：注入对象的类型，必须与目标成员变量类型相同或者是其子类（实现类）  【推荐】
    2. AutoWired Qualifier 基于名字注入【了解】
       基于名字的注入：注入对象的id值，必须与Qualifier注解中设置的名字相同
    3. Autowired注解放置位置
       1. 放置在对应成员变量的set方法上
       2. 直接把注解放在成员变量上，Spring通过反射直接对成员变量进行注入（赋值）【推荐】
    4. JavaEE规范中类似功能的注解
       JSR250 @Resouce（name=""）基于名字的注入·
       ==@Autowired
         @Qualifier（""）
         注意：如果在应用Resource注解时，名字没有配对成功，那么他会继续按照类型进行注入
       JSR330 @Inject 作用与@Autowired完全一致 基于类型进行注入
       <dependency>
     <groupId>javax.inject</groupId>
     <artifactId>javax.inject</artifactId>
     <version>1</version>
     </dependency>
       
    ```

    

- jdk类型

    ```xml
    @Value注解完成
    1.设置xxx.properties
      id=10
      name=djc
    2.Spring的工厂读取这个配置文件
      <context:property-placeholder location=""/>
    3.代码
      属性 @Value（"${key}")
    ```

    - @PropertySource

    - ```markdown
        1. 作用：用于替换Spring配置文件中<context:property-placeholder location=""/>标签
        2. 开发步骤
            1. 设置xxx.properties
              id=10
            2. 应用@PropertySource
            3. 代码
               属性 @Value()
        ```

    - @Value使用细节

        1. 不能用在静态成员变量上

            如果应用，注入失败

        2. @Value注解+Properties这种方式，不能注入集合类型

            ```markdown
            Spring提供了新的配置形式 YAML YML（SpringBoot）
            ```

### 3.注解扫描详解

```markdown
<context:component-scan base-package="com.djc"/>
当前包及其子包
```

1. 排除方式

    ![image-20220820101602170](E:\tools\Typora\photo\image-20220820101602170.png)

2. 包含方式

    ![image-20220820102612119](E:\tools\Typora\photo\image-20220820102612119.png)

### 4.对于注解开发的思考

- 配置互通

- ![image-20220820103041243](E:\tools\Typora\photo\image-20220820103041243.png)

- 什么情况下使用注解，什么情况下使用配置文件

- ```markdown
    @Component 替换 <bea
    
    基础注解（@Component @Autowired。。。）程序员开发的类型的配置
    
    1. 在程序员开发的类型上，可以加入对应的注解，进行对象的创建
       User UserService 。。。
    2. 应用其他非程序员开发的类型时，还是需要使用<bean 进行配置
       SqlSessionFactoryBean  MapperScannerConfigure
    ```

### 5.SSM整合开发（半注解开发）

## 3.Spring的高级注解（Spring3.x及以上）

####  1.配置Bean

```java
Spring在3.x提供了新的注解，用于替换XML配置文件
    @Configuration
  public class AppConfig{
      
  }
```

1. 配置Bean在应用的过程中 替换了xml具体什么内容

- ![image-20220820111654931](E:\tools\Typora\photo\image-20220820111654931.png)

2. AnnotationConfigApplicationContext

    ```markdown
    1. 创建工厂
       ApplicationContext context=new 。。。。。。；
    2. 指定配置文件
       1. 指定配置bean的Class
           ApplicationContext context=new 。。。。。（AppConfig.class);
       2. 指定配置bean所在的路径
           ApplicationContext context=new 。。。。。。（“com.djc")
    ```

- 配置Bean开发的细节分析

    - 基于注解开发使用日志

        ```markdown
        不能集成Log4j
        集成logback
        ```

        - 引入相关jar

            ```
            
            ```

        - 引入logback配置文件

        - ```xml
                      <dependency>
                        <groupId>org.slf4j</groupId>
                        <artifactId>slf4j-api</artifactId>
                        <version>1.7.36</version>
                    </dependency>
                    <dependency>
                        <groupId>org.slf4j</groupId>
                        <artifactId>jcl-over-slf4j</artifactId>
                        <version>1.7.25</version>
                    </dependency>
                    <dependency>
                        <groupId>ch.qos.logback</groupId>
                        <artifactId>logback-classic</artifactId>
                        <version>1.2.3</version>
                    </dependency>
                    <dependency>
                        <groupId>ch.qos.logback</groupId>
                        <artifactId>logback-core</artifactId>
                        <version>1.2.3</version>
                    </dependency>
                    <dependency>
                        <groupId>org.logback-extensions</groupId>
                        <artifactId>logback-ext-spring</artifactId>
                        <version>0.1.5</version>
                    </dependency>
            ```

        - 引入logback配置文件

        - ```xml
            <?xml version="1.0" encoding="UTF-8"?>
            <configuration>
                <!--控制台输出 -->
                <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
                    <encoder>
                        <!--格式化输出：%d表示日期，thread表示线程名，%-5level：级别从左显示5个字符宽度msg：日志消息，%n是换行符-->
                        <pattern>%d(yyyy-MM-dd HH:mm:ss.SS5} [8thread] 8-5level %logger{50} - %m5g%n</pattern>
                    </encoder>
                </appender>
                <root Level="DEBUG">
                    <appender-ref ref="STDOUT" />
                </root>
            </configuration>
            ```

    - @Congiguration注解的本质

        ```markdown
        本质： 也是@Configuration注解的衍生注解
        
        也可以用<context:component-scan进行扫描
        ```

#### 2.@Bean注解

~~~markdown
@Bean注解在配置bean中进行使用，等同于XML配置文件中的<bean标签
~~~

1. @Bean注解的基本使用

    - 对象的创建

    - ![image-20220820121918075](E:\tools\Typora\photo\image-20220820121918075.png)

    - ```markdown
        1. 简单对象
            直接能够通过new方式创建的对象
            User UserService。。。
        2. 复杂对象
          不能通过new的方式直接创建的对象
          Connection SqlSessionFactory
        ```

    - @Bean注解创建复杂对象的注意事项

        ```java
        @Bean
        public Connection conn1 () {Connection conn = null;try i
        ConnectionFactoryBean factoryBean = new ConnectionFactoryBean() ;
                                    conn = factoryBean . get0bject( );
        }catch (Exception e) {
        e.printStackTrace( );}
        return conn ;}
        
        ```

    - 自定义id值

    @Bean("xxx")

    - 控制对象创建次数

    ```java
    @Bean
    @Scope（"Singleton|prototype") 默认值 singleton
    ```

2. @Bean的注入

    - 用户自定义类型

    - ![image-20220820153233679](E:\tools\Typora\photo\image-20220820153233679.png)

    - JDK类型的注入

    - ![image-20220820153523799](E:\tools\Typora\photo\image-20220820153523799.png)

    - JDK类型注入的细节

        ![image-20220820154323251](E:\tools\Typora\photo\image-20220820154323251.png)

#### 3.@ComponentScan注解

```
@ComponentScan 注解在配置bean中进行使用，等同于XML配置文件中的<context:component-scan>标签

目的：进行相关注解的扫描（@Component @Value。。。。@Autowired）
```

##### 1.基本使用

```java
@Configuration
@ComponentScan(basePackages = "com.djc.Use")
public class Config {

}
```

##### 2.排除、包含地使用

- 排除

- ```java
    @ComponentScan(basePackages = "com.djc.Use"
                   ,excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,value = {Service.class})
                   ,@ComponentScan.Filter(type = FilterType.ASPECTJ,pattern = "*..User1")})
    
    ```

    ![image-20220820161002198](E:\tools\Typora\photo\image-20220820161002198.png)

- 包含

- ```java
    @ComponentScan(basePackages = "com.djc.Use",
                   useDefaultFilters = false
                   ,includeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,value = {Service.class})
                   ,@ComponentScan.Filter(type = FilterType.ASPECTJ,pattern = "*..User1")})
    ```

## 4.Spring工厂创建对象的多种方式

#### 1.多种方式的应用场景

![image-20220820163056493](E:\tools\Typora\photo\image-20220820163056493.png)

#### 2. 配置优先级

```
@Component及其衍生注解 < @Bean < 配置文件的bean标签
优先级高的配置 覆盖优先级第配置

配置覆盖： id值 保持一致
```

- 解决基于注解进行配置的耦合问题
- ![image-20220820164755425](E:\tools\Typora\photo\image-20220820164755425.png)

## 5.整合多个配置信息

- 为什么会有多个配置信息

    拆分多个配置bean的开发，是一种模块化开发的方式，也体现了面向对象各司其职的设计思想

- 多配置信息的整合方式

    - 多个配置Bean的整合
    - 配置Bean与@Component相关注解的整合
    - 配置Bean与SpringXML配置文件的整合

- 整合多种配置需要关注哪些要点

    - 如何使用多配置的信息 汇总成一个整体
    - 如何实现跨配置的注入

#### 1.多个配置Bean的整合

- 多配置的信息汇总

    - base-package进行多个配置Bean的整合

        ![image-20220820171609897](E:\tools\Typora\photo\image-20220820171609897.png)

    - @Import

    - ```markdown
        1. 可以创建对象
        2. 多配置bean的整合
        ```

        ![image-20220820171939675](E:\tools\Typora\photo\image-20220820171939675.png)

    - 在工厂创建时，指定多个配置Bean的Class对象【了解】

    - ```
                 ApplicationContext context=new AnnotationConfigApplicationContext(Config.class，Config2.class);
                    
        ```

- 跨配置进行注入

    ![image-20220820182916777](E:\tools\Typora\photo\image-20220820182916777.png)

#### 2.配置Bean与@Component相关注解的整合

![image-20220820183651639](E:\tools\Typora\photo\image-20220820183651639.png)

#### 3.配置Bean与配置文件整合

![image-20220820184456812](E:\tools\Typora\photo\image-20220820184456812.png)

## 6.配置Bean底层实现原理

```
Spring在配置Bean中加入了@Configuration注解后，底层就会通过Cglib的代理方式，来进行对象相关的配置、处理
```

![image-20220820185703628](E:\tools\Typora\photo\image-20220820185703628.png)

## 7.四维一体的开发思想

### 1.什么是四维一体

```markdown
Spring开发一个功能的4种形式，虽然开发方式不同，但是最终效果是一样的。
1. 基于schema
2. 基于特定功能注解
3. 基于原始<bean
4. 基于@Bean注解
```

### 2.四维一体的开发案例

![image-20220820192234936](E:\tools\Typora\photo\image-20220820192234936.png)

## 8.纯注解版AOP编程

### 1.搭建环境

```markdown
1. 应用配置Bean
2. 注解扫描
```

### 2.开发步骤

![image-20220820193245323](E:\tools\Typora\photo\image-20220820193245323.png)

### 3.注解AOP细节分析

![image-20220820193924656](E:\tools\Typora\photo\image-20220820193924656.png)

## 9.纯注解版Spring+MyBatis整合



### 11.Spring框架中YML的使用

### 1.什么是YML

  ![image-20220820205459798](E:\tools\Typora\photo\image-20220820205459798.png)

![image-20220820210106592](E:\tools\Typora\photo\image-20220820210106592.png)