# page buffer和 buffer pool

两者的功能定位都是缓存数据以提高存取修改的速度。

存储区别：page  buffer位于内核态区域，buffer pool位于用户态。
存储对象：page buffer是用于通用文件，而buffer pool 是mysql专用的。

此外就是令人瞩目的拷贝问题，即磁盘-》page buffer -> buffer pool这两次拷贝，其中page buffer的拷贝毫无意义，甚至会占用额外的空间。


对此innodb存在着O_DIRECT模式可以绕过page buffer,并且在mysql 8.0之后的版本，innodb_flush_method的默认值是改为 `O_DIRECT`（如果操作系统支持）


# 缓存内容是什么？
buffer pool从缓存机制上讲类似于虚拟内存，即先分配地址，当使用使判断是否存在于内存当中，发生缺页中断之后再去磁盘中获取。

![](img/Pasted%20image%2020250217105809.png)



bufferpool会为上述不同类型的信息都分配足够的页，此外为了控制这些页，buffer pool为每一个页都创建了一个控制块。



