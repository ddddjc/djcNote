# 类加载与字节码技术

 ## 1.类文件结构 黑马p96-p105



## 2.字节码指令

### 2.1

### 2.2 javap工具

Oracle提供了javap工具来翻倍反编译class 文件

`javap -v java.class`

### 2.3 try -catch -finally

在字节码角度，在try部分的字节码内，如果发生异常，则捕获到异常表。如果catch里面的异常可以捕获，则使用catch里的异常，如果不能，则使用any的异常来捕获。

finally部分的内容， 在字节码里则是复制了多分，负责在try部分后面，catch内容后面以及any异常后面，保证是否发生异常，是否捕获到异常都能执行finally内容。不要在finally里面return，可能会把tyr里面的return吞掉

![image-20230720113606734](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307201136834.png)

- 由于finally中的 ireturn 被插入了所有可能·的流程，结果会以finally的为准。

### 2.4 synchronized

黑马p129

## 3. 编译期处理

所谓的`语法糖`，其实就是指java编译器把\*.java源码编译为\* .class 字节码的过程中，自动生成和转换的一些代码，主要是为了减轻程序员的负担，算是java编译器给我们的一个额外福利

### 3.1默认构造器

```java
public class Candy1{
    
}
-->
public class Candy1{
    //无参构造器是编译期帮我们加上去的
    public candy1{
        super();
    }
}
```

### 3.2 自动拆装箱

在编译期间自动进行java基本数据类型和包装数据类型中间的转换

```java
     public static void main(String[] args) {
            Integer x=1;
            int y=x;
        }
-->
     public static void main(String[] args) {
            Integer x=Integer.valueOf(1);
            int y=x.intValue();
        }
```

### 3.3 泛型集合取值

java在编译泛型代码后会执行`泛型檫除`的动作，即泛型信息在编译为字节码最后就丢失了，实际的类型都当做了Object类型来处理

```java
        public static void main(String[] args) {
            List<Integer> list=new ArrayList<>();
            list.add(10); //实际调用的是List.add(Object e)
            Integer x=list.get(0); //实际调用的是 Object obj =list.get(int idex);
        }
```

所以在取值时，百年一起真正生成的字节码中，还要额外做一个类型转换的操作

```java
//需要将Object 转化为 Integer
Integer x=(Integer)list.get(0);
```

如果前面的x变量类型修改为int基本类型，那么最终生成的字节码相当于：

```java
//将x变量类型修改为int基本类型那么最终生成的字节码是
int x =((Integer)list.get(0)).intValue();
```

`见解：这实际上操作的还是Objcet对象，只不过自动进行了类型的转换，让我们在编写代码的时候少写了重复的代码，所以也被称为语法糖`

### 3.4 可变参数

```java
        public static void main(String[] args) {
            str("111","222");
        }
		public static void str(String... args){
            System.out.println(args[0]);
            String[] arry=args;
            System.out.println(arry);
        }
-->
        public static void main(String[] args) {
            str(new String[]{"111","222"});
        }
    	public static void str(String[] args){
            System.out.println(args[0]);
            String[] arry=args;
            System.out.println(arry);
        }
```

可变参数String ...args实际是一个String[] args;在调用方法时，直接new一个String[]。

如果调用了`str()`（没有附带参数）则代码等价与str(new String[]{}),创建了一个空的数组，而不会传递null进去

### 3.5 foreach循环

对于数组：foreache被编辑器简化为for(int i=0;i<array.length;i++);

对于结合的循环：使用迭代其进行遍历。

```java
foreach()
-->
    List<Integer> list=new ArrarList<>();
    Iterator iter=list.iterator(); //用集合构造一个迭代器。
    while(iter.hasNext()){
        Integer e=(Integer)iter.next;
        //.....循环内代码
    }
```

注意：foreach循环写法，能够配合数组，以及所有实现了Iterable接口的集合类一起使用，其中Iterable用来获取集合的迭代器。

### 3.6数组初始化简写

```java
int[] array ={1,2,3,4,5};
-->
int[] array=new int[]{1,2,3,4,5};
```

### 3.7 switch 字符串

```java
        public static void str(String str){
            switch (str){
                case "hello":{
                    System.out.println("h");
                    break;
                }
                case "word":{
                    System.out.println("w");
                    break;
                }
            }
        }
--->
    public static void str(String str){
            byte x=-1;
            switch (str.hashCode()){
                case 99162322: //hello的hashCode
                    if (str.equals("hello")){
                        x=0;
                    }
                    break;
                case 113318802:
                    if (str.equals("word")){
                        x=1;
                    }
            }
            switch (x){
                case 0:
                    System.out.println("h");
                    break;
                case 1:
                    System.out.println("w");
            }
        }
```

