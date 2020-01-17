---
title: " mysql存储过程\t\t"
tags:
  - mysql
url: 422.html
id: 422
comments: false
categories:
  - mysql
date: 2019-08-23 15:09:15
---

参考： [Mysql存储过程](https://www.cnblogs.com/l5580/p/5993238.html) [DECLARE ... HANDLER](http://imysql.com/mysql-refman/5.7/sql-syntax.html#declare-handler)

存储过程的创建
-------

```sql
CREATE PROCEDURE 存储过程名()
```


存储过程的删除
-------

```sql
-- 删除存储过程后面不需要跟(),只给出存储过程名
DROP PROCEDURE productpricing;     
```


### 删除时先判断是否存在

```sql
DROP PROCEDURE IF EXISTS productpricing
```


Mysql命令行客户机的分隔符
---------------

默认的MySQL语句分隔符为分号;作为语句分隔符。如果命令行实用程序要解释存储过程自身的 ; 字符，则他们最终不会成为存储过程的成分，这会使存储过程中的SQL出现句法错误，解决方法是临时更改命令实用程序的语句分隔符，使用DELIMITER

```sql
-- 定义新的语句分隔符为//
DELIMITER //    

CREATE PROCEDURE productpricing()
-- 表示开始存储过程
BEGIN

SELECT Avg(prod_price) AS priceaverage

FROM products;
-- 表示存储过程结束
END //
-- 改回原来的语句分隔符为;
DELIMITER ;    
-- 除\符号外，任何字符都可以作为语句分隔符
```


参数
--

### 入参

创建存储过程时指定

```sql
-- 每个参数都要指定类型
CREATE PROCEDURE productpricing(
    IN param1 VARCHAR(50),
    IN param2 Integer(11)
)
```


### 出参（返回值）

```sql
-- 每个参数都要指定类型 DEFAULT用于指定默认值
CREATE PROCEDURE productpricing(
    IN param1 VARCHAR(50) DEFAULT "",
    IN param2 INT(11),
    OUT param3 DECIMAL(8,2)
    OUT param4 TIMESTAMP
)
```


#### 调用

```sql
-- 这里调用时指定了出参的变量，前面要加@
CALL productpricing("调用入参",11,@outParam1,@outParam2);
select @outParam1,@outParam2
```


### 局部变量

```sql
-- 定义局部变量，局部变量声明要放在存储过程开始的地方
DECLARE x VARCHAR(11);
```


### 查询赋值

```sql
DECLARE _roleId INT(11);
DECLARE _roleName VARCHAR(20);
-- 可以使用INTO将查询语句中的列结果赋值给变量
SELECT role_id,role_name
INTO _roleId,_roleName
FROM
sys_role
```


### 设置变量的值

```sql
SET x = 5;
```


流程控制语句
------

### IF

```sql
-- IF
IF x = 5 THEN
SELECT * FROM sys_role;
-- ELSEIF
ELSEIF x = 6 THEN
SELECT id FROM sys_role;
-- ELSE 这里不用写THEN
ELSE
SELECT role_name FROM sys_role;
-- IF执行之后一定要END IF，这里有;
END IF;
```


### Case语句

```sql
CASE [case值]
WHEN [条件] THEN [执行的内容]
WHEN [条件] THEN [执行的内容] 
ELSE [执行的内容]
End CASE
```


游标
--

游标（cursor）是一个存储在MYSQL服务器上的数据库查询，它不是一条SELECT语句，而是被该语句检索出来的结果集。在存储了游标之后，应用程序可以根据需要滚动或浏览其中的数据。游标主要用于交互式应用，其中用户需要滚动屏幕上的数据，并对数据进行浏览或做出更改。 **使用游标** 使用游标涉及几个明确的步骤：

1.  在能够使用游标前，必须声明（定义）它，这个过程实际上没有检索数据，它只是定义要使用的SELECT语句
    
2.  一旦声明后，必须打开游标以供使用。这个过程用钱吗定义的SELECT语句吧数据实际检索出来
    
3.  对于填有数据的游标，根据需要取出（检索）的各行
    
4.  在接受游标使用时，必须关闭它 **如果不明确关闭游标，MySQL将会在到达END语句时自动关闭它**
    

### 创建游标

游标可用DECLARE 语句创建。 DECLARE命名游标，并定义相应的SELECT语句。

```sql
DECLARE [游标名称] CURSOR FOR [查询语句];
-- 例子
DECLARE cur CURSOR FOR SELECT role_id FROM sys_role;
```


### 打开和关闭游标

```sql
OPEN [游标名称];
-- CLOSE释放游标使用的所有内部内存和资源，因此，每个游标不需要时都应该关闭
CLOSE [游标名称];
```


### 使用游标数据

在一个游标被打开后，可以使用FETCH语句分别访问它的每一行。FETCH指定检索什么数据（所需的要列），检索出来的数据存储在什么地方。它还向前移动游标中的内部行指针，使下一条FETCH语句检索下一行从第一行到最后一行

```sql
-- 每次FETCH取的是一行的数据
FETCH cura INTO userId,roleId,roleName;
```


### 循环

有几种循环游标的方式

*   WHILE \[循环条件\] DO \[循环体\] END WHILE
*   REPEAT \[循环体\] UNTIL \[结束标志\] END REPEAT;
*   loop名称:LOOP ...... END LOOP; 使用 LEAVE \[loop名称\] 可以 离开LOOP循环

定义循环结束的标志

```sql
DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET [变量]=TRUE;
-- 或者下面的也可以
DECLARE CONTINUE HANDLER FOR NOT FOUND SET [变量] = TRUE;
```


关于MYSQL的DECLARE...HANDLER用法可以参考 [DECLARE ... HANDLER](http://imysql.com/mysql-refman/5.7/sql-syntax.html#declare-handler)

比较完整的例子
-------

```sql
-- 如果存在就删除
DROP PROCEDURE IF EXISTS testPro;
-- 定义分隔符
DELIMITER //
-- 创建存储过程
CREATE PROCEDURE testPro(
    IN inParam Int(11),
    OUT outParam VARCHAR(20)
)
BEGIN
    -- 定义局部变量
    DECLARE _roleName VARCHAR(20);
    DECLARE done BIT DEFAULT FALSE;
    DECLARE roleName VARCHAR(50);
    DECLARE roleId INT(11);
    DECLARE userId INT(11);
    -- 定义一个游标
    DECLARE cura CURSOR FOR SELECT id,role_id,role_name FROM sys_user;
    DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=TRUE;
    -- 也可以这样写DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    -- 查询表
    SELECT role_name
    -- 将查询的结果赋值给_roleName
    INTO _roleName
    FROM sys_role WHERE role_id = inParam;
    -- 设置outParam的值
    IF _roleName = "ADMIN" THEN
        SET outParam = "管理员";
    ELSEIF _roleName = "EMPLOYEE" THEN
        SET outParam = "员工";
    ELSE
        SET outParam = "未知";
    END IF;
    -- 打开游标
    OPEN cura;
        -- 循环
        REPEAT
            FETCH cura INTO userId,roleId,roleName;
            IF roleId IS NOT NULL THEN
                UPDATE sys_user SET role_name = (SELECT role_name FROM sys_role WHERE role_id = roleId) WHERE id = userId;
            END IF;
        UNTIL done END REPEAT;
    CLOSE cura;
END //
DELIMITER ;
-- 调用
CALL testPro(2,@testOut);
SELECT @testOut;
```