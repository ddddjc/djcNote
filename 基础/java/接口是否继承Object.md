# 接口继承自Object

Java中，接口是否继承自Object？
说来惭愧，一直没想到过这个问题，女友老师灵魂一问，让我如梦初醒~

网上有这么说的：

在c++中的多继承，造成了一定程度的冗余，例如，基类B和C继承自基类A，现在有一个基类D，继承自B和C，形成了一个菱形的继承关系，当创建一个D的实例时，会同时创建两个A类的构造器，这不仅在使用上要进行区分，还对内存造成了一定量的浪费。
现在我们回到java，假如有一个接口Inter，一个基类A（继承Object），这时有一个基类B继承基类A，同时实现接口Inter，如果说interface继承自Object类，那么又会出现在c++上的多继承冗余问题，这违背了java单继承的初衷，所以我认为是没有继承Object类的

哦~有点道理

但老师说：是继承的！

空口无凭，上代码：

````java
/**
 * @author sxy
 * @Description
 * @Date 2022/3/30
 */
public interface InterfaceFather {

}
````



一个空接口，反编译一下：

![img](E:\mytool\Typora\photo\ab86dbe73df14a84a571e9658476a03c.png)

所以，老师说，接口是继承Object滴

可我还是感觉上面那位兄弟说的没毛病，，

那我只能自己动手，丰衣足食了，还是上面那个空接口，再新建一个空类：

```java
/**
 * @author sxy
 * @Description
 * @Date 2022/3/30
 */
public class ClassFather {
  
}
```



然后通过反射获取他俩及其父类的方法：

````java
/**
 * @author sxy
 * @Description
 * @Date 2022/3/30
 */
public class InterfaceFatherTest {
    public static void main(String[] args) {
        Class<InterfaceFather> interfaceClazz = InterfaceFather.class;
        Method[] interfaceMethods = interfaceClazz.getMethods();
        // 获取到的方法个数为0
        System.out.println(interfaceMethods.length);

        InterfaceFather interfaceFather = null;
        // 能访问Object里的方法（虽然不能运行）
        interfaceFather.hashCode();

        Class<ClassFather> classClazz = ClassFather.class;
        Method[] classMethods = classClazz.getMethods();
        // 可以获取到Object里的方法
        System.out.println(classMethods.length);
    }
}

````



神奇不？

总结一下目前的现象：

- 反编译看到接口继承自Object
- 反射获取不到接口继承自Object的方法，但接口对象可以访问Object的方法

特别是第二条，完全就是矛盾的

这时候，我已经觉着老师说的对了，接口肯定继承自Object，不然不能访问Object的方法

但为啥反射获取不到嘞？

我猜测问题应该出在反射上

看下getMethods()源码：

```java
@CallerSensitive
public Method[] getMethods() throws SecurityException {
     checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), true);
     return copyMethods(privateGetPublicMethods());
}

```

不得不说，人家这名字起的就是好，没猜错的话应该是进入privateGetPublicMethods()方法：

```java
private Constructor<T>[] privateGetDeclaredConstructors(boolean publicOnly) {
    Constructor<T>[] res;
    ReflectionData<T> rd = reflectionData();
    if (rd != null) {
        res = publicOnly ? rd.publicConstructors : rd.declaredConstructors;
        if (res != null) return res;
    }
    // No cached value available; request value from VM
    if (isInterface()) {
        @SuppressWarnings("unchecked")
        Constructor<T>[] temporaryRes = (Constructor<T>[]) new Constructor<?>[0];
        res = temporaryRes;
    } else {
        res = getDeclaredConstructors0(publicOnly);
    }
    if (rd != null) {
        if (publicOnly) {
            rd.publicConstructors = res;
        } else {
            rd.declaredConstructors = res;
        }
    }
    return res;
}
```

就是他：

```java
if (!isInterface()) {
            Class<?> c = getSuperclass();
```

只有是类的时候，才会去获取它的父类信息

所以，我们使用反射获取不到接口中Object的方法
————————————————
版权声明：本文为CSDN博主「importGuitar」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_43955020/article/details/123859048