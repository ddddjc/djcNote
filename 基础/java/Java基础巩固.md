# Java基础内容巩固

## 数据类型：

- 基本数据类型
    - 数值型：整数（**byte（8位）,short（16位）,int（32）,long（64）**，**（值为有符号二进制补码整数）**），浮点数（**float（32）,double（64）**）,字符（**char（16位无符号整数，使用Unicode字符集**）
    - 非数值：**boolean（8）**
- 引用数据类型：
    - **class**，**interdace**，数组（**[]**)

### 整数数据类型

- 整数常量

    十进制整数：

    ​                如123 （默认int）

    ​                     123L或123l（long）

    ​                     0，0L  （默认值）

    十六进制：以0x或0X开头，如0x123表示十进制数291

     二进制：以0b或0B开头，如0b10000000 （JDK7开始支持）
      八进制：以0开头，如072

- 不同类型表示的范围：

- | 数据类型 | 所占位数 | 树的范围         |
    | -------- | -------- | ---------------- |
    | byte     | 8        | -2^7^ ~2^7^ -1   |
    | short    | 16       | -2^15^ ~2^15^ -1 |
    | int      | 32       | -2^31^ ~2^31^ -1 |
    | lng      | 64       | -2^63^ ~2^63^ -1 |

    ## 注意：
    
    - **类型转换问题：两个byte或short整数相加，结果默认转化为int，赋值给byte或short时会发生类型转换问题。**

### 浮点类型

- 浮点类型常量
    - 十进制数形式： 由数字和小数点组成，且必须有小数点，如0.123, 1.23, 123.0
    - 科学计数法形式：如：123e3或123E3，其中e或E之前必须有数字，且e或E后面的指数必须为整数。
    - 在十进制和科学计数法常数后面可以跟“F”或“f”(单精度)、“D”或“d”（双精度）,来表示float型或double
    - 的值：如1.23f，2.3e3D，默认型为双精度浮点数。
    - 默认值：0.0F，0.0

### 字符类型

- 字符常量：字符常量是用单引号括起来的一个字符。

- （1）普通字符‘a’，‘A’，

- （2）或是形如‘\u????’的Unicode形式的字符，其中“????”应严格按照四个16进制数字进行替换，  ‘\u0041’;    等价于  ‘A’; ‘\u0000’为字符类型默认值，即空字符 

- （3）单引号所引的转义字符，如 ‘\n’，表示换行。

- | 引用方法 | 对应的Unicode | 标准表示方法 | 意义          |
    | -------- | ------------- | ------------ | ------------- |
    | '\b'     | '\u0008'      | BS           | 退格          |
    | '\t'     | '\u0009'      | HT           | 水平制表符tab |
    | '\n'     | '\u000a'      | LF           | 换行          |
    | '\f'     | '\u000c'      | FF           | 表格符        |
    | '\r'     | '\u000d'      | CR           | 回车          |
    | '\"'     | '\u0022'      | "            | 双引号        |
    | '\''     | '\u0027'      | '            | 单引号        |
    | '\\'     | '\u005c'      | \            | 反斜线        |

### 数据类型转换

- 不同类型数据间的优先关系如下：

    低------------------------------------------->高

    byte,short,char-> int -> long -> float -> double

- 自动类型转换规则

    - byte、short、char类型被提升到int类型
    - 整型, 浮点型,字符型数据可以混合运算。运算中，不同类型的数据先转化为同一类型，然后进行运算，转换从低级到高级 

| 操作数1类型                         | 操作数2类型 | 转换后的类型 |
| ----------------------------------- | ----------- | ------------ |
| byte、short、char                   | int         | int          |
| byte、short、char、int              | long        | long         |
| byte、short、char、int、long        | float       | float        |
| byte、short、char、int、long、float | double      | double       |

### 基本数据类型及其包装类

