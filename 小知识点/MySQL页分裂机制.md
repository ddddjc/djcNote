# MySQL页分裂机制

## 数据页：![img](https://cdn.jsdelivr.net/gh/ddddjc/djcPicture@master/202312272121909.png)

## 数据区：

在MySQL设定中，同一个表空间内的一组连续的数据页为一个区（extent），默认区的大小为1MB，页大小为16KB。一个区里会有64个连续的数据页

![img](https://cdn.jsdelivr.net/gh/ddddjc/djcPicture@master/202312272124495.jpeg)

方便控制管理

## 数据页分裂问题

数据页中的每行要保证期有序性，同时，每个相邻的数据页也要保证它的有序性。因此，如果前面的某个数据页满了，如果要进行插入，这需要把一些数据移到其他数据页，这样就产生了数据页的分裂现象，

> 一般方法时保留一般数据，然后新建数据页存放另外一半，因为数据页本身时链表所以页不需要一定连续

参考：[InnoDB中的页合并与分裂 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/98818611)

[一看就懂的：MySQL数据页以及页分裂机制 - 赐我白日梦 - 博客园 (cnblogs.com)](https://www.cnblogs.com/ZhuChangwu/p/14041410.html)