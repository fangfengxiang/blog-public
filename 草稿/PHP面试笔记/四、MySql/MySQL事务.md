MySQL事务
===

> 事务是数据库区别于文件系统的重要特性之一。事务用来保证数据库的完整性，要么都做修改，要么都不做。

事务有严格的定义，必须同时满足四大特性(ACID)：

- 原子性(atomicity)
原子性是指整个数据库的事务是不可分割的工作单位。只有使事务中所有的数据库操作都执行成功，才能算整个事务成功。如果事务中任何一个SQL语句执行失败，那么执行成功的SQL语句也必须撤销。数据库的状态退回执行事务之前的状态。

-  一致性(consistency)
一致性是指事务将数据库从一种状态转变为下一种一致的状态。在事务开始前和结束后，数据库的完整约束没有被破坏。

- 隔离性(isolation)
一个事务的影响在该事务的提交之前对其他事务都不可见

- 持久性(durability)
事务一旦提交，其结果是永久性的。即使发送宕机等故障。数据库也能将数据恢复。

其中原子性，一致性，持久性是通过数据库的redo和undo来实现的。隔离性是通过锁来完成的。

### 事务控制语句
- start transaction | begin 
显示地开启一个事务。在存储过程中只能使用start transaction来开启事务
- commit 
提交一个事务
- rollback 
回滚一个事务
- savepoint xxx 
在事务中创建一个保存点
- release savepoint xxx 
删除一个事务的保存点
- rollback to xxx
回滚到某个保存点
- set transaction 
设置事务的隔离级别
- set autocommit = 0
禁止自动提交，具体了解innodb的自动提交机制


### 事务隔离级别
- read uncommitted 
未提交读
- read committed
已提交读
- repeatable read
可重复读
- serializable
可序列化读，主要用于分布式事务

> innodb存储引擎默认的隔离级别是repeatable read。与其他数据库不同，innodb在repeatable read的事务隔离级别下，由于采用锁的算法不同，已经能保证事务隔离性的要求，即达到SQL标准的serializable隔离级别

隔离级别越低，事务请求的锁就越少，或者保持锁的时间就越短。

#### 隔离级别的设置与查看
- 可使用`set transaction`语句来设置隔离级别，如果想在MySQL库启动时就设置事务的默认隔离级别，那就需要修改MySQL的配置文件。在[mysqld]中添加如下行
```
[mysqld]
transaction-isolation = READ-COMMITTED
```
- 查看当前会话的事务隔离级别
`select @@tx_isolation\G`
- 查看全局的事务隔离级别
`select @@global.tx_isolation\G`

#### 不同隔离级别下的问题
- 脏读
脏读在使用read uncommitted隔离级别的时候产生，由于事务未提交时更改就可读。如果事务A读取某一行数据时，事务B正好在更改这一行记录，就会出现脏读。
- 不可重复读
不可重复读在使用read committed隔离级别时出现，由于事务提交的更改影响了另一事务的两次读取数据不一致。如果事务A读取某一行记录后，事务B更改了这一行记录并提交，此时事务A再次读取该行记录时会出现两次数据读取不一致的情况。
- 幻读
使用repeatable read隔离级别时，会读取本事务开启时数据的快照，所以不会出现两次读取数据不一致的情况。但是这个隔离级别，只能防止更新的影响，不能防止其他事务的影响。也就是说如果事务A在读取过程中，事务B新增了一行记录，可能会照成事务A前后两次读取的数据行数不同。
- serializable隔离级别下，innodb存储引擎会对读取操作加上一个共享锁。因此，在这个事务隔离级别，一致性非锁定读不再支持。serializable级别下操作开销大，一般在分布式事务中使用，不在本地事务中使用。

### 事务的实现
原子性，一致性，持久性是通过数据库的redo和undo来实现的。

#### redo 
在innodb存储引擎中，事务日志通过重做(redo)日志文件和innodb存储日志缓冲来实现。
- 当开始一个事务时，会记录该事务的一个LSN(日志序列号)
- 当事务执行时，会在innodb存储引擎的日志缓冲里插入事务日志
- 当事务提交时，必须将innodb存储引擎的日志缓冲写入磁盘。
- 也就是说innodb在写数据前，需要先写日志，这种方式称为预写日志方式。
- 重做日志(redo)记录了事务的行为，可以很好地进行“重做”。但是事务有些时候还需要撤销，这时就需要undo

#### undo 
- undo与redo相反，对于数据库进行修改时，不会产生redo，而且还会产生一定量的undo，即使你执行的事务或语句失败时，就可以利用这些undo的信息来进行回滚。
- redo是存放在重做日志中的，undo是放在数据库的特殊段，称为undo段，undo段位位于共享表空间内。

### 事务的操作统计
### 分布式事务
未学习，待补充


### 参考文章
《MySQL技术内幕：innodb存储引擎》
- [Mysql之事务详解](https://blog.csdn.net/mevicky/article/details/50332443)
- [Innodb中的事务隔离级别和锁的关系](https://tech.meituan.com/innodb-lock.html)
- [MySQL数据库事务各隔离级别加锁情况--read committed](http://www.imooc.com/article/17290?block_id=tuijian_wz)
- [何登成大神对Innodb加锁的分析](http://www.cnblogs.com/metoy/p/8073303.html)
- [Undo和Redo以及牛逼的MVCC](https://yq.aliyun.com/articles/53601)
- [MySql-Undo及Redo详解](https://blog.csdn.net/alexdamiao/article/details/51872477)