| 普通数据 类型 | 对应的包 装类 |
| ------------- | ------------- |
| char          | Character     |
| byte          | Byte          |
| short         | Short         |
| int           | Integer       |
| long          | Long          |
| float         | Float         |
| double        | Double        |
| boolean       | Boolean       |

作用：

- Byte.MAX_VALUE
- Interger.parselnt("12")

## 面向对象

- 权限修饰符

| 修饰符    | 同一个类 | 同一个包中子类无关类 | 不同包的子类 | 不同包的无关类 |
| --------- | -------- | -------------------- | ------------ | -------------- |
| private   | √        |                      |              |                |
| 默认      | √        | √                    |              |                |
| protected | √        | √                    | √            |                |
| public    | √        | √                    | √            | √              |

- 状态修饰符
    - final
    - static

### 多态

**多态的形式:具体类多态，抽象类多态，接口多态。**
**多态的前提:有继承或者实现关系;有方法重写;有父(类/接口)引用指向(子/实现)类对象**

#### 1.具体类多态

- 可理解为以父类为api，仅可使用父类中定义的成员变量和方法（子类方法），比接口多了成员变量，但没有接口纯粹

- 好处：提高了持续的拓展性
- 弊端：不能使用子类特有的功能（多态的转型可应用）；

#### 2.抽象类

- 成员方法可抽象可不抽象

#### 3.接口多态

##### 1.接口概述：

- 接口是一种公共的规范标准，只要符合规范标准，大家都可以通用。
- Java中的接口更多的体现在对行为的抽象

##### 2.接口的特点：

- 接口中成员变量默认final，static；
- 可多继承

#### 4.抽象类与接口的区别：

- 成员区别
    - 抽象类        变量，常量；有构造方法，有抽象方法和非抽象方法
    - 接口            常量，抽象方法
- 关系区别
    - 类与类        继承，单继承
    - 类与接口    实现，可单实现可多
    - 接口与接口   继承，单继承，多继承
- 设计理念区别
    - 抽象类         对类抽象，包括属性，行为
    - 接口             对行为抽象，主要是行为

### 常用API（工具类）

#### 1.Math;

#### 2.System；

#### 3.Object;

#### 4.Arrays

#### 5.Data

#### 6.SimpleDataFormate

#### 7.Calender

## 集合

![image-20220905205605606](E:\tools\Typora\photo\image-20220905205605606.png)

### 1.Collection

```java
public interface Collection<E> extends Iterable<E>
```

| 方法名                                                    |
| --------------------------------------------------------- |
| int size();                                               |
| boolean isEmpty();                                        |
| boolean contains(Object o);                               |
| Object[] toArray();                                       |
| boolean add(E e);                                         |
| boolean remove(Object o);                                 |
| boolean removeAll(Collection<?> c);                       |
| void clear();                                             |
| Iterator<E> iterator();  （**迭代器 boolean hasNext()**） |

### 1.1.List

```java
public interface List<E> extends Collection<E>
```

- 有序
- 允许重复

| 特有方法（比Collection）                                |
| ------------------------------------------------------- |
| boolean addAll(int var1, Collection<? extends E> var2); |
| default void replaceAll(UnaryOperator<E> var1)          |
| default void sort(Comparator<? super E> var1)           |
| int indexOf(Object var1);                               |
| int lastIndexOf(Object var1);                           |
| ListIterator<E> listIterator(int var1);（迭代器）       |
| ListIterator<E> listIterator();                         |
| List<E> subList(int var1, int var2);                    |
|                                                         |

#### 1.ArrayList

- ```java
    public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable {
    ```

- **数组实现**

- | 方法名                               |
    | ------------------------------------ |
    | public boolean add(E e);             |
    | public void add(int index,E element) |
    | public boolean remove(Object o)      |
    | public E remove(int index)           |
    | public E set(int index,E elemeny)    |
    | public E get(..)                     |
    | public int size()                    |

#### 2.LinkedList

