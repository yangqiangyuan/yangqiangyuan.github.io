---
title: mysql导出select语句结果
date: 2022-12-16 16:13:00
tags: mysql
---

# mysql导出select查询结果



<!--more-->

```
方法一：

mysql -hxx -uxx -pxx -e "query statement" db > file

　　-h：后面跟的是链接的host（主机）

　　-u:后面跟的是用户名

　　-p:后面跟的是密码

　　db:你要查询的数据库

　　file:你要写入的文件，绝对路径

例如：
mysql -h127.0.0.1 -uroot -p000000 -e"select * from a" test > 1.txt
```

```
方法二：into outfile
mysql -hxxx -uxx -pxx 
select * from table into outfile 'xxx.txt';

问题：使用into outfile报错：ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement


解决办法：先执行命令show variables like '%secure%';(查询数据库默认导出路径)
mysql> show variables like '%secure%';
+--------------------------+-----------------------+
| Variable_name            | Value                 |
+--------------------------+-----------------------+
| require_secure_transport | OFF                   |
| secure_auth              | ON                    |
| secure_file_priv         | /var/lib/mysql-files/ |
+--------------------------+-----------------------+
3 rows in set (0.00 sec)


执行into outfile导出时加上secure_file_priv的路径
例如：
mysql -hxxx -uxx -pxx 
select * from table into outfile '/var/lib/mysql-files/xxx.txt';
命令执行成功后就会在/var/lib/mysql-files/目录下生成导出的文件。
```

