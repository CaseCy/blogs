---
title: mysql的join语句
categories:
  - 数据库
date: 2020-02-21 18:58:12
tags: [数据库,mysq]
---

参考：

 https://segmentfault.com/a/1190000015572505 

 https://www.cnblogs.com/beginman/p/3754322.html 

 https://www.codeproject.com/Articles/33052/Visual-Representation-of-SQL-Joins

 https://www.cnblogs.com/beginman/p/3754322.html 

---

### Inner Join

![](/static/InnerJoin.png)

数据包含关系如上图，这种Join**会返回左边表和右边表满足匹配条件的所有的数据**，写法如下：

```sql
SELECT <select_list> 
FROM Table_A A
INNER JOIN Table_B B
ON A.Key = B.Key
```

### Left Join

![](/static/LeftJoin.png)

Left Join会返回左边表的所有数据和右边表中和左边表中匹配的数据，写法如下：

```sql
SELECT <select_list>
FROM Table_A A
LEFT JOIN Table_B B
ON A.Key = B.Key
```

### Right Join

![](/static/RightJoin.png)

Right Join是Left Join的对立面，Right Join会返回右边表的所有数据和左边表中和右边表中匹配的数据，写法如下：

```sql
SELECT <select_list>
FROM Table_A A
RIGHT JOIN Table_B B
ON A.Key = B.Key
```

### Outer Join

![](/static/OutterJoin.png)

也可以称为`FULL OUTER JOIN`或者`FULL JOIN`，这种方式会返回两张表中的所有数据，将两张表中匹配的数据连接起来，写法如下：

```sql
SELECT <select_list>
FROM Table_A A
FULL OUTER JOIN Table_B B
ON A.Key = B.Key
```

### Left Excluding Join

![](/static/LeftExcludingJoin.png)

这种方式会返回左边表中和右边表中不匹配的所有数据，写法如下：

```sql
SELECT <select_list> 
FROM Table_A A
LEFT JOIN Table_B B
ON A.Key = B.Key
WHERE B.Key IS NULL
```

### Right Excluding Join

![](/static/RightExcludingJoin.png)

这种方式会返回右边表中和左边表中不匹配的所有数据，写法如下：

```sql
SELECT <select_list>
FROM Table_A A
RIGHT JOIN Table_B B
ON A.Key = B.Key
WHERE A.Key IS NULL
```

### Outer Excluding JOIN

![](/static/OuterExcludingJoin.png)

这种方式会返回左右两张表中都不匹配的数据，写法如下：

```sql
SELECT <select_list>
FROM Table_A A
FULL OUTER JOIN Table_B B
ON A.Key = B.Key
WHERE A.Key IS NULL OR B.Key IS NULL
```

例子参考： https://www.codeproject.com/Articles/33052/Visual-Representation-of-SQL-Joins 

### Cross Join

得到的是两个表的乘积，即笛卡尔积

 **在 MySQL 中（仅限于 MySQL） CROSS JOIN 与 INNER JOIN 的表现是一样的** 在不指定 ON 条件得到的结果都是笛卡尔积，反之取得两个表完全匹配的结果。 