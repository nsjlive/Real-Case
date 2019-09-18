#### 索引    Index

是帮助mysql 高效获取数据的**排好序的数据结构**

###### 索引保存的数据(类似于key-value)：

> 1.索引列的值 data 
>
> 2.指向数据行的指针 index

优点：

**提高检索速度，降低磁盘IO**

缺点：

**索引需要存储，也需要空间，索引实际上就是一张表，字段更新（insert,delete,update)会有性能损耗。**

建索引原则：

> **适合建索引**

> 频繁作为where条件的字段
>
> 关联子可以建索引，例如外键
>
> order by col,group by col

###### 不适合建索引

> where 条件中用不到的字段
>
> 频繁更新的字段
>
> 数据值分布比较均匀，例如真假，男女 

什么情况下索引失效：

> select  *from user order by age 索引生效
>
> Index(name ,age) 复合索引
>
> Select * from user order by age 索引失效   （必须按照顺序来走）
>
> Select * from user where age =18 and name =‘zhangfei' 索引失效

> index (age ,name)
>
> Select * from where age >18 and name ='zhangfei'  部分失效 （name 失效）  

Btree   Hash  

Btree: balance tree 

| id   | name  |
| ---- | ----- |
| 1    | zhang |
| 2    | a     |
| 3    | b     |
| 4    | c     |
| 5    | d     |
| 6    | e     |
| 7    | f     |

规定：左边放那个小的，右边放大的

sql 1: select id from table where id =?

sql 2：select if,name from table where id =7 



###### 聚集索引

> 字典最前面几页的索引，或者书的目录聚集索引和非聚集索引

非聚集索引

> 页码下面的数字

数据结构

提高检索效率

排好序

#### Mysql  存储引擎

插件式存储引擎，基于**表**

MyIsam , InnoDB,  memory 

MyIsam:不支持事务，支持表锁和全文索引，查找效率极高；(非聚集索引)

##### InnoDB： 

支持事务，行锁，查询效率相对MyIsam要低；（聚集索引）

Innodb只有一个聚集索引：

> 1. 默认会拿主键id作为聚集索引
> 2. 如果没有主键，会取非空的唯一索引作为聚集索引
> 3. 如果上面都没有，innodb 会自己维护一个唯一id来作为聚集索引

#### 业务

电商项目中，订单表，商品表

商品表：后台运营插入商品数据，前台终端用户基本上只会查看商品信息，**读多写少**  MyIsam 

订单表：**必须支持事务**，**写多读少**    InnoDB

索引必须基于存储引擎 