`先进行hashcode判读后进行equals，是为了提高效率，减少case次数`

### 3.8 switch 枚举

```java
 enum Sex{
        MALE,FEMALE
    }
public class Candy{
	public void sss(Sex sex){
        switch (sex){
            case MALE:
                System.out.println("男");
                break;
            case FEMALE:
                System.out.println("女");
                break;
        }
    }
}
--->
    static class $MAP{
        /**
        * 定义一个合成类（仅JVM使用，对我们不可见）
        * 用来映射枚举类的ordinal 与数组元素的关系
        * 枚举的ordinal 表示对枚举对象的序号，从0开始
        * 即MALE的orinal（）=0，FEMAL的orinal（）=1
        */
        static int[] map=new int[2];//数组长度大于枚举类枚举元素个数
        static {
            map[Sex.MALE.ordinal()]=1;
            map[Sex.FEMALE.ordinal()]=2;
        }
    }
    public void sss(Sex sex){
        int x=$MAP.map[sex.ordinal()];
        switch (x){
            case 1:
                System.out.println("男");
                break;
            case 2:
                System.out.println("女");
                break;
        }
    }
```

### 3.9 枚举类

实际上相当于一个class，实例个数有限的。

```java
 enum Sex{
        MALE,FEMALE
    }
--->
public final class Sex extends Enum<Sex>{
    public static final Sex MALE;
    public static final Sex FEMALE;
    private static final Sex[] $VALUES;
    static {
        MALE =new Sex("MALE",0);
        FEMALE=new Sex("FEMALE",1);
        $VALUES =new Sex[]{MALE,FEMALE};
    }
}
```

### 3.10 try-with-resources

对需要关闭的资源处理的特殊语法：

```java
        try (资源变量 =创建资源变量){
            
        }catch (){
            
        }
```

其中资源对象需要实现AutoCloseable接口，例如 InputStream、OutoutStream、Connection、Statement、ResultSet等接口都实现了AutoCloseable，使用try-with-resources可以不用写finally语句块，编译期会帮助生成关闭资源代码。

```java
	try (InputStream im=new FileInputStream("")){
            System.out.println(im);
        }catch (IOException e){
            e.printStackTrace();
        }
}
----->
        try {
                InputStream is=new FileInputStream("");
                Throwable t=null;
                try{
                    System.out.println(is);
                }catch (Throwable e1){  //t是自己写的代码出现了异常
                    t=e1;
                    throw e1;
                }finally {
                    if (is!=null){//判断资源不为空
                        if (t!=null){//如果自己的代码有异常
                            try {
                                is.close();
                            }catch (Throwable e2){//如果close出现异常，作为被压制异常
                                t.addSuppressed(e2);
                            }
                        }else {
                            is.close(); //如果代码没有异常，close出现的异常就是最后的catch快中的e
                        }
                    }
                }
        }catch (IOException e){
            e.printStackTrace();
        }
```

- 为什么要设计一个addSuppressed(Throwable e)（添加被压制异常）的方法？
- weird防止异常信息的丢失

### 3.11 方法重写时的桥接方法

方法重写时返回值有两种情况

- 父子的返回值完全一致
- 子类返回值可以是父类返回值的子类

```java
例如：
        class A{
        public Number m(){
            return 1;
        }
    }
    class B extends A{
        @Override
        public Integer m() {
            return 2;
        }
    }
-->
        class B extends A{
        public Integer m() {
            return 2;
        }

        public synthetic bridge  Number m(){  //synthetic bridge JVM内部的合成方法，不可见，允许方法同名且参数一致
            return m();
        }
    }
```

通过反射可以观察到子类有两个m方法，且一个返回值是Integer，一个是Number

### 3.12 匿名内部类

```java
    public static void main(String[] args) {
        Runnable runnable=new Runnable() {
            @Override
            public void run() {
                System.out.println("ok");
            }
        };
    }
-->
        final class candyRunable implements Runnable{
        public candyRunable() {
        }

        @Override
        public void run() {
            System.out.println("ok");
        }
    }
//然后在使用到的地方new一个
-----------------------------------------------------
    //如果使用到了成员变量例如：
        public static void main(String[] args) {
    	int m=0;
        Runnable runnable=new Runnable() {
            @Override
            public void run() {
                System.out.println("ok"+m);
            }
        };
    }
-->
   final class candyRunable implements Runnable{
       int val$m;
        public candyRunable(int m) {
            this.val$m=m;
        }

        @Override
        public void run() {
            System.out.println("ok"+this.val$m);
        }
    }
```

