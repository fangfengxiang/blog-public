### OpenResty 

OpenResty 不等于 lua_nginx_module 它包含lua_nginx_module,测试集，resty库，工具集上一篇： [INNODB锁(上)](xxx) (不用点了，这是个flag,还没有上)。我们介绍了INNODB是如何加锁的，丁奇老师的专栏里也将加锁规则总结为：`两个原则，两个优化和一个bug` 。但是对于一些例子为啥会这样block住，为啥加个条件或字段加锁的范围改变了。还是有些疑惑，于是这两天对专栏里的例子进行了测试。在这个过程中，有了一些新思考。又自己举了一些新例子，来反证，自得其乐。故总结在这里。

#### 锁的三种算法
- Record Lock :  单个记录上的锁
- Gap Lock : 间隙锁，锁定一个范围
- Next-Key Lock : 锁定一个范围，并且锁定这个记录本身。(Next-Key Lock = Gap Lock + Record Lock)

Gap Lock 的引入是为了解决RR隔离级别下Current Read(当前读)出现Phantom Problem(幻读)(快照读幻读现象RR隔离级别由MVCC解决)。INNODB在RR隔离级别下加锁的基本单位是Next-Key Lock , 在RC隔离级别下加锁基本单位是Record Lock。所以INNODB在RR隔离级别下就没有幻读现象了。 故，本文举的例子，场景为RR隔离级别，MySQL版本为5.6。

对于MySQL的加锁过程，我个人觉得可总结为以下两个思想：
-  加锁是基于索引的，访问过的对象都要加锁 , 访问的顺序就是加锁的顺序。
-  加锁的基本单位是Next-Key Lock , Next-Key Lock 是一个前开后闭区间，在符合条件下退化为Gap Lock 或 Record Lock , 所谓的间隙锁的范围，是由间隙右值决定。

