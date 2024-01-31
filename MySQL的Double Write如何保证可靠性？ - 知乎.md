# MySQL的Double Write如何保证可靠性？ - 知乎
前言
--

前几篇对MySQL的知识介绍，让我们知道MySQL基本单位是数据页，默认情况下每个数据页的大小是16kb。数据页被读取到内存（Buffer Pool）中后被称为**缓存页**,，当对Buffer Pool中的数据页做了更新后，此时的数据页叫做：**脏页**，脏页最终是要刷入磁盘的，那么问题来了。

> **当数据页写入磁盘时，此时突然断电宕机了，数据只写入了一部分，数据丢失了吗？**  
> **partial page write（部分页面写入）是啥，来电重启怎么恢复？**

上面两个问题存在于这种场景：当把一个16K大小的数据页写入到磁盘中时，结果刚写了8k，突然断电机器宕机了，那此时只有一部分是写入成功的，这种情况就叫partial page write（部分页面写入）。

因为你想啊，MySQL的数据页默认是16K，而文件系统的数据页是4K，磁盘IO的最小单位是512字节，出现宕机很大可能磁盘中只有一部分写入成功，因为数据页写入到文件系统中需要经历 (16/4) 4次物理写。

而InnoDB的 **Double Write**就时用来解决partial page write问题的，具体怎么解决的，我们一探究竟。

Double Write是啥
--------------

为了解决文章开头中描述的问题，MySQL引入了double write这个特性，它针对的是脏数据（脏页），提高innodb的可靠性，用来解决部分写失败(partial page write）的问题。

我们知道被修改的数据最终是要输入到磁盘中的，为什么叫 double write呢，字面上可理解为两次写入的意思。对的，double write 刷盘是将一份脏数据写到共享【**共享表空间 ibdata**】中，一份写到真正的【**数据文件 ibd**】永久的保存，所以就叫double wriete。此时double write中存在副本，可以直接覆盖到ibd中对应的页中 。

Double Write写入流程
----------------

![](https://pic2.zhimg.com/v2-0c3ab3d1ae1c85e67e268ec195c03a29_b.jpg)

我们可以看到 Double Write）由两部分组成：内存中的 Double Write Buffer 和 磁盘上的 ibdata1的两个区（连续的128页，2M大小）

具体的实现步骤如下：

1.  用户修改过得数据页（脏页）先Copy到Double Write Buffer中
2.  然后从Double Write Buffer 中以1MB大小顺序写入磁盘中的【共享表空间 ibdata】（因为该空间维护的是128个连续页，最小16KB，所以不会出现部分写失效情况）
3.  再写入到磁盘的【数据文件 ibd】

是的，一切看起来都是那么的合乎逻辑，但是这里有几个问题，写Double Write 时候Crash了怎么办？写ibd时Crash了怎么办？

嗯，带着这些问题，继续往下看！

崩溃场景和恢复
-------

> **1：写Double Write 的时候Crash了怎么办？**

在Double Write Buffer写Double Write是Crash掉了，此时ibd还是原始干净数据，因为还没从Double Write Buffer写到ibd呢。

其实还有Redo Log文件，服务重启动后通过redo进行恢复 ，因为Redo Log是在修改数据页前完成的（WAL：预写日志），即DB需要保证Redo Log先完整安全地落盘，然后才能去修改对应的数据页（先在内存Buffer Pool上修改修改）。

此时根据Redo Log文件向缓存池中加载数据页，通过比较Redo Log与数据页的LSN大小 redo log lsn > page lsn，来更新数据页，更新完成后该数据页为脏页，然后再讲脏页复制到Double Write Buffer，重复上面写入原理的步骤。

我们将上面Double Write的原理图再完善一点，整个恢复过程如下：

![](https://pic1.zhimg.com/v2-daf25c0a05a09707b6ec8ac5480450c4_b.jpg)

> **2：写ibd时Crash了怎么办？**

写ibd时发生crash，因此此时Double Write中存在副本，可以直接覆盖到ibd中对应的页中，然后再继续redo恢复。

> **3：为何需要Double Write？有Redo Log还不够吗？**

我们知道Redo Log是被MySQL设计为异常崩溃恢复的，Double Write Buffer同样是为了保证数据完整性。

因为Redo Log一种操作日志，记录的是 “ 在某个数据页上做了什么修改 ”，用于MySQL异常崩溃恢复使用，本质上是物理日志。问题时如果数据页本身已经发生了损坏，Redo Log来恢复已经损坏的数据块就无效了，因为数据快已经是一个损坏的坏块。

而此时恢复数据需要的是一个跟原始数据块一样的页，而Double Write刚好存的就是数据块的副本，然后Redo Log再对数据块进行重做操作进行恢复。

可以这么说Double Write与Redo Log对于容灾场景，缺一不可。

总结
--

Double Write 部分页写入问题，崩溃恢复策略解决了每个阶段的Crash问题，结合Redo Log对崩溃数据能进行完成性恢复。

数据页落盘刷新的过程如下：

1.  Mysql的WAL日志预写日志，在数据页修改前先记录Redo Log
2.  Buffer Pool 数据页先copy到Double Write Buffer 的内存里
3.  Double Write Buffer 的内存数据刷到Double Write【磁盘中的共享表空间 ibdata】
4.  Double Write Buffer 的内存再刷到数据磁盘上【数据文件 ibd】