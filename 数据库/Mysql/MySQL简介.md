## 什么是MySQL

MySQL是一种关系数据库，与我们使用的大多软件一样，它由客户端和服务端组成。其中服务器程序直接和我们存储的数据打交道，客户端程序在连接到服务器程序后，发送增删改查的请求，而服务器响应请求。

### 连接

MySQL服务器程序和客户端程序本质属于计算机中的进程，MySQL服务器进程的默认名称为mysqld，MySQL客户端进程的默认名称为mysql。

客户端进程向服务器进程发送请求并得到回复的过程本质是一个进程间通信的过程，MySQL支持如下进程通信方式：

1）TCP/IP：MySQL服务器启动时默认申请3306端口号，默认监听此端口。

2）命名管道和共享内存。如果是Windows操作系统，则进程间可以考虑使用这两种方式进行通信。

3）Unix域套接字⽂件。若服务器进程和客户端进程都运行在类Unix的机器上，可使用该方法。

### 处理请求

服务器程序处理客户端的查询请求需要经过三个部分：连接管理、解析与优化、存储引擎。

**（1）连接管理**

每当有⼀个客户端进程连接到服务器进程时，服务器进程都会创建⼀个线程来专门处理与这个客户端的交互，当该客户端退出时会与服务器断开连接，服务器并不会⽴即把与该客户端交互的线程销毁 掉，⽽是把它缓存起来，在另⼀个新的客户端再进⾏连接时，把这个缓存的线程分配给该新客户端。这样就不会频繁创建和销毁线程。

客户端发起连接时，需要携带主机信息，用户名，密码。服务器程序对其进行验证。若客户端程序和服务器程序不在⼀台计算机上，可以采⽤使⽤了SSL（安全套接字）的⽹络连接进⾏通信，来保证数据传输的安全性。

**（2）解析与优化**

**查询缓存**

服务器程序在处理查询请求时，会把处理过的查询请求和结果缓存，如果下⼀次有⼀样的请求过来，直接从缓存中查找结果。这个查询缓存可以在不同客户端之间共享。

不过维护查询缓存会造成一定的开销，从MySQL 5.7.20开始，不推荐使⽤查询缓存，并在MySQL 8.0中删除。

**语法解析**

当查询缓存没有命中，则进入正式的查询阶段。客户端发送来的请求是一段文本，服务器需要对文本进行分许，判断语法是否正确，然后从文本提取出要查询的表，各种查询条件，放到服务器内部使用的一些数据结构中。

**查询优化**

MySQL 的优化程序会对语句做一些优化：如外连接转换为内连接。优化的结果是生成一个执行计划，该计划表明应使用哪些索引进行查询，表之间的连接顺序是怎样的。我们可以使用EXPLAIN语句来查看某个语句的查询计划。

**（3）存储引擎**

截止到服务器完成查询优化为止，还没有真正的访问真实的数据表。服务器把数据的存储和提取操作都封装到**存储引擎**的模块中。表是由一行一行的记录组成，而物理上如何表示记录，怎样从表中读取数据，怎么把数据写入具体的物理存储器上，这些都是存储引擎负责的事情。MySQL提供了各式各样的存储引擎，不同存储引擎管理的表具体的存储结构可能不同，采⽤的存取算法也可能不同。

我们将连接管理，查询缓存，语法解析，查询优化这些不涉及真实数据存储的功能划分为MySQL服务器的功能，而真实存取数据的功能划分为存储引擎的功能。各种不同的存储引擎向上面的MySQL服务器提供同一的调用接口，包含几十个底层函数。

因此MySQL服务器在完成查询优化后，只需按照生成的执行计划来调用底层提供的接口，获取到数据后返回给客户端即可。

我们可以通过  show engines;  语句来查询当前服务器程序支持的存储引擎。

