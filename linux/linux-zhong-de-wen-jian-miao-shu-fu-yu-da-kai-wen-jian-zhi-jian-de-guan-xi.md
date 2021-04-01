# Linux中的文件描述符与打开文件之间的关系

## 概述

 在Linux系统中一切皆可以看成是文件，文件又可分为：普通文件、目录文件、链接文件和设备文件。

文件描述符（file descriptor）是内核为了高效管理已被打开的文件所创建的索引，其是一个非负整数（通常是小整数），用于指代被打开的文件，所有执行I/O操作的系统调用都通过文件描述符。

程序刚刚启动的时候，`0` 是标准输入，`1` 是标准输出，`2` 是标准错误。如果此时去打开一个新的文件，它的文件描述符会是 `3` 。

**POSIX标准要求每次打开文件时（含socket）必须使用当前进程中最小可用的文件描述符号码 。**因此，在网络通信过程中稍不注意就有可能造成串话。标准文件描述符图如下：

![](../.gitbook/assets/image%20%281%29.png)

文件描述与打开的文件对应模型如下图：

![](../.gitbook/assets/image%20%282%29.png)

## 文件描述限制

在编写文件操作的或者网络通信的软件时，初学者一般可能会遇到 “**Too many open files**” 的问题。这主要是因为文件描述符是系统的一个重要资源。虽然说系统内存有多少就可以打开多少的文件描述符，但是在实际实现过程中内核是会做相应的处理的，一般最大打开文件数会是系统内存的10%（以KB来计算）（称之为**系统级限制**），查看系统级别的最大打开文件数可以使用 `sysctl -a | grep fs.file-max` 命令查看。与此同时，内核为了不让某一个进程消耗掉所有的文件资源，其也会对单个进程最大打开文件数做默认值处理（称之为 **用户级限制**），默认值一般是`1024`，使用 `ulimit -n` 命令可以查看。在Web服务器中，通过更改系统默认值文件描述符的最大值来优化服务器是最常见的方式之一，具体优化方式请查看[http://blog.csdn.net/kumu\_linux/article/details/7877770。](http://blog.csdn.net/kumu_linux/article/details/7877770。)

## 文件描述符合打开文件之间的关系

每一个文件描述符会与一个打开文件相对应，同时，不同的文件描述符也会指向同一个文件。相同的文件可以被不同的进程打开，也可以在同一个进程中被多次打开。系统为每一个进程维护了一个文件描述符表，该表的值都是从0开始的，所以在不同的进程中你会看到相同的文件描述符。这种情况下相同文件描述符有可能指向同一个文件，也有可能指向不同的文件。具体情况要具体分析，要理解具体其概况如何，需要查看由内核维护的3个数据结构。 

* **进程级**的文件描述符表
* **系统级**的打开文件描述符表 
* **文件系统**的 i-node 表

进程级的描述符表的每一条目记录了单个文件描述符的相关信息。 

* 控制文件描述符操作的一组标志。（目前，此类标志仅定义了一个，即close-on-exec标志）
* 对打开文件句柄的引用

内核对所有打开的文件的文件维护有一个系统级的描述符表格（open file description table）。有时，也称之为打开文件表（open file table），并将表格中各条目称为打开文件句柄（open file handle）。一个打开文件句柄存储了与一个打开文件相关的全部信息，如下所示：

1. 当前文件偏移量（调用read\(\)和write\(\)时更新，或使用lseek\(\)直接修改） 
2. 打开文件时所使用的状态标识（即，open\(\)的flags参数）
3. 文件访问模式（如调用open\(\)时所设置的只读模式、只写模式或读写模式）
4. 与信号驱动相关的设置 
5. 对该文件 i-node 对象的引用 
6. 文件类型（例如：常规文件、套接字或FIFO）和访问权限 
7.  一个指针，指向该文件所持有的锁列表 
8. 文件的各种属性，包括文件大小以及与不同类型操作相关的时间戳

下图展示了文件描述符、打开的文件句柄以及i-node之间的关系，图中，两个进程拥有诸多打开的文件描述符。

![](../.gitbook/assets/image%20%284%29.png)

### 文件描述符复制

在进程 A 中，文件描述符 `1` 和文件描述符 `20` 都指向同一个打开文件描述体\(标号 `23` \)。 这很可能是通过调用 `dup()` 系列函数形成的。

文件描述符复制，在某些场景下非常有用，比如：标准输入/输出重定向。 在 shell 下，完成这个操作非常简单，大部分人都会，但是极少人思考过背后的原理。

大概描述一下需要的几个步骤，以标准输出\(文件描述符为 1 \)重定向为例：

1. 打开目标文件，返回文件描述符 `n` ；
2. 关闭文件描述符 `1` ；
3. 调用 dup 将文件描述符 `n` 复制到 `1` ；
4. 关闭文件描述符 `n` ；

### 子进程继承文件描述符

进程 A 的文件描述符 `2` 和进程 B 的文件描述符 `2` 都指向同一个打开文件描述体\(标号 `73` \)。 这种情形很可能发生在调用 `fork()` 派生子进程之后，比如A调用 `fork()` 派生出 B 。 这时， B 作为子进程，从父进程 A 继承了文件描述符表，其中包括图中标明的文件描述符 2 。 这就是 **子进程继承父进程打开的文件** 这句话的由来。

当然了，进程 A 通过Unix套接字将一个文件描述符传递给 B 也会出现类似的情形，但一般文件描述符数值是不一样的。 同时为 2 要非常凑巧才发生。

### 不同进程打开同一文件

进程A的描述符 `0` 和进程B的描述符 `3` 分别指向不同的打开文件句柄，但这些句柄均指向 `i-node` 表的相同条目（`1976`），换言之，指向同一个文件。发生这种情况是因为每个进程各自对同一个文件发起了`open()`调用。

### 同一进程多次打开同一文件

同一个进程多次打开同一个文件与不同进程打开同一文件相同，只是在进程的文件描述符表中有多条记录，在文件表中分别对应自己的记录，只是最中指向了同一个 inode 中的记录。 

## 总结

1. 由于进程级文件描述符表的存在，不同的进程中会出现相同的文件描述符，它们可能指向同一个文件，也可能指向不同的文件
2. 两个不同的文件描述符，若指向同一个打开文件句柄，将共享同一文件偏移量。因此，如果通过其中一个文件描述符来修改文件偏移量（由调用read\(\)、write\(\)或lseek\(\)所致），那么从另一个描述符中也会观察到变化，无论这两个文件描述符是否属于不同进程，还是同一个进程，情况都是如此。
3. 要获取和修改打开的文件标志（例如：O\_APPEND、O\_NONBLOCK和O\_ASYNC），可执行fcntl\(\)的F\_GETFL和F\_SETFL操作，其对作用域的约束与上一条颇为类似。
4. 文件描述符标志（即，close-on-exec）为进程和文件描述符所私有。对这一标志的修改将不会影响同一进程或不同进程中的其他文件描述符

## References

* [文件描述符](https://linux.fasionchan.com/zh_CN/latest/system-programming/file-io/file-descriptor.html)
* [每天进步一点点——Linux中的文件描述符与打开文件之间的关系](https://blog.csdn.net/cywosp/article/details/38965239) ,  [cywosp](https://blog.csdn.net/cywosp)
* [http://blog.chinaunix.net/uid-20633888-id-2747146.html](http://blog.chinaunix.net/uid-20633888-id-2747146.html) 
* [http://www.cppblog.com/guojingjia2006/archive/2012/11/21/195450.html](http://www.cppblog.com/guojingjia2006/archive/2012/11/21/195450.html) 
* [http://blog.csdn.net/kumu\_linux/article/details/7877770](http://blog.csdn.net/kumu_linux/article/details/7877770)
* 《Linux/UNIX系统编程手册》