**注意：**匿名内部类引用局部变量时，局部变量必须是final的，因为在创建对象时，将x的值付给了你们内哦不累对象的val$m属性，所以x不应该再发生变化了，如果发生变化，那么valx属性没有机会跟着一起变化。

## 4. 类加载阶段

类什么时候加载以及类什么时候初始化,从语境上，属于同一个问题，就是这个.class文件什么时候被读取到虚拟机内存中，并大到可用的状态，

而从`严格意义上`讲，加载和初始化，是类生命周期的两个阶段。

### 4.1 加载

![image-20230720165911049](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307201659172.png)

![image-20230720170123212](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307201701337.png)

### 4.2 链接

- 验证：验证类是否符合JVM规范，安全性检查
- 准备：为static变量分配空间设置默认值
  - static变量在JDK 7 之前存储于instanceKlass末尾，从JDK 7 开始，存储与_java_mirror 末尾
  - static变量分配空间和赋值是两个步骤，分配空间是在准备阶段完成的，负责是在初始化阶段完成的
  - 如果static 变量是final 的基本类型，那么编译阶段就确定了，赋值是在准备阶段完成
  - 如果static变量是final的，但属于引用类，那么赋值也会在初始化阶段完成
- 解析：将常量池中的符号解析为直接引用。

### 4.3 初始化

`<cinit> () v 方法`

初始化阶段即调用\<cinit> () v，虚拟机会保证这个类的【构造方法】的线程安全

#### 发生的时机

`类的初始化是【懒惰的】`

- main方法所在的类，总会被实现初始化
- 首次访问这个类的静态变量或静态方法时
- 子类初始化，如果父类还没初始化，会引发
- 子类访问父类的静态变量，只会触发父类的初始化
- Class.forName
- new会导致初始化

不会导致初始化的情况

- 访问类的static final几个跳常量（基本数据类型和字符串）不会触发初始化
- 类对象.class不会触发初始化
- 类加载器的loadClass方法
- Class.firName 的参数2为false时

## 5. 类加载器

![image-20230720200321088](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307202003193.png)

### 自定义加载器：

什么时候需要自定义加载器：

- 想加载非classpath随意路径中的类文件
- 都是通过接口来实现，希望解耦合时，常用在框架实际
- 这些类希望予以隔了，不同应用的同类名都可以额加载，不冲突，常见于tomcat容器

步骤：

1. 继承ClassLoader父类
2. 要遵循双亲委派机制，重写findClass方法
   - 注意不是重写loadClass方法，否则不会走双亲委派机制
3. 读取类文件中的字节码
4. 调用父类的defineClass方法来加载类
5. 使用者调用该类加载器的loadClass方法

