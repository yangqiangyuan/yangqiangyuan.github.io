---
title: postgresql常用命令
date: 2022-10-08 14:31:21
tags: postgres
---

# postgresql常用命令



<!--more-->

```perl
1.修改密码
postgres=# ALTER ROLE postgres WITH PASSWORD 'Ftlp_2077';

2.查看已经存在的数据库
postgres=# \l

3.查看表
postgres=# \dt

4.查看表结构
postgres=# \d tablename

5.使用数据库
postgres=# \c databasename

6.查看库大小
postgres=# select datname, pg_size_pretty (pg_database_size(datname)) AS size from pg_database;

7.查看表大小
postgres=# \c corehr
You are now connected to database "corehr" as user "postgres".
corehr=# select relname, pg_size_pretty(pg_total_relation_size(relid)) as size from pg_stat_user_tables;

8.备份所有的库
pg_dumpall -U postgres -c > all_backup_20211018.sql

9.导入所有备份库
psql -U postgres < all_backup_20211018.sql

10.备份某个库
pg_dump -U postgres -c platform > platform_backup_20211018.sql

11.导入某个备份库
psql -U postgres platform< platform_backup_20211018.sql

12.备份某个表
pg_dump -U postgres -c platform -t sec_page_t > sec_page_t.sql

13.导入sql
psql -U postgres -d platform < sec_page_t.sql

14.备份某个库的表结构不备份数据
pg_dump -U postgres -c -s ecs_proxy > uat_ecs_proxy_20211116.sql

15.导入某个库的表结构
psql "host=pg51pb29.postgresql.db.cloud.papub  port=3738 user=laopai password=xxx dbname=ecs_proxy" < uat_ecs_proxy_20211116.sql

16.关闭链接
如果删除数据库的时候提示有连接，可以先将连接的关掉，如删除platform库时。
SELECT pg_terminate_backend(pg_stat_activity.pid) FROM pg_stat_activity WHERE datname='platform' AND pid<>pg_backend_pid();
```

