### mysql数据库引擎

#### 逻辑分层

##### ![img](https://img2018.cnblogs.com/blog/506422/201908/506422-20190819104443534-603608371.png)

#### mysql锁机制：

​		表级锁、页面锁、行级锁

##### MyISAM锁：

​		表共享读锁、表独占写锁

​		可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定争夺：如果Table_locks_waited的值比较高，则说明存在着较严重的表级锁争用情况。

​		允许设置concurrent_insert使表支持并发插入；

​		锁调度问题：默认写操作比读优先获得锁——>MyISAM引擎不适合大量insert、delete操作

##### InnoDB锁：

行锁、支持事务

两种行锁：共享锁（读锁）+排他锁（写锁）

**加了共享锁的数据禁止其他事务加排他锁，但允许加共享锁**

**加过排他锁的数据行在其他事务种是不能修改数据的，也不能通过for update和lock in share mode锁的方式查询数据，但可以直接通过select …from…查询数据，因为普通查询没有任何锁机制。**

**意向锁：**

意向共享锁：-- 事务要获取某些行的 S 锁，必须先获得表的 IS 锁。

意向排他锁：-- 事务要获取某些行的 X 锁，必须先获得表的 IX 锁。

![è¿éåå¾çæè¿°](http://img.blog.csdn.net/20170419164638753?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc29vbmZseQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**InnoDB行锁实现：**

​		**InnoDB行锁是通过给索引上的索引项加锁来实现的**，这一点MySQL与Oracle不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB这种行锁实现特点意味着：**只有通过索引条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁！**

#### 数据库引擎：

##### 			读写数据都要通过数据库引擎，是用于存储、处理和保护数据的核心服务

​		**B+树索引不能找到具体的一条记录**，而是只能找到对应的页。**把页从磁盘装入到内存中**，再通过**Page Directory进行二分查找**，同时此**二分查找也可能找不到具体的行记录**（有可能会找到），只是能找到一个接近的链表中的点，再从此点开始遍历链表进行查找。







## Innodb：

事务型数据库，支持ACID(Atomicity,Consistency,Isolation,Durability)，支持行锁、外键。

采用**聚集索引**，叶子页数据（16K）连续，避免了读取分散页时所耗费的随机I/O。

**Innodb读写采用MVCC来支持高并发**。

1.支持事务（提交、回滚等），行级别锁定，select语句非锁定读

2.处理巨大数据量，CPU效率高

3.InnoDB存储引擎为在主内存中缓存数据和索引而维持它自己的缓冲池

​	参考：http://www.cnblogs.com/janehoo/p/7717041.html

### 什么是MVCC?

英文全称为Multi-Version Concurrency Control,翻译为中文即 多版本并发控制。在小编看来，他无非就是乐观锁的一种实现方式。在Java编程中，如果把乐观锁看成一个接口，MVCC便是这个接口的一个实现类而已。





### 数据库优化：

![img](https://images2017.cnblogs.com/blog/603809/201801/603809-20180112172203144-1124991646.png)



### 什么时候索引失效：

​    1.条件中有or，索引失效

​    2.复合索引没有使用第一项

```mysql
   alter table student add index my_index(name, age)   // name左边的列， age 右边的列 
   select * from student where name = 'aaa'     // 会用到索引
   select * from student where age = 18          //  不会使用索引
```
​    3.条件中使用like查询时，查询如果是 ‘%aaa’ 不会使用索引，而 ‘aaa%’ 会使用到索引。

​    4.字符串类型列，一定要加 “ ” 

​    5.如果mysql认为全表扫面要比使用索引快，则不使用索引。

