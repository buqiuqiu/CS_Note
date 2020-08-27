## 存储过程和函数
### 概述
存储过程和函数是 事先经过编译并存储在数据库中的一段 SQL 语句的集合，调用存储过程和函数可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。

存储过程和函数的区别在于函数必须有返回值，而存储过程没有。
函数 ： 是一个有返回值的过程 ；
过程 ： 是一个没有返回值的函数 ；

### 创建存储过程
语法：
```sql
CREATE PROCEDURE procedure_name ([proc_parameter[,...]])
begin
    ---SQL语句
end;
```

示例：
```sql
delimiter $

create procedure pro_test1()
begin
    select 'Hello';
end$

delimiter ;
```

tips:
DELIMITER
该关键字用来声明SQL语句的分隔符 , 告诉 MySQL 解释器，该段命令是否已经结束了，mysql是否可以执行了。
默认情况下，delimiter是分号;。在命令行客户端中，如果有一行命令以分号结束，那么回车后，mysql将会执行该命令。

### 调用存储过程

语法：
```sql
call procedure_name();
```

示例：
```sql
mysql> call pro_test1();
+-------+
| Hello |
+-------+
| Hello |
+-------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)
;
```
### 查看存储过程

```sql
-- 查询db_name数据库中的所有的存储过程
select name from mysql.proc where db='db_name';
-- 查询存储过程的状态信息
show procedure status;
-- 查询某个存储过程的定义
show create procedure test.pro_test1 \G;
```

```sql
mysql> show procedure status\G;
*************************** 1. row ***************************
                  Db: demo_01
                Name: pro_test1
                Type: PROCEDURE
             Definer: root@localhost
            Modified: 2020-08-20 05:31:26
             Created: 2020-08-20 05:31:26
       Security_type: DEFINER
             Comment:
character_set_client: utf8
collation_connection: utf8_general_ci
  Database Collation: utf8mb4_general_ci
1 row in set (0.00 sec)


mysql> show create procedure pro_test1\G;
*************************** 1. row ***************************
           Procedure: pro_test1
            sql_mode: STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
    Create Procedure: CREATE DEFINER=`root`@`localhost` PROCEDURE `pro_test1`()
begin
select 'Hello';
end
character_set_client: utf8
collation_connection: utf8_general_ci
  Database Collation: utf8mb4_general_ci
1 row in set (0.00 sec)


mysql> show create procedure demo_01.pro_test1\G;
*************************** 1. row ***************************
           Procedure: pro_test1
            sql_mode: STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
    Create Procedure: CREATE DEFINER=`root`@`localhost` PROCEDURE `pro_test1`()
begin
select 'Hello';
end
character_set_client: utf8
collation_connection: utf8_general_ci
  Database Collation: utf8mb4_general_ci
1 row in set (0.00 sec)
```

### 删除存储过程
```sql
DROP PROCEDURE [IF EXISTS] sp_name ;
```
```sql
mysql> drop procedure demo_01.pro_test1;
Query OK, 0 rows affected (0.00 sec)

mysql> show procedure status;
Empty set (0.00 sec)

```

### 语法
#### 变量
1. DECLARE
   通过 DECLARE 可以定义一个局部变量，该变量的作用范围只能在 BEGIN…END 块中。
   语法：
   ```sql
   DECLARE var_name[,...] type [DEFAULT value]

   ```
   
   示例：
   ```sql
    mysql> delimiter $
    mysql>
    mysql>    create procedure pro_test2()
        ->    begin
        ->    declare num int default 5;
        ->    select num+10;
        ->    end $
    Query OK, 0 rows affected (0.00 sec)

    mysql>
    mysql>    delimiter ;
    mysql> call pro_test2();
    +--------+
    | num+10 |
    +--------+
    |     15 |
    +--------+
    1 row in set (0.00 sec)

    Query OK, 0 rows affected (0.00 sec);
   ```
