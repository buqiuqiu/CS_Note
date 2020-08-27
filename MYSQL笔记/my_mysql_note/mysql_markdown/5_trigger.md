## 触发器
### 介绍
触发器是与表有关的数据库对象，指在 insert/update/delete 之前或之后，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性 , 日志记录 , 数据校验等操作 。
使用别名 OLD 和 NEW 来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。

**现在触发器还只支持行级触发，不支持语句级触发。**

|触发器类型|NEW 和 OLD的使用|
|---|---|
|INSERT 型触发器|NEW 表示将要或者已经新增的数据|
|UPDATE 型触发器|OLD 表示修改之前的数据 , NEW 表示将要或已经修改后的数据|
|DELETE 型触发器|OLD 表示将要或者已经删除的数据|


### 创建触发器

语法结构：
```sql
CREATE TRIGGER trigger_name
BEFORE/AFTER insert/update/delete
ON tbl_name
[for each row] --行触发器
begin
    srigger_stmt;
end;
```
示例：通过触发器记录emp表的数据变更日志，包含增加，修改，删除；首先创建一张日志表。
```sql
mysql> create table emp_logs(
    -> id int(11) not null auto_increment,
    -> operation varchar(20) not null comment '操作类型, insert/update/delete',
    -> operate_time datetime not null comment '操作时间',
    -> operate_id int(11) not null comment '操作表的ID',
    -> operate_params varchar(500) comment '操作参数',
    -> primary key(`id`)
    -> )engine=innodb default charset=utf8;
Query OK, 0 rows affected (0.01 sec)

```
创建 insert 型触发器，完成插入数据时的日志记录 :
```sql
DELIMITER $

CREATE TRIGGER emp_logs_insert_trigger
after insert 
on emp
for each row
begin   
    insert into emp_logs (id, operation,operate_time,operate_id,operate_params)
    values (null,'insert',now(),new.id,concat('插入后（id:',new.id,',name:',new.name,', age:',new.age,', salary:',new.salary,')'));
end $

DELIMITER ;
```

创建 update 型触发器，完成更新数据时的日志记录 :
```sql
DELIMITER $
create trigger emp_logs_update_trigger
after update
on emp
for each row
begin
    insert into emp_logs (id,operation,operate_time,operate_id,operate_params)
    values(null,'update',now(),new.id,concat('修改前(id:',old.id,', name:',old.name,',
    age:',old.age,', salary:',old.salary,') , 修改后(id',new.id, 'name:',new.name,',
    age:',new.age,', salary:',new.salary,')'));
end $
DELIMITER ;
```
创建delete 行的触发器 , 完成删除数据时的日志记录 :
```sql
DELIMITER $
create trigger emp_logs_delete_trigger
after delete
on emp
for each row
begin
    insert into emp_logs (id,operation,operate_time,operate_id,operate_params)
    values(null,'delete',now(),old.id,concat('删除前(id:',old.id,', name:',old.name,',
    age:',old.age,', salary:',old.salary,')'));
end $
DELIMITER ;
```

测试：
```sql
mysql> insert into emp(id,name,age,salary) values(null, '光明左使',30,3500);
Query OK, 1 row affected (0.00 sec)

mysql> insert into emp(id,name,age,salary) values(null, '光明右使',33,3200);
delete from emp where id = 5;Query OK, 1 row affected (0.00 sec)

mysql> update emp set age = 39 where id = 3;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> delete from emp where id = 5;
Query OK, 1 row affected (0.00 sec)


mysql> select * from emp_logs\G;
*************************** 1. row ***************************
            id: 1
     operation: insert
  operate_time: 2020-08-20 11:16:48
    operate_id: 5
operate_params: 插入后（id:5,name:光明左使, age:30, salary:3500)
*************************** 2. row ***************************
            id: 2
     operation: insert
  operate_time: 2020-08-20 11:16:48
    operate_id: 6
operate_params: 插入后（id:6,name:光明右使, age:33, salary:3200)
*************************** 3. row ***************************
            id: 3
     operation: update
  operate_time: 2020-08-20 11:16:48
    operate_id: 3
operate_params: 修改前(id:3, name:青翼蝠王,

    age:38, salary:2800) , 修改后(id3name:青翼蝠王,

    age:39, salary:2800)
*************************** 4. row ***************************
            id: 4
     operation: delete
  operate_time: 2020-08-20 11:17:11
    operate_id: 5
operate_params: 删除前(id:5, name:光明左使,

    age:30, salary:3500)
4 rows in set (0.00 sec)

```


### 查看触发器
```sql
show triggers;
```

### 删除触发器
```sql
drop trigger [schema_name.]trigger_name;
```

```sql
mysql> drop trigger demo_01.emp_logs_delete_trigger;
Query OK, 0 rows affected (0.00 sec)
//如果没有指定 schema_name，默认为当前数据库 。
mysql> show triggers\G;
*************************** 1. row ***************************
             Trigger: emp_logs_insert_trigger
               Event: INSERT
               Table: emp
           Statement: begin
    insert into emp_logs (id, operation,operate_time,operate_id,operate_params)
    values (null,'insert',now(),new.id,concat('插入后（id:',new.id,',name:',new.name,', age:',new.age,', salary:',new.salary,')'));

end
              Timing: AFTER
             Created: NULL
            sql_mode: STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
             Definer: root@localhost
character_set_client: utf8
collation_connection: utf8_general_ci
  Database Collation: utf8mb4_general_ci
*************************** 2. row ***************************
             Trigger: emp_logs_update_trigger
               Event: UPDATE
               Table: emp
           Statement: begin
    insert into emp_logs (id,operation,operate_time,operate_id,operate_params)
    values(null,'update',now(),new.id,concat('修改前(id:',old.id,', name:',old.name,',

    age:',old.age,', salary:',old.salary,') , 修改后(id',new.id, 'name:',new.name,',

    age:',new.age,', salary:',new.salary,')'));
end
              Timing: AFTER
             Created: NULL
            sql_mode: STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
             Definer: root@localhost
character_set_client: utf8
collation_connection: utf8_general_ci
  Database Collation: utf8mb4_general_ci
2 rows in set (0.01 sec)

```