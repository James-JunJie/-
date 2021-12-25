## Limit

limit n, m; 

* n:偏移量
* m:返回记录行的最大数目

ex:

查找第三行：limit 2,1;

检索记录行 96-last: LIMIT 95,-1; 

LIMIT n 等价于 LIMIT 0,n。 

# join

```sql
FROM Websites
INNER JOIN access_log
ON Websites.id=access_log.site_id;
```

- **INNER JOIN**：如果表中有**至少一个匹配**，则返回行
- **LEFT JOIN**：即使右表中没有匹配，也从**左表返回所有的行**
- **RIGHT JOIN**：即使左表中没有匹配，也从**右表返回所有的行**
- **FULL JOIN**：只要其中**一个表中存在匹配**，则返回行

#HAVING 

在 SQL 中增加 HAVING 子句原因是，**WHERE 关键字无法与聚合函数**一起使用。

```sql
GROUP BY column_name
HAVING aggregate_function(column_name) operator value;
```

# Order by

ORDER BY *column_name*,*column_name* ASC|DESC;

ASC:顺序

#EXISTS 

```sql
WHERE EXISTS
(SELECT column_name FROM table_name WHERE condition);
```

只要后面select成立---返回true；

# sql命令行

连接数据库

```sql
mysql -u root -p
```