- ```java
    public class LinkedList<E>
        extends AbstractSequentialList<E>
        implements List<E>, Deque<E>, Cloneable, java.io.Serializable
    ```

- **双链表实现**

### 1.2.Set

```java
public interface Set<E> extends Collection<E> 
```

- 不包含重复元素
- 没有带索引方法
- 实现：
    - HashSet
    - LinkeHashSet  (按存取顺序)
    - TreeSet             (按比较器排序)

### 2.Map

```java
public interface Map<K,V> 
```

- **键不可重复**

- | boolean containsKey(Object key);     |
    | ------------------------------------ |
    | boolean containsValue(Object value); |
    | Collection<V> values();              |
    | Set<K> keySet();                     |
    | Set<Map.Entry<K, V>> entrySet();     |
    |                                      |

### 3.Collections

- **Collection的工具类**

## IO流

```java
public class File
    implements Serializable, Comparable<File>
```

### 1.File

| 构造及方法函数                              |
| ------------------------------------------- |
| public File(String pathname) {              |
| public File(String parent, String child)    |
| public File(File parent, String child)      |
| public String getName()                     |
| public String getParent()                   |
| public File getParentFile()                 |
| public boolean exists()                     |
| public boolean delete()                     |
| public void deleteOnExit()                  |
| public boolean mkdir()     （创建目录       |
| public boolean mkdirs()     (创建包括父目录 |
| public boolean createNewFile()  （创建文件  |
| public boolean isFile()                     |
| public boolean isDirectory()                |
| public String[] list()                      |
| public File[] listFiles()                   |

### 2.字节流

- InputStream

```Java
public abstract class InputStream implements Closeable
```

- ![image-20220907195738173](E:\tools\Typora\photo\image-20220907195738173.png)

- FileInputStream

    ```java
    class FileInputStream extends InputStream
    ```

    - ![image-20220907195857164](E:\tools\Typora\photo\image-20220907195857164.png)

- OutputStream

    ```java
    public abstract class OutputStream implements Closeable, Flushable {
    ```

    - ![image-20220907200043090](E:\tools\Typora\photo\image-20220907200043090.png)

- FileOutputStream

    ```java
    class FileOutputStream extends OutputStream
    ```

    - ![image-20220907200141171](E:\tools\Typora\photo\image-20220907200141171.png)



### 3.字节缓冲流

**仅仅提供缓冲区，真正的读写数据害的依靠基本的字节流对象操作**

- BufferedInputStream

- ```java
    public class BufferedInputStream extends FilterInputStream 
    ```

- ![image-20220908164938852](E:\tools\Typora\photo\image-20220908164938852.png)

- BufferedOutputStream

- ```java
    public class BufferedOutputStream extends FilterOutputStream 
    ```

- ![image-20220908165223481](E:\tools\Typora\photo\image-20220908165223481.png)

### 4.字符流

- Writer

    ```java
    public abstract class Writer implements Appendable, Closeable, Flushable 
    ```

    ![image-20220910123449170](E:\tools\Typora\photo\image-20220910123449170.png)

- InputStreamWriter

- FileWriter

- BufferedWriter

- 

- Reader

    ```java
    public abstract class Reader implements Readable, Closeable 
    ```

    ![image-20220910123515206](E:\tools\Typora\photo\image-20220910123515206.png)

- FileReader

- InputStreamRead

- BufferedReader

### 5.对象序列化流

- ObjectOutputStream

- ```java
    public class ObjectOutputStream
        extends OutputStream implements ObjectOutput, ObjectStreamConstants
    ```

- ![image-20220910130241826](E:\tools\Typora\photo\image-20220910130241826.png)

- ObjectInputStream

-  

    ```java
    public class ObjectInputStream
        extends InputStream implements ObjectInput, ObjectStreamConstants
    ```

- ![image-20220910130402157](E:\tools\Typora\photo\image-20220910130402157.png)

## 多线程

