## MySQL 安装
首先使用了阿里云的`CentOS`服务器安装，没有成功。总是提示 `The server quit without updating PID file (/var/run/mysqld/xxxx.pid).`
后来改使用自己虚拟机的`CentOS`成功。
### 基本步骤
#### 卸载预先安装的 MySQL
<pre>[root@localhost bqq]# rpm -qa | grep -i mysql
<font color="#EF2929"><b>MySQL</b></font>-python-1.2.5-1.el7.x86_64
akonadi-<font color="#EF2929"><b>mysql</b></font>-1.9.2-4.el7.x86_64
perl-DBD-<font color="#EF2929"><b>MySQL</b></font>-4.023-6.el7.x86_64
qt-<font color="#EF2929"><b>mysql</b></font>-4.8.7-3.el7_6.x86_64
</pre>

接着卸载包含MySQL的程序。
<pre>[root@localhost bqq]# rpm -e MySQL-python-1.2.5-1.el7.x86_64
[root@localhost bqq]# rpm -e akonadi-mysql-1.9.2-4.el7.x86_64
[root@localhost bqq]# rpm -e perl-DBD-MySQL-4.023-6.el7.x86_64
</pre>

#### 下载对应的MySQL源，并安装
```shell
[root@localhost bqq]# wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
[root@localhost bqq]# sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
[root@localhost bqq]# sudo yum install mysql-server
```
如果报错，内容含有
``` shell
Error: Package: mysql-community-libs-5.6.35-2.el7.x86_64 (mysql56-community)
           Requires: libc.so.6(GLIBC_2.17)(64bit)
Error: Package: mysql-community-server-5.6.35-2.el7.x86_64 (mysql56-community)
           Requires: libc.so.6(GLIBC_2.17)(64bit)
Error: Package: mysql-community-server-5.6.35-2.el7.x86_64 (mysql56-community)
           Requires: systemd
Error: Package: mysql-community-server-5.6.35-2.el7.x86_64 (mysql56-community)
           Requires: libstdc++.so.6(GLIBCXX_3.4.15)(64bit)
Error: Package: mysql-community-client-5.6.35-2.el7.x86_64 (mysql56-community)
           Requires: libc.so.6(GLIBC_2.17)(64bit)
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```

解决：
``` shell
 yum install glibc.i686
 yum list libstdc++*
```
#### 启动 MySQL 服务
``` shell
service mysql start
service mysql stop
service mysql status
service mysql restart
```
#### 登录以及修改密码
``` sql
//mysql 安装完成之后, 会自动生成一个随机的密码, 并且保存在一个密码文件中 : /root/.mysql_secret
mysql -u root -p
//登录之后, 修改密码 :
set password = password('itcast');
//授权远程访问 :
grant all privileges on *.* to 'root' @'%' identified by 'itcast';
flush privileges;
```

```sql
//创建用户及修改密码

create user u1@localhost identified by '123';
set password for u1@localhost = password('1');

mysql -uu1 -p
```

```sql
//授权用户
GRANT privileges ON  databasename.tablename  TO  ‘username’@‘host’

//privileges：表示要授予什么权力，例如可以有 select ， insert ，delete，update等，如果要授予全部权力，则填 ALL

//databasename.tablename：表示用户的权限能用在哪个库的哪个表中，如果想要用户的权限很作用于所有的数据库所有的表，则填 *.*，*是一个通配符，表示全部。

//’username‘@‘host’：表示授权给哪个用户。
//撤销用户权限：

REVOKE   privileges   ON  database.tablename  FROM  ‘username’@‘host’；

//例如： REVOKE  SELECT ON  *.*  FROM  ‘zje’@‘%’；



但注意：

若授予权利是这样写： GRANT  SELECT  ON  *.*  TO ‘zje’@‘%’；

则用 REVOKE  SELECT ON   zje.aaa  TO  ‘zje’@‘%’；是不能撤销用户zje 对 zje.aaa 中的SELECT 权利的。


反过来 GRANT SELECT  ON  zje.aaa  TO  ‘zje’@‘%’；授予权力

用 REVOKE SELECT ON  *.*  FROM  ‘zje’@‘%’；也是不能用来撤销用户zje 对zje库的aaa表的SELECT 权利的

```