2. SET
   直接赋值使用 SET，可以赋常量或者赋表达式，具体语法如下：
   ```sql
    SET var_name = expr [, var_name = expr] ...
   ```
   示例：
   ```sql
    mysql> DELIMITER $
    mysql> CREATE PROCEDURE pro_test3()
        -> BEGIN
        -> DECLARE NAME VARCHAR(20);
        -> SET NAME = 'MYSQL';
        -> SELECT NAME ;
        -> END$
    Query OK, 0 rows affected (0.01 sec)

    mysql> DELIMITER ;
    mysql> call pro_test3();
    +-------+
    | NAME  |
    +-------+
    | MYSQL |
    +-------+
    1 row in set (0.00 sec)

    Query OK, 0 rows affected (0.00 sec)

   ```

   也可以通过select ... into 方式进行赋值操作 :
   ```sql
    mysql> DELIMITER $
    mysql> CREATE PROCEDURE pro_test5()
        -> BEGIN
        -> declare countnum int;
        -> select count(*) into countnum from city;
        -> select countnum;
        -> END$
    Query OK, 0 rows affected (0.00 sec)

    mysql> DELIMITER ;
    mysql> call pro_test5();
    +----------+
    | countnum |
    +----------+
    |        6 |
    +----------+
    1 row in set (0.00 sec)

    Query OK, 0 rows affected (0.00 sec)

   ```
#### if 条件判断
语法：
```sql
if search_condition then statement_list
[elseif search_condition then statement_list] ...
[else statement_list]
end if;
```
需求：
根据定义的身高变量，判定当前身高的所属的身材类型
180 及以上 ----------> 身材高挑
170 - 180 ---------> 标准身材
170 以下 ----------> 一般身材

示例：
```sql
mysql> delimiter $
mysql> create procedure pro_test6()
    -> begin
    -> declare height int default 175;
    -> declare description varchar(50);
    -> if height >= 180 then
    -> set description = '身材高挑';
    -> elseif height >= 170 and height < 180 then
    -> set description = '标准身材';
    -> else
    -> set description = '一般身材';
    -> end if;
    -> select description ;
    -> end$
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter ;
mysql> call pro_test6();
+--------------+
| description  |
+--------------+
| 标准身材     |
+--------------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)
```
#### 传递参数
语法格式：
```sql
create procedure procedure_name([in/out/inout] 参数名 参数类型)
...
IN : 该参数可以作为输入，也就是需要调用方传入值 , 默认
OUT : 该参数作为输出，也就是该参数可以作为返回值
INOUT : 既可以作为输入参数，也可以作为输出参数
```

```sql
mysql> delimiter $
mysql> create procedure pro_test7(in height int)
    -> begin
    -> declare description varchar(50) default '';
    -> if height >= 180 then
    -> set description='身材高挑';
    -> elseif height >= 170 and height < 180 then
    -> set description='标准身材';
    -> else
    -> set description='一般身材';
    -> end if;
    -> select concat('身高 ', height , '对应的身材类型为:',description);
    -> end$
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter ;
mysql> 
mysql> call pro_test7(168);
+---------------------------------------------------------------------+
| concat('身高 ', height , '对应的身材类型为:',description)           |
+---------------------------------------------------------------------+
| 身高 168对应的身材类型为:一般身材                                   |
+---------------------------------------------------------------------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

```

