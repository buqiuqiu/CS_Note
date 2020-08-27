## 视图
### 视图概述
视图（View）是一种虚拟存在的表。视图并不在数据库中实际存在，行和列数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的。

通俗的讲，视图就是一条SELECT语句执行后返回的结果集。

所以我们在创建视图的时候，主要的工作就落在创建这条SQL查询语句上。
视图相对于普通的表的优势主要包括以下几项。

1. 简单：使用视图的用户完全不需要关心后面对应的表的结构、关联条件和筛选条件，对用户来说已经是过滤
好的复合条件的结果集。
2. 安全：使用视图的用户只能访问他们被允许查询的结果集，对表的权限管理并不能限制到某个行某个列，但
是通过视图就可以简单的实现。
3. 数据独立：一旦视图的结构确定了，可以屏蔽表结构变化对用户的影响，源表增加列对视图没有影响；源表
修改列名，则可以通过修改视图来解决，不会造成对访问者的影响。

### 创建或者修改视图

创建视图：
```sql
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
VIEW view_name [(column_list)]
AS select_statement
[WITH [CASCADED | LOCAL] CHECK OPTION]

```
修改视图：
```sql
ALTER [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE }]
VIEW view_name [(column_list)]
AS select_statement
[WITH [CASCADE | LOCAL] CHECK OPTION]
```

示例：
创建视图：
```sql
mysql> create or replace view city_country_view
    -> as
    -> select t.*,c.country_name from country c, city t where c.country_id = t.country_id;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from city_country_view;
+---------+-----------+------------+--------------+
| city_id | city_name | country_id | country_name |
+---------+-----------+------------+--------------+
|       1 | 西安      |          1 | China        |
|       2 | NewYork   |          2 | America      |
|       3 | 北京      |          1 | China        |
|       4 | 上海      |          1 | China        |
|       5 | 天津      |          1 | China        |
|       6 | 廊坊      |          1 | China        |
+---------+-----------+------------+--------------+
6 rows in set (0.00 sec)

```

### 查看视图
从 MySQL 5.1 版本开始，使用 SHOW TABLES 命令的时候不仅显示表的名字，同时也会显示视图的名字，而不存在单独显示视图的 SHOW VIEWS 命令。
同样，在使用 SHOW TABLE STATUS 命令的时候，不但可以显示表的信息，同时也可以显示视图的信息。
如果需要查询某个视图的定义，可以使用 SHOW CREATE VIEW 命令进行查看：

```sql
mysql> show tables;
+-------------------+
| Tables_in_demo_01 |
+-------------------+
| city              |
| city_country_view |
| country           |
+-------------------+
3 rows in set (0.00 sec)



mysql> show table status;
+-------------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+-----------------+----------+----------------+---------+
| Name              | Engine | Version | Row_format | Rows | Avg_row_length | Data_length | Max_data_length | Index_length | Data_free | Auto_increment | Create_time         | Update_time | Check_time | Collation       | Checksum | Create_options | Comment |
+-------------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+-----------------+----------+----------------+---------+
| city              | InnoDB |      10 | Compact    |    6 |           2730 |       16384 |               0 |        16384 |         0 |              7 | 2020-08-20 03:16:49 | NULL        | NULL       | utf8_general_ci |     NULL |                |         |
| city_country_view | NULL   |    NULL | NULL       | NULL |           NULL |        NULL |            NULL |         NULL |      NULL |           NULL | NULL                | NULL        | NULL       | NULL            |     NULL | NULL           | VIEW    |
| country           | InnoDB |      10 | Compact    |    3 |           5461 |       16384 |               0 |            0 |         0 |              5 | 2020-08-20 00:53:24 | NULL        | NULL       | utf8_general_ci |     NULL |                |         |
+-------------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+-----------------+----------+----------------+---------+
3 rows in set (0.00 sec)


mysql> show table status\G;
*************************** 1. row ***************************
           Name: city
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 6
 Avg_row_length: 2730
    Data_length: 16384
Max_data_length: 0
   Index_length: 16384
      Data_free: 0
 Auto_increment: 7
    Create_time: 2020-08-20 03:16:49
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options:
        Comment:
*************************** 2. row ***************************
           Name: city_country_view
         Engine: NULL
        Version: NULL
     Row_format: NULL
           Rows: NULL
 Avg_row_length: NULL
    Data_length: NULL
Max_data_length: NULL
   Index_length: NULL
      Data_free: NULL
 Auto_increment: NULL
    Create_time: NULL
    Update_time: NULL
     Check_time: NULL
      Collation: NULL
       Checksum: NULL
 Create_options: NULL
        Comment: VIEW
*************************** 3. row ***************************
           Name: country
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 3
 Avg_row_length: 5461
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 5
    Create_time: 2020-08-20 00:53:24
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options:
        Comment:
3 rows in set (0.00 sec)


mysql> show table status like 'city_country_view'\G;
*************************** 1. row ***************************
           Name: city_country_view
         Engine: NULL
        Version: NULL
     Row_format: NULL
           Rows: NULL
 Avg_row_length: NULL
    Data_length: NULL
Max_data_length: NULL
   Index_length: NULL
      Data_free: NULL
 Auto_increment: NULL
    Create_time: NULL
    Update_time: NULL
     Check_time: NULL
      Collation: NULL
       Checksum: NULL
 Create_options: NULL
        Comment: VIEW
1 row in set (0.00 sec)


mysql> show create view city_country_view \G;
*************************** 1. row ***************************
                View: city_country_view
         Create View: CREATE ALGORITHM=UNDEFINED DEFINER=`root`@`localhost` SQL SECURITY DEFINER VIEW `city_country_view` AS select `t`.`city_id` AS `city_id`,`t`.`city_name` AS `city_name`,`t`.`country_id` AS `country_id`,`c`.`country_name` AS `country_name` from (`country` `c` join `city` `t`) where (`c`.`country_id` = `t`.`country_id`)
character_set_client: utf8
collation_connection: utf8_general_ci
1 row in set (0.00 sec)

```

### 删除视图
语法：
```sql
DROP VIEW [IF EXISTS] view_name [,view_name]...[RESTRICT | CASCADE]

```
示例：删除视图city_country_view
```sql
DROP VIEW city_country_view;
```