# 内存模型（详细看juc）

## java内存模型

与【java内存结构】有区别。

`java内存模型`是Java Memory Model（JMM）

简单地说：JMM定义了一套在多线程读写共享数据时（成员变量、数组），对数据得物可见性、有序性、和原子性的规则和保障

### 1.原子性

解决：synchronized

### 2.可见性

解决 volatile

### 3.有序性

### 4. cas