```sql
delimiter $
create procedure pro_test8(in height int , out description varchar(100))
begin
if height >= 180 then
set description='身材高挑';
elseif height >= 170 and height < 180 then
set description='标准身材';
else
set description='一般身材';
end if;
end$
delimiter ;
```
```sql
mysql> delimiter $
mysql> create procedure pro_test8(in height int , out description varchar(100))
    -> begin
    -> if height >= 180 then
    -> set description='身材高挑';
    -> elseif height >= 170 and height < 180 then
    -> set description='标准身材';
    -> else
    -> set description='一般身材';
    -> end if;
    -> end$
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter ;
mysql> 
mysql> call pro_test8(168,@description);
Query OK, 0 rows affected (0.00 sec)

mysql> select @description;
+--------------+
| @description |
+--------------+
| 一般身材     |
+--------------+
1 row in set (0.00 sec)

```
> tips
> @description : 这种变量要在变量名称前面加上“@”符号，叫做用户会话变量，代表整个会话过程他都是有作用的，这个类似于全局变量一样。
> @@global.sort_buffer_size : 这种在变量前加上 "@@" 符号, 叫做 系统变量
#### case 结构
语法结构：
```sql
方式一 :
CASE case_value
WHEN when_value THEN statement_list
[WHEN when_value THEN statement_list] ...
[ELSE statement_list]
END CASE;
方式二 :
CASE
WHEN search_condition THEN statement_list
[WHEN search_condition THEN statement_list] ...
[ELSE statement_list]
END CASE;
```
示例：
```sql
mysql> delimiter $
mysql> create procedure pro_test9(month int)
    -> begin
    -> declare result varchar(20);
    -> case
    -> when month >= 1 and month <=3 then
    -> set result = '第一季度';
    -> when month >= 4 and month <=6 then
    -> set result = '第二季度';
    -> when month >= 7 and month <=9 then
    -> set result = '第三季度';
    -> when month >= 10 and month <=12 then
    -> set result = '第四季度';
    -> end case;
    ->
    -> select concat('您输入的月份为 :', month , ' , 该月份为 : ' , result) as content ;
    -> end$
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter ;
mysql> call pro_test9(9);
+--------------------------------------------------------+
| content                                                |
+--------------------------------------------------------+
| 您输入的月份为 :9 , 该月份为 : 第三季度                |
+--------------------------------------------------------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

```
#### while 循环
语法结构：
```sql
while search_condition do
statement_list
end while;
```
示例：计算从1加到n的值
```sql
mysql> delimiter $
mysql>
mysql> create procedure pro_test10(n int)
    -> begin
    -> declare total int default 0;
    -> declare num int default 1;
    -> while num<=n do
    ->     set total = total + num;
    ->     set num = num + 1;
    -> end while;
    -> select total;
    ->
    -> end $
Query OK, 0 rows affected (0.00 sec)

mysql>
mysql> delimiter ;
mysql> call pro_test10(100);
+-------+
| total |
+-------+
|  5050 |
+-------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

```
#### repeat 结构
有条件的循环控制语句, 当满足条件的时候退出循环 。while 是满足条件才执行，repeat 是满足条件就退出循环。
语法结构 :
```sql
REPEAT
    statement_list
    UNTIL search_condition
END REPEAT;
```
示例：
```sql
delimiter $
create procedure pro_test11(n int)
begin
declare total int default 0;
repeat
set total = total + n;
set n = n - 1;
until n=0
end repeat;
select total ;
end$
delimiter ;
```
#### loop 语句
LOOP 实现简单的循环，退出循环的条件需要使用其他的语句定义，通常可以使用 LEAVE 语句实现，具体语法如
下：
```sql
[begin_label:] LOOP
statement_list
END LOOP [end_label]
```
如果不在 statement_list 中增加退出循环的语句，那么 LOOP 语句可以用来实现简单的死循环。
#### leave 语句
用来从标注的流程构造中退出，通常和 BEGIN ... END 或者循环一起使用。下面是一个使用 LOOP 和 LEAVE 的简
单例子 , 退出循环：
```sql
delimiter $
CREATE PROCEDURE pro_test11(n int)
BEGIN
declare total int default 0;
ins: LOOP
    IF n <= 0 then
        leave ins;
    END IF;
    set total = total + n;
    set n = n - 1;
END LOOP ins;
    select total;
END$
delimiter ;
```
#### 游标/光标
游标是用来存储查询结果集的数据类型 , 在存储过程和函数中可以使用光标对结果集进行循环的处理。光标的使用包括光标的声明、OPEN、FETCH 和 CLOSE，其语法分别如下:
DECLARE:
```SQL
DECLARE cursor_name CURSOR FOR select_statement;
```
OPEN:
```SQL
OPEN cursor_name;
```
FETCH:
```SQL
FETCH cursor_name INTO var_name [,var_name]...
```
CLOSE
```SQL
CLOSE cursor_name;
```
示例：
初始化：
```sql
delimiter $
create table emp(
    id int(11) not null auto_increment ,
    name varchar(50) not null comment '姓名',
    age int(11) comment '年龄',
    salary int(11) comment '薪水',
    primary key(`id`)
)engine=innodb default charset=utf8 ;
insert into emp(id,name,age,salary) values(null,'金毛狮王',55,3800),(null,'白眉鹰王',60,4000),(null,'青翼蝠王',38,2800),(null,'紫衫龙王',42,1800);

end $
delimiter ;
```
-- 查询emp表中数据, 并逐行获取进行展示
```sql
delimiter $
create procedure pro_test12()
begin
    declare e_id int(11);
    declare e_name varchar(50);
    declare e_age int(11);
    declare e_salary int(11);
    declare emp_result cursor for select * from emp;
    open emp_result;
    fetch emp_result into e_id,e_name,e_age,e_salary;
    select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为:
    ',e_salary);
    fetch emp_result into e_id,e_name,e_age,e_salary;
    select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为:
    ',e_salary);
    fetch emp_result into e_id,e_name,e_age,e_salary;
    select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为:
    ',e_salary);
    fetch emp_result into e_id,e_name,e_age,e_salary;
    select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为:
    ',e_salary);
    fetch emp_result into e_id,e_name,e_age,e_salary;
    select concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为:
    ',e_salary);
    close emp_result;
end$
delimiter ;

mysql> call pro_test12();
+---------------------------------------------------------------------------------------+
| concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为:

    ',e_salary)    |
+---------------------------------------------------------------------------------------+
| id=1, name=金毛狮王, age=55, 薪资为:

    3800                                        |
+---------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

+---------------------------------------------------------------------------------------+
| concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为:

    ',e_salary)    |
+---------------------------------------------------------------------------------------+
| id=2, name=白眉鹰王, age=60, 薪资为:

    4000                                        |
+---------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

+---------------------------------------------------------------------------------------+
| concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为:

    ',e_salary)    |
+---------------------------------------------------------------------------------------+
| id=3, name=青翼蝠王, age=38, 薪资为:

    2800                                        |
+---------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

+---------------------------------------------------------------------------------------+
| concat('id=',e_id , ', name=',e_name, ', age=', e_age, ', 薪资为:

    ',e_salary)    |
+---------------------------------------------------------------------------------------+
| id=4, name=紫衫龙王, age=42, 薪资为:

    1800                                        |
+---------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

```

