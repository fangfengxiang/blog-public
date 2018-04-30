3,mysql语句编写
===

### 面试题
有A(id,c1,c2),B(id,age,c1,c2)两张表,其中a.id与b.id关联,现在要求写出一条SQL语句，将B中记录的c1,c2更新到A表中同一记录的c1,c2字段(关联更新)
```
UPDATE A,B,SET A.c1 = B.c1,A.c2 = B.c2 WHERE A.id = B.id

UPDATE A INNER JOIN B ON A.id = B.id SET A.c1 = B.c1,A.c2 = B.c2
```
### 考点
关联更新
关联查询

### 关联查询语句
#### 分类
- 交叉关联 cross join  笛卡尔积
- 内连接 inner join  
- 外连接 left join / right join
- 联合 union / union all
- 全连接 full join 按道理不支持

#### 交叉连接
> SELECT * FROM A , B 
SELECT * FROM A CROSS JOIN B 

没有任何关联条件，结果是笛卡尔积，结果集会很大，没有意见，很少使用

#### 内连接
> SELECT  * FROM A , B WHERE A. id= B.id
SELECT * FROM A INNER JOIN B ON A.id=B.id

多表中同时符合某种条件的数据记录集合

- 等值连接 ： ON A.id = B.id 
- 不等值连接 ：ON A.id > B.id 
- 自连接 : SELECT * FROM A t1 INNER JOIN A t2 ON t1.id = t2.pid

#### 外连接
> 左外连接 LEFT OUT JOIN ，以左表为主，先查出左表，按照ON后的关联条件匹配右表，没有匹配到的用NUll填充，可以简写成LEFT JOIN

- 右外连接 RIGHT OUTER JOIN ，以右表为主。
- 外连接和内连接的区别在于 外联接以一个表为关联主表，按照ON后的条件匹配，没有匹配到就用null填充，内连接，没有主表，没有匹配到则不显示

#### 联合查询
> SELECT  * 	FROM  A UNION SELECT * FROM B
联合查询就是把多个结果集集中到一起，UNION前的结果为基准，需要注意的是联合查询的列要相等，相同的记录行会合并
 
如果使用UNION ALl  的话就不会合并重复的记录行
效率 方面 
####  全连接
- MySQL 不支持全连接
- 解决方案可以使用LEFT JOIN 和 UNION 和RIGHT JOIN 联合使用
```
SELECT * FROM A LEFT JOIN B ON A.id=B.id UNION SELECT * FROM A RIGHT JOIN B ON A.id = B.id
```
#### 嵌套查询
- 不建议使用，效率不好把握
>用一条SQL语句的结果作为另一条SQL语句的条件
```
SELECT * FORM A WHERE id IN (SELECT id FORM B)
```
#### 解题方法

