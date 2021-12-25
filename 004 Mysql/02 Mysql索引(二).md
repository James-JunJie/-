#Explain工具介绍

```sql
 DROP TABLE IF EXISTS `actor`;
 CREATE TABLE `actor` (
`id` int(11) NOT NULL,
`name` varchar(45) DEFAULT NULL,
`update_time` datetime DEFAULT NULL,
 PRIMARY KEY (`id`)
    )
 ENGINE=InnoDB DEFAULT CHARSET=utf8;
 INSERT INTO `actor` (`id`, `name`, `update_time`) VALUES (1,'a','2020-10-01 11:53:00'), (2,'b','2020-10-01 11:53:00'), (3,'c','2020-10-01 11:53:02');

 DROP TABLE IF EXISTS `film`;
 CREATE TABLE `film` (
 `id` int(11) NOT NULL AUTO_INCREMENT,
 `name` varchar(10) DEFAULT NULL,
 PRIMARY KEY (`id`),
 KEY `idx_name` (`name`)
            )
 ENGINE=InnoDB DEFAULT CHARSET=utf8;
 INSERT INTO `film` (`id`, `name`) VALUES (3,'film0'),(1,'film1'),(2,'film2');
 
 DROP TABLE IF EXISTS `film_actor`;
 CREATE TABLE `film_actor` (`id` int(11) NOT NULL,
 `film_id` int(11) NOT NULL,
 `actor_id` int(11) NOT NULL,
 `remark` varchar(255) DEFAULT NULL,
 PRIMARY KEY (`id`),
 KEY `idx_film_actor_id` (`film_id`,`actor_id`)
 ENGINE=InnoDB DEFAULT CHARSET=utf8;
 INSERT INTO `film_actor` (`id`, `film_id`, `actor_id`) VALUES (1,1,1),(2,1,2),(3,2,1);
```

> 衍生表开关
>
> SET SESSION optimizer_switch = 'derived_merge=off'

##explain中的列

###1.id

id列的编号是 select 的序列号

id值越大，优先级越高，id相同则从上往下执行，id为NULL最后执行。

###2.select_type列

**1）simple**：简单查询。查询不包含子查询和union

> mysql> explain select * from film where id = 2;

**2）primary**：复杂查询中最外层的 select

**3）subquery**：包含在 select 中的子查询（不在 from 子句中）
4）**derived**：包含在 from 子句中的子查询。MySQL会将结果存放在一个临时表中，也称为**派生表**（derived的英文含义）

```sql
set session optimizer_switch='derived_merge=off'; #关闭mysql5.7新特性对衍生表的合并优化
explain select (select 1 from actor where id = 1) from 
(select * from film where id = 1) der;
```

<img src="02 Mysql索引(二).assets/image-20210525211255615.png" alt="image-20210525211255615" style="zoom: 150%;" />

###4. type列

解释：

这一列表示关联类型或访问类型，即MySQL决定如何查找表中的行，查找数据行记录的大概范围。
依次从最优到最差分别为：**system > const > eq_ref > ref > range > index > ALL**
一般来说，得保证查询达到range级别，最好达到ref

> explain select min(id) from film;

**NULL**：mysql能够在优化阶段分解查询语句，在执行阶段用**不着再访问表**或**索引**。例如：在
索引列中选取最小值，可以**单独查找索引**来完成，不需要在执行时访问表

**const, system**:

mysql能对查询的某部分进行优化并将其转化成一个常量（可以看showwarnings 的结果）。

用于 primary key 或 unique key 的所有列与常数比较时，所以表最多有一个匹配行，读取1次

system是const的特例，表里只有一条元组匹配时。

>  explain extended select * from (select * from film where id = 1) tmp;

![image-20210525213537821](02 Mysql索引(二).assets/image-20210525213537821.png)

####ref

ref：相比 eq_ref，不使用唯一索引，而是使用普通索引或者唯一性索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行。

1. 简单 select 查询，name是普通索引（非唯一索引）

####range

范围扫描通常出现在 in(), between ,> ,<, >= 等操作中。使用一个索引来检索给定范围的行。

> mysql> explain select * from actor where id > 1;

####index

index：扫描全表索引，这通常比ALL快一些。

> mysql> explain select * from film; 

####All

即全表扫描，意味着mysql需要从头到尾去查找所需要的行。通常情况下这需要增加索引来进行优化了

> explain select * from actor;

![image-20210525164949455](02 Mysql索引(二).assets/image-20210525164949455.png)

###5. possible_keys列

可能使用哪些索引来查找

![image-20210525165355688](02 Mysql索引(二).assets/image-20210525165355688.png)

###6.key列

### 7.key_len列

### 10. Extra列

1）Using index：使用覆盖索引

> mysql> explain select film_id from film_ac tor where film_id = 1;

![image-20210525173417563](02 Mysql索引(二).assets/image-20210525173417563.png)



3）Using index condition：查询的列不完全被索引覆盖，where条件中是一个前导列的范围；

4）Using temporary：mysql需要创建一张临时表来处理查询。出现这种情况一般是要**进行优化的**，首先是想到用**索引**来优化。

1. actor.name没有索引，此时创建了张临时表来distinct

> explain select distinct name from actor;

![image-20210525173619316](02 Mysql索引(二).assets/image-20210525173619316.png)

5）Using filesort：将用外部排序而不是索引排序，数据较小时从内存排序，否则需要在磁盘完成排序。这种情况下一般也是要考虑使用索引来优化的。

