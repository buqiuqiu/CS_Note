## MySql中常用工具
### mysql
该 mysql 不是指 MySQL 服务, 而是指 MySQL 的客户端工具
语法:
```sql
mysql [options][database]
```

#### 连接选项
```sql
参数:
    -u, --user=name         指定用户名
    -p, --password[=name]   指定密码
    -h, --host=name         指定服务器IP或域名
    -P, --port=#            指定连接端口

示例:
    mysql -h 127.0.0.1 -P 3306 -u root -p

    mysql -h127.0.0.1 -P3306 -uroot -p2143
```

#### 执行选项
```sql
参数:
    -e, --execute=name      执行SQL语句并退出

示例:
    mysql -uroot -p2143 db01 -e "select * from tb_book";
```
此选项可以在Mysql客户端执行SQL语句，而不用连接到MySQL数据库再执行，对于一些批处理脚本，这种方式尤其方便。
![](image/2020-08-22-13-38-36.png)

### mysqladmin
mysqladmin 是一个执行管理操作的客户端程序。可以用它来检查服务器的配置和当前状态、创建并删除数据库等。
可以通过 ： mysqladmin --help 指令查看帮助文档

![](image/2020-08-22-13-44-25.png)

```sql
mysqladmin -uroot -p2143 create 'test01';
mysqladmin -uroot -p2143 drop 'test01';
mysqladmin -uroot -p2143 version;
```
### mysqlbinlog

由于服务器生成的二进制日志文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到mysqlbinlog 日志管理工具。
语法 ：
```sql
mysqlbinlog [options] log-files1 log-files2 ...

选项：
    -d, --database=name : 指定数据库名称，只列出指定的数据库相关操作。
    -o, --offset=# : 忽略掉日志中的前n行命令。
    -r,--result-file=name : 将输出的文本格式日志输出到指定文件。
    -s, --short-form : 显示简单格式， 省略掉一些信息。
    --start-datatime=date1 --stop-datetime=date2 : 指定日期间隔内的所有日志。
    --start-position=pos1 --stop-position=pos2 : 指定位置间隔内的所有日志。
```

### mysqldump
语法:
```sql
mysqldump [options] db_name [tables]                #备份单个数据库或者库中部分数据表
mysqldump [options] --database/-B db1 [db2 db3...]  #备份指定的一个或者多个数据库
mysqldump [options] --all-databases/-A              #备份所有数据库
```

#### 连接选项

```sql
参数 ：
    -u, --user=name 指定用户名
    -p, --password[=name] 指定密码
    -h, --host=name 指定服务器IP或域名
    -P, --port=# 指定连接端口
```
#### 输出内容选项
```sql
参数：
    --add-drop-database     在每个数据库创建语句前加上 Drop database 语句
    --add-drop-table        在每个表创建语句前加上 Drop table 语句 , 默认开启 ; 不开启 (--skip-add-drop-table)
                           // 这两个选项可以在导入数据库的时候不用先手工删除旧的数据库，而是会自动删除，提高导入效率，但是导入前一定要做好备份并且确认旧数据库的确已经可以删除，否则误操作将会造成数据的损失。在默认情况下，这两个参数都自动加上。
    -n, --no-create-db      不包含数据库的创建语句
    -t, --no-create-info    不包含数据表的创建语句
    -d --no-data            不包含数据
    -T, --tab=name          自动生成两个文件：一个.sql文件，创建表结构的语句；一个.txt文件，数据文件，相当于select into outfile
示例 ：
    mysqldump -uroot -p2143 db01 tb_book --add-drop-database --add-drop-table > a
    mysqldump -uroot -p2143 -T /tmp test city
    
```
![](image/2020-08-22-14-28-12.png)

### mysqlimport/source
mysqlimport 是客户端数据导入工具，用来导入mysqldump 加 -T 参数后导出的文本文件。如果需要导入sql文件,可以使用mysql中的source 指令 :
```sql
mysqlimport [options] db_name textfile1 [textfile2...]
mysqlimport -uroot -p2143 test /tmp/city.txt

source /root/tb_book.sql
```

### mysqlshow
mysqlshow 客户端对象查找工具，用来很快地查找存在哪些数据库、数据库中的表、表中的列或者索引。

```sql
mysqlshow [options] [db_name [table_name [col_name]]]

参数:
    --count 显示数据库及表的统计信息（数据库，表 均可以不指定）
    -i 显示指定数据库或者指定表的状态信息

#查询每个数据库的表的数量及表中记录的数量
mysqlshow -uroot -p2143 --count
#查询test库中每个表中的字段书，及行数
mysqlshow -uroot -p2143 test --count
#查询test库中book表的详细情况
mysqlshow -uroot -p2143 test book --count
```
