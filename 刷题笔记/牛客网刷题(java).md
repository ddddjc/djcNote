# 牛客网刷题(java)

## 语法

1. ```java
    关于下面的程序Test.java说法正确的是(程序有编译错误)。
    public class Test {
        static String x="1";
        static int y=1;
        public static void main(String args[]) {
            static int z=2;
            System.out.println(x+y+z);
        }
    }
    //静态变量属于整个类，局部变量属于方法，只在该方法中有效，所以static不能修饰局部变量。
    ```

2. final 不能修饰抽象类

## 逻辑

## 线程

## 类及工具类

### String方面

- ```java
    下面这条语句一共创建了多少个对象：String s="welcome"+"to"+360;        //1
    ```

    - 直接“xxx”的拼接操作是在java编译器编译期间就执行了，就是说直接把字面量进行“+”操作，得到"welcometo360"常量，并直接将这个常量放入字符串池中，避免了创建多余的对象。而字符串引用的“+”运算是在编译期间执行。若引用字段被final修饰，则为常量运算与直接“xxx”运算相同，结果放在常量池，否则结果在堆中。
-