1. actor.name未创建索引，会浏览actor整个表，保存排序关键字name和对应的id，然后排序name并检索行记录
  
  > mysql> explain select * from actor order by name; 
2. film.name建立了idx_name索引,此时查询时extra是using index

  > mysql> explain select * from film order by name;

###覆盖索引：

查询的所有字段，是在key里面，不用去加载data(紫色部分)

<img src="Mysql索引.assets/image-20210505172548617.png" alt="image-20210505172548617" style="zoom:80%;" />



# 索引最佳实践

##1.全值匹配

 

##2.最左匹配原则：

如果索引了多列，要遵守最左前缀法则。指的是**查询**从索引的**最左前列开始**并且**不跳过索引中的列**。

 索引顺序： `name`, `age`, `position`

`EXPLAIN SELECT * FROM employees WHERE age = 22 AND position ='manager'; `不走索引，因为跳过了name

`EXPLAIN SELECT * FROM employees WHERE age = 22 AND name = 'LiLei';`	内部优化，name,age走索引

##3.不在索引列上做任何操作（计算、函数、（自动or手动）类型转换），会导致索引失效而转

EXPLAIN SELECT * FROM employees WHERE name = 'LiLei';
向全表扫描: EXPLAIN SELECT * FROM employees WHERE left(name,3) = 'LiLei';

ex:查找当天日期

**hire_time 添加索引**

原来:没走索引

> EXPLAIN select * from employees where date(hire_time) ='2018-09-30';

优化:走索引

> EXPLAIN select * from employees where hire_time >='2018-09-30 00:00:00' and
> hire_time <='2018-09-30 23:59:59';

##4.存储引擎不能使用索引中范围条件右边的列

4.1**索引顺序： `name`, `age`, `position`**

>  EXPLAIN SELECT * FROM employees WHERE name= 'LiLei' AND age > 22 AND position ='manager';

不走position的索引：

![image-20210530153640209](02 Mysql索引(二).assets/image-20210530153640209.png)

**底层分析：**

age>22范围查找后，第三字段的值可能在最后，第三个值一 一比对等同于全表扫描

![image-20210525183101255](02 Mysql索引(二).assets/image-20210525183101255.png)

如果最后字段position在最后一个，

4.2**索引是 idx_name_age_position `name`, `position`, `age`**

> EXPLAIN SELECT * FROM employees WHERE name= 'LiLei' AND position ='manager' AND age > 22 ;

以上sql等于下面sql

> EXPLAIN SELECT * FROM employees WHERE name= 'LiLei' AND age > 22 AND position ='manager';

EXPLAIN SELECT * FROM employees WHERE name= 'LiLei' AND age = 22 AND position ='manager';

**走索引**

![image-20210525182428592](02 Mysql索引(二).assets/image-20210525182428592.png)

##5.尽量使用覆盖索引（只访问索引的查询（索引列包含查询列）），减少select *语句
> EXPLAIN SELECT name,age FROM employees WHERE name= 'LiLei' AND age = 23  AND  position ='manager';

推荐不写 select * 

##6.mysql在使用不等于（！=或者<>）的时候无法使用索引会导致全表扫描

> EXPLAIN SELECT * FROM employees WHERE name != 'LiLei';

##7.is null,is not null 也无法使用索引

> EXPLAIN SELECT * FROM employees WHERE name is null

为了使用索引则优化：建立字段的时候 NOT Null，设立默认值

##8.like以通配符开头（'$abc...'）mysql索引失效会变成全表扫描操作

**1.走索引**
`EXPLAIN SELECT * FROM employees WHERE name like 'Lei%'`

![image-20210525184607422](02 Mysql索引(二).assets/image-20210525184607422.png)

底层：因为底层字典有序来判断

![image-20210525184839144](02 Mysql索引(二).assets/image-20210525184839144.png)

**2.不走索引**

`EXPLAIN SELECT * FROM employees WHERE name like '%Lei'`

![image-20210525184642192](02 Mysql索引(二).assets/image-20210525184642192.png)

**3.问题：解决like'%字符串%'索引不被使用的方法？**
a）使用覆盖索引，查询**字段**必须是建立**覆盖索引字段**

`EXPLAIN SELECT name,age,position FROM employees WHERE name like '%Lei%';`

![image-20210525185059575](02 Mysql索引(二).assets/image-20210525185059575.png)

b）如果不能使用覆盖索引则可能需要借助搜索引擎

**与left的区别：**

mysql底层逻辑的优化：凡是有函数的都不走索引!!!

##9.字符串不加单引号索引失效
EXPLAIN SELECT * FROM employees WHERE name = '1000';

mysql会默认使用函数(int-->String),所以索引失效！

`EXPLAIN SELECT * FROM employees WHERE name = 1000;`

###10.少用or或in，用它查询时，mysql不一定使用索引，mysql内部优化器会根据检索比例、表大小等多个因素整体评估是否使用索引，详见范围查询优化

##11.范围查询优化

age加入索引：ALTER TABLE `employees`ADD INDEX `idx_age` (`age`) USING BTREE ;

`explain select * from employees where age >=1 and age <=2000;`

没走索引原因：mysql内部优化器会根据检索比例，表大小等多个因素整体评估是否使用索引

优化方法：可以讲大的范围拆分成多个小范围



索引使用的总结：

<img src="02 Mysql索引(二).assets/image-20210528181023561.png" alt="image-20210528181023561" style="zoom:150%;" />