通过循环结构 , 获取游标中的数据 :
```sql
DELIMITER $
create procedure pro_test13()
begin
    DECLARE id int(11);
    DECLARE name varchar(50);
    DECLARE age int(11);
    DECLARE salary int(11);
    DECLARE has_data int default 1;
    DECLARE emp_result CURSOR FOR select * from emp;
    DECLARE EXIT HANDLER FOR NOT FOUND set has_data = 0;
    open emp_result;
    repeat
    fetch emp_result into id , name , age , salary;
    select concat('id为',id, ', name 为' ,name , ', age为 ' ,age , ', 薪水为: ',
    salary);
    until has_data = 0
    end repeat;
    close emp_result;
end$
DELIMITER ;mysql> call pro_test13();
+------------------------------------------------------------------------------------------+
| concat('id为',id, ', name 为' ,name , ', age为 ' ,age , ', 薪水为: ',

    salary)       |
+------------------------------------------------------------------------------------------+
| id为1, name 为金毛狮王, age为 55, 薪水为: 3800                                           |
+------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

+------------------------------------------------------------------------------------------+
| concat('id为',id, ', name 为' ,name , ', age为 ' ,age , ', 薪水为: ',

    salary)       |
+------------------------------------------------------------------------------------------+
| id为2, name 为白眉鹰王, age为 60, 薪水为: 4000                                           |
+------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

+------------------------------------------------------------------------------------------+
| concat('id为',id, ', name 为' ,name , ', age为 ' ,age , ', 薪水为: ',

    salary)       |
+------------------------------------------------------------------------------------------+
| id为3, name 为青翼蝠王, age为 38, 薪水为: 2800                                           |
+------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

+------------------------------------------------------------------------------------------+
| concat('id为',id, ', name 为' ,name , ', age为 ' ,age , ', 薪水为: ',

    salary)       |
+------------------------------------------------------------------------------------------+
| id为4, name 为紫衫龙王, age为 42, 薪水为: 1800                                           |
+------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)


```
### 存储函数
语法结构：
```sql
CREATE FUNCTION function_name([param type ... ])
RETURNS type
BEGIN
...
END;
```
案例：定义一个存储过程，请求满足条件的总记录数 
```sql
mysql> delimiter $
mysql> create function count_city(countryId int)
    -> returns int
    -> begin
    ->     declare cnum int ;
    ->     select count(*) into cnum from city where country_id = countryId;
    -> return cnum;
    -> end$
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter ;


调用：
mysql> select count_city(1);
+---------------+
| count_city(1) |
+---------------+
|             5 |
+---------------+
1 row in set (0.00 sec)

mysql> select count_city(2);
+---------------+
| count_city(2) |
+---------------+
|             1 |
+---------------+
1 row in set (0.00 sec)
;
```