![img](https://img-blog.csdnimg.cn/20200417173708127.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

- Support列表表示该存储引擎是否可用；
- DEFAULT值代表是当前服务器程序的默认存储引擎。
- Comment列是对存储引擎的⼀个描述。
- Transactions列代表该存储引擎是否⽀持事务处理。

我们可以为不同的表设置不同的存储引擎，即不同的表可以有不同的物理存储结构，不同的提取和写入方式。

```sqll
CREATE	TABLE	表名(
   	建表语句;
)	ENGINE	=	存储引擎名称;
```

同样也可以修改已有表的存储引擎

```sql
ALTER	TABLE	表名	ENGINE	=	存储引擎名称;
```



### 字符集

计算机只能存储二进制数据，对于字符串的存储，则需要建立字符与二进制的映射关系。我们将一个字符映射成⼀个⼆进制数据的过程称为编码，⼀个⼆进制数据映射到⼀个字符的过程叫做解码。我们将某个字符范围的编码规则称为字符集。同一个字符集可以有多种比较规则。

常见的字符集有：

1）ASCII字符集

一共128个字符，包括空格、标点符号、数字、⼤⼩写字⺟和⼀些不可⻅字符。使⽤1个字节来进⾏编码。

- 'L' -> 01001100（⼗六进制：0x4C，⼗进制：76）
- 'M' -> 01001101（⼗六进制：0x4D，⼗进制：77）

> 没有汉字

2）GB2312字符集

收录了汉字以及拉丁字⺟、希腊字⺟、⽇⽂平假名及⽚假名字⺟、俄语⻄⾥尔字⺟，同时也兼容 ASCII字符集，编码方式为：

- 如果该字符在ASCII字符集中，则采⽤1字节编码。
- 否则采⽤2字节编码。

例如字符串'爱u'，其中'爱'需要2个字节编码，，编码后的的⼗六进制表示为 0xCED2，'u'需要⽤1个字节进⾏编码，编码后的⼗六进制表示为0x75，拼合后是0xCED275。

3）GBK字符集

在收录字符范围上对GB2312字符集作了扩充，编码⽅式上兼容GB2312。

4）utf8字符集

收录地球上能想到的所有字符，⽽且还在不断扩充。这种字符集兼容ASCII字符集，编码⼀个字符需要使⽤1～4个字节。

> 对于同一个字符，不同字符集可能有不同编码方式=，比如对于汉字'我'，utf8和gb2312字符集的编码方式为：
>
> - utf8编码：111001101000100010010001 (3个字节，⼗六进制表示是：0xE68891)
> - gb2312编码：1100111011010010 (2个字节，⼗六进制表示是：0xCED2)

若编码和解码使用的字符集不同，则导致两者的结果出现错误。



**MySQL⽀持的字符集和排序规则**

MySQL支持字符集utf8和utf8mb4，不过此处的utf8只使用1~3个字节表示字符。

我们可以通过 SHOW CHARSET; 语句来查看MySQL中⽀持的字符集。

一种字符集可能对于若干个比较规则，因此MySQL支持的比较规则很多。可以通过如下语句查看：

```sql
SHOW COLLATION LIKE 'utf8\_%';
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



## 存储引擎InnoDB

MySQL默认存储引擎为InnoDB，它将表中的数据存储到磁盘上，因此即便关机，我们的数据仍就存在。真正处理数据的过程发生在内存，因此需要把磁盘中的数据加载到内存中。若是处理写入或修改请求，则还需要把内存中的内容刷新到磁盘上。

由于读写磁盘的速度比内存读写慢的很多，因此当我们想从表中获取某些记录时，InnoDB不是一条一条的读取数据，而时将数据分为若干个页，以页作为磁盘和内存之间交换的单位。

### 数据⻚结构

我们平时以记录为单位，向表中插入数据，存放表中记录的的页称为数据页，大小为16KB。

<img src="https://img-blog.csdnimg.cn/20200417192242640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70" alt="img" style="zoom:50%;" />![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



InnoDB把页中的记录划分为若干组，每个组的最后一个记录的地址偏移量单独提取出来，按顺序存储到靠近页的尾部的地方。该地方是**Page Directory****页目录**。页目录的这些地址偏移量被称为槽，页目录是由槽组成的。

![img](https://img-blog.csdnimg.cn/20200418090404893.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在一个页中根据主键查找记录分为两步：

- 通过二分法找到记录所在的槽
- 通过next_record属性遍历该槽所在的组的各个记录

每个数据页的File Header部分都有上⼀个和下⼀个⻚的编号，所以所有的数据⻚会组成⼀个双链表。

![img](https://img-blog.csdnimg.cn/20200418084645440.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)





**MyISAM和InnoDB区别**

（1）MyISAM 只有表级锁，而InnoDB 支持行级锁和表级锁，默认为行级锁。

（2）MyISAM不支持事务，而InnoDB支持事务，且支持事物的回滚和崩溃恢复能力。支持MVCC。

（3）如果执行大量的SELECT，MyISAM是更好的选择；执行大量的INSERT、UPDATE或DELETE，出于性能方面的考虑，应该使用InnoDB

（4）当执行删除时，InnoDB不会重新建立表，而是一行一行的删除，在innodb上如果要清空保存有大量数据的表，最好使用truncate table这个命令；MyISAM则重新建表

（5）MyISAM不支持外键；InnoDB支持外键。



## 参考资料

[MySQL是怎样运行的：从根儿上理解MySQL](https://juejin.im/book/5bffcbc9f265da614b11b731)