[csdn加载器详解](https://blog.csdn.net/qdzjo/article/details/115572326)

## 6. 运行期优化

### 6.1 即时编译（Just In Time ，JIT）

#### 相关概念

##### **什么是动态编译**：

*动态编译*（dynamic compilation）指的是“在运行时进行编译”；与之相对的是事前编译（ahead-of-time compilation，简称AOT），也叫*静态编译*（static compilation）

##### **什么是即时编译：**

- java程序最初都是通过解释器（Interpreter）进行解释执行的，当虚拟机发现某个方法或代码块的运行特别频繁，就会把这些代码认定为``热点代码`（Hot Spot Code），为了提高热点代码的执行效率，在幸运星时，虚拟机会把这些代码编译为本地机器码，并以各种手段尽可能的进行代码优化，这个过程就叫做即时编译，运行时完成这功能的后端编译器哦被称为即时编译器。

##### **既然编译后的机器码效率高，为什么不把全部代码提前编译**

至于为什么不采用提前编译（Ahead Of Time，AOT）直接编译的方法，在峰值性能差不多的这个前提下，线下编译和即时编译就是两种选项，**各有优缺点**。JVM这样做，**主要也是看重字节码的可移植性**，而牺牲了启动性能（程序需要 JIT 预热才能达到最高性能）。

##### **什么是热点代码**：

1. 被多次调用的方法
2. 被多次执行的循环体

##### 如何判断方法或一段代码是不是热点代码

要知道方法或一段代码是不是热点代码，是不是需要触发即时编译，需要进行HOT Spot Detection （热点探测）

目前主要的热点探测方法有以下两种：

1. 基于采样的热点探测：采用这种方法的虚拟机会周期性地检查各个线程的栈顶，如果发现某种方法经常出现在栈顶，那这个方法就是`热点方法`。这种探测的好处是实现简单高效，还很容易地获取方法调用的关系（将栈堆展开即可），缺点是很难精确地确认一个方法的热度，因为收到线程阻塞或别的外界因素的影响而扰乱热点探测
2. 基于计算器的热点探测：采用这种方法的虚拟机会为每个方法（甚至是代码块）建立计数器，统计方法的执行次数，如果执行次数超过一定的阈值，就认定它是`热点方法`。这种统计方法实现复杂一些，需要为每个方法建立并维护计数器，而不能直接获取到方法的调用关系，但是他的统计结果相对更加精确严谨。

##### **HotSpot不你急中使用的是那种热点检测方法**

在Hotspot虚拟机中使用的是第二种——基于计数器的热点探测方法，因此他为每个方法准备了两个计数器：`方法调用计数器`和`回边计数器`。在确定虚拟机运行参数的前提下，这两个计数器都有一个确定的阈值每当计数器戳了的阈值溢出，就会触发JIT编译。

<p style="color: red;">方法调用计数器</p>

这个计数器用于统计方法被调用次数。

当一个方法被调用时，会先检查改方法是否存在被JIT编译过的版本，如果存在，则优先使用编译后的本地代码来执行。如果不存在已被编译过的版本，则将此方法的调用计数器值+1，然后判断方法调用器与回边计数器值之和是否超过方法调用计数器的阈值。如果超过阈值，那么将会向即时编译器提交一个该方法的代码编译请求。

如果不做任何设置，执行引擎并不会同步等待编译请求完成，而是继续进行解释器按照解释器方法执行字节码，直到提交的请求被编译器编译完成。当编译器工作完成后，这个方法调用入口地址就会系统自动改写新的，下次调用该方法时就会使用已编译的版本

![img](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307212000342.png)

<p style="color: red;">回边计数器</p>

它的作用是统计一个方法中循环代码执行的次数，在字节码中遇到控制流后跳转的指令称为“回边“。

![img](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture@master/202307212001714.png)

#### 分层优化

```java
        for (int i = 0; i < 200; i++) {
            long start = System.nanoTime();
            for (int j = 0; j < 1000; j++) {
                new Object();
            }
            long end = System.nanoTime();
            System.out.printf("%d\t%d\n", i, (end - start));
        }
```

会发现速度在逐渐加快。

***

JVM将执行状态分为5个层次

- 0层，解释执行（Interpreter)
- 1层，使用C1即时编译器执行（不带profiling）
- 2层，使用C1即时编译器执行（带基本的profiling）
- 3层，使用C1即时编译器执行（带完全的profiling）
- 4层，使用C2即时编译器执行

> profiling是指在运行过程中收集一些程序执行状态的数据，例如【方法的调用次数】，循环的【回边次数】等

即时编译器（JIT）与解释器的区别

- 解释器是将字节码解释为机器码，下次及时遇到相同的代码，仍会执行重复的解释
- JIT是将一些字节码编译为字节码，并存入CODE Cache，下次遇到相同的代码，直接执行，无需再编译
- 解释器是将字节码解释为针对所有平台都通用的机器码
- JIT会根据平台类型，生成平台特定的机器码

对于占据大部分的不常用的代码，我们无需耗费时间将其编译为机器码，而是采用解释执行的方式运行；另外一方面，对于仅占小部分的热点代码，我们可以将其编译为机器码，以达到理想的运行速度。执行效率上简单比较一下：Interpreter<C1<C2.总的目标是发现热点代码（hotspot名称的由来），优化它

> 刚才的一种优化手段称之为【逃逸分析】，发现新建的对象是否逃逸，可以使用-XX:-DoEscapeAnalysis 关闭逃逸分析，再运行刚才的示例观察结果。

#### 方法内联

``` java
    public static int square(final int i){  //一个热点方法
        return i*i;
    }            


	System.out.println(square(9)); //别的地方可能会经常调用这方法

```

如果发现square是热点方法，并且长度不会太长，会进行内联，即把方法内代码拷贝到调用者的位置

```java
System.out.println(9*9);
```

还能进行常量折叠的优化

```java
System.out.println(81); 
```

#### 字段优化

### 6.2 反射优化

略

