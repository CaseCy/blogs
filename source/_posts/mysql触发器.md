---
title: " Mysql触发器\t\t"
tags:
  - mysql
url: 127.html
id: 127
comments: false
categories:
  - mysql
date: 2018-10-19 09:49:37
---
```sql
DROP TRIGGER IF EXISTS `before_insert_log`;
CREATE TRIGGER `before_insert_log` 
BEFORE INSERT ON `cc_log`
FOR EACH ROW BEGIN
	SELECT a,b,c
  INTO @_a,@_b,@_c
	FROM abc aa
	WHERE aa.d = new.e;
	SET new.e=@_e,new.f=@_f,new.g=@_g;
END
```
触发器可以设置给某表插入数据之前或者之后，

```sql
BEFORE INSERT ON `cc_log`
```
由before或者after决定，触发器中的变量不用事先声明，
注:每一个sql都必须结尾;
##给单个变量设置值
set a = (可以是单个结果的sql，也可以直接是个值)
##给多个变量设置值
如：

```sql
  SELECT a,b,c
  INTO @_a,@_b,@_c
```
用into来给多个变量赋值，变量前最好加个@，两边不能加括号
## 其他的语法

```sql
IF a > 0 THEN b=1
```

注：有IF开头，必有END IF结尾