下面举几个特殊例子：
```
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `c` int(11) DEFAULT NULL,
  `d` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `c` (`c`)
) ENGINE=InnoDB
```
![图片.png](https://upload-images.jianshu.io/upload_images/5261067-e8a579bd72895f41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### select id 和 select * 在 lock in share mode 下的区别
| | SESSION A| SESSION B |
|---|---|---|
| T1| begin | |
| T2| select id from t where c = 5 lock in share mode | begin |
| T3| | update set d = 3 where id = 5 (success) |
![图片.png](https://upload-images.jianshu.io/upload_images/5261067-07811ace4b31e576.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![图片.png](https://upload-images.jianshu.io/upload_images/5261067-a902356d6eb4a29d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

| | SESSION A| SESSION B |
|---|---|---|
| T1| begin | |
| T2| select * from t where c = 5 lock in share mode| begin |
| T3| | update set d = 3 where id = 5 (block) |
![图片.png](https://upload-images.jianshu.io/upload_images/5261067-4b31feef4f3e4ab9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![图片.png](https://upload-images.jianshu.io/upload_images/5261067-088249124491231d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

why：
- `select  id from t where c = 5`  因为C是建了索引列。id 可以直接从C的索引树上拿到，不需要回表。所以加锁只加在C索引树上，主键索引没有加锁。而`select * `要获取所有列的数据，所以无法单纯从C索引树上拿到数据，要回表。根据访问过的对象都要加锁，主键索引上也会加锁，所以就block住了。
- 同理的，如果对于`select id from  t where  c  = 5 in share mode` ,此时另外一个 session 执行的不是`update t set d = 3 where id = 5 `, 而是`update t set id = 33 where id =5 ` 同样是会block住，因为id这个列存在于C索引上，修改id这个值，同时要更新C索引树上的id值，虽然主键索引上没有加锁，但是C索引树上id=5这行锁住了，所以这个语句也会block 住

### lock in share mode 和 for update 加锁的区别
| | SESSION A| SESSION B |
|---|---|---|
| T1| begin | |
| T2| select id from t where c = 5 for update| begin |
| T3| | update set d = 3 where id = 5 (block) |
why：
-  `for update` 与`lock in share mode` 的区别是 `for update` 会同时在主键索引上加锁。所以block住了。

### limit 减少加锁范围
| | SESSION A| SESSION B |
|---|---|---|
| T1| begin | |
| T2| select id from t where C > 2  lock in share mode| begin |
| T3| | insert t values(26,26,26) (block)|

| | SESSION A| SESSION B |
|---|---|---|
| T1| begin | |
| T2| select id from t where c > 2 limit 2 lock in share mode| begin |
| T3| | insert t values(9,9,9) (success)|

why：
`select id from t where C > 2  lock in share mode` 由于访问的对象是C>2 所以，next-key Lock范围是 (2, 5],(5,8],(8,10],(10,15] ~ (25,supremum]
`select id from t where c > 2 limit 2 lock in share mode` 由于limit 2 的存在，所以当扫描到C = 8 这一行时已经有两行记录了，所以不再访问。则这个加锁的范围是(2,5],(5,8] 。

延伸：
```
select id from t where C = 8 lock in share mode  
```
加锁 范围 ： (5,8],(8,10) 
影响 ：` insert t values(6,6,6) ,insert t values(9,9,9) 都block住`
```
select id from t where c= 8 limit 1 lock in share mode  (不会退化为record lock)
```
加锁范围 : (5,8]
影响： ` insert t values(6,6,6)  => block` ,`insert t values(9,9,9) => success`,` insert t values(6,8,6)  => block`,`insert t values(9,8,9) => success`
why:  主键列的大小，决定非唯一值索引相同值的间隙位置 [参考阅读](https://helloworlde.github.io/blog/blog/MySQL/MySQL-%E4%B8%AD%E5%85%B3%E4%BA%8Egap-lock-next-key-lock-%E7%9A%84%E4%B8%80%E4%B8%AA%E9%97%AE%E9%A2%98.html)

### order by desc 造成加锁范围不一致 (间隙由右值决定)
| | SESSION A| SESSION B |
|---|---|---|
| T1| begin | |
| T2| select id from t where id > 9 and id < 13 for update | begin |
| T3| | insert t values(7,7,7) (success)|
| T4| | update t set c =2 where id = 8 (success)|
| T5| | update t set c=2 where id = 15 (block)|

| | SESSION A| SESSION B |
|---|---|---|
| T1| begin | |
| T2| select id from t where id > 9 and id < 13 order by id desc for update | begin |
| T3| | update t set c = 2 where id = 15 (success)|
| T4| | update t set c = 2 where id = 5 (success)|
| T5| | update t set c =2 where id = 8 (block)|
| T6| | insert t values(7,7,7)(block)|

why:
`select id from t where id > 9 and id < 13 for update` 访问的顺序是先找到id=9 这个值，找到了(8,10)这个间隙,然后再向右遍历符合这个条件的。加锁的范围是(8,10],(10,15]。
`select id from t where id > 9 and id < 13 for update order by id  desc` 由于`order by desc` 的存在，查询优化器为了避免再排一次序，会将查找顺序优化为先找id = 13 这个值，找到(10,15)这个间隙,再向左遍历，所以加锁范围是(5,8],(8,10],(10,15) 
没有`order by id desc` 这句由于没有id = 13 这个等值查询 (10,15]这个next-key lock 无法退化为 grap lock .
由于`order by id desc` 导致遍历范围为向左遍历, 而`间隙总是由右值决定的` 所以扫到(8,10] 范围时，由于`10>9` 此时还会继续向左遍历，扫到(5,8] 发现 `8 < 9` 不满足条件，才停止向左遍历，加上这个 next-key lock

延伸：order by desc 造成加锁顺序改变
```
select * from t where id in (5,8,10) for update
```
依次给 `id =5 ,id =8, id=10 这三个记录`加record lock 
```
select * from t where id in (5,8,10)  order by id desc for update
```
依次给 `id =10,id =8, id=5 这三个记录`加record lock。

###  delete 造成的加锁间隙改变
| | SESSION A| SESSION B |
|---|---|---|
| T1| begin | |
| T2| select * from t where id > 10 and id <= 15 for update| begin |
| T3| | delete from t where id =10  (success) |
| T3| | insert t values(10,10,10) (block) |
why: 我个人在MYSQL5.6 版本并未复现。据丁奇老师说是，因为delete改变了加锁的间隙, 既原来的next-key lock 范围是(10,15],(15,20] delate id =10 这行后，next-key lock 变为(8,15],(15,20] 所以，不能重新insert 回去。
`ps: 如果读者复现了，麻烦留言介绍版本号复现步骤，thx`

---


以上就是对自己对加锁的 一些思考。由于作者见识有限，文中难免纰漏繁多。欢迎读者交流指正。

----
参考阅读：
- [InnoDB Locking (MySQL 5.6 Reference Manual)](https://dev.mysql.com/doc/refman/5.6/en/innodb-locking.html)
- [MySQL 加锁处理分析(何登成的技术博客)](http://hedengcheng.com/?p=771#_Toc374698321)
- [MySQL实战45讲(阿里丁奇专栏)](https://time.geekbang.org/column/139)

![image.png](https://upload-images.jianshu.io/upload_images/5261067-e137b8fa0a00c4ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
