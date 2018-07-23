---
layout: post
title: mysqldump 使用：MySQL导入导出数据
tags: [MySQL, mysqldump, 数据库, Portfolio]
---

>  mysqldump 常用于MySQL数据库逻辑备份,它主要产生一个SQL脚本，其中包含重新创建数据库所必需的命令CREATE TABLE INSERT等命令。

### mysqldump 的常用参数和用法：

```
-u: 指定用户
-p: 指定密码
--single-transaction: 确保事务性操作，只对 innodb 有效，保证备份期间没有 DDL 操作
-l (–lock-table): 对于非 Innodb 引擎的备份进行锁表，只能进行读操作。与 single-transaction 互斥
-x，–lock-all-table: 给实例下的所有数据库的表进行加锁，保证一致性，该参数会导致在备份过程中数据库只读，不可写
-d: 只备份表结构，不备份数据
-master-data: 有两个值: 1 和 2。1 时只记录change master 语句，为 2 时change master 会注释掉，建议设置为 2
-all-database: 备份 MySQL 实例下的所有数据库
-database: 指定对应的数据库进行备份
-R, --routines: 备份所有的存储过程
--tiggers: 备份触发器
-E, –events: 备份数据库中的调度时间
-hex-blob: 将对 blog/binary 等格式的数据转为 16 进制形式进行保存
-tab=path: 在指定路径下分别生成结构文件和数据文件，会对每个表分别生成一个记录表结构的 sql 文件和记录数据的 txt 文件
-w,–where= 过滤条件，对于单表进行过滤条件的备份
```

1. 全量备份：
> 备份了数据库结构和数据的同时，还会备份存储过程，触发器和调度事件，并且在备份的时候不允许写操作，–master-data=2 参数会注释 change master 语句, 语句内容如下，记录了记录备份操作的二进制日志文件和时间点，对于之后使用 mysqlbinlog 进行增量备份非常有用
```
mysqldump -uroot -p --master-data=2 --single-transaction --routines --triggers --events demo_db > demo_db.bkp.1117.sql
```
2. 备份数据库中的表
> 有时候并不需要对整个数据库进行备份，只需要备份其中的表即可，这时可以直接在数据库后面跟表名即可，下面是备份 demo_db 中的 user 表的命令

```
mysqldump -uroot -p --master-data=2 --single-transaction --routines --triggers --events demo_db user > demo_db.bkp.user.1117.sql
```
3. 使用 --tab 指定备份的目录

```
mysqldump -uroot -p --master-data=2 --single-transaction -R -E --triggers --tab='/tmp/backup_db' demo_db
```

4. 其他常用操作：


```
1 导出一个数据库的结构

mysqldump -d dbname -uroot -p > dbname.sql

2 导出多个数据库的结构

mysqldump -d -B dbname1 dbname2 -uroot -p > dbname.sql

3 导出一个数据库中数据（不包含结构）

mysqldump -t dbname -uroot -p > dbname.sql

4 导出多个数据库中数据（不包含结构）

mysqldump -t -B dbname1 dbname2 -uroot -p > dbname.sql

5 导出一个数据库的结构以及数据

mysqldump dbname -uroot -p > dbname.sql

6 导出多个数据库的结构以及数据

mysqldump -B dbname1 dbname2 -uroot -p > dbname.sql

7 导出一个数据库中一个表的结构

mysqldump -d dbname1 tablename -uroot -p > tablename.sql

8 导出一个数据库中多个表的结构

mysqldump -d -B dbname1 --tables tablename1 tablename2 -uroot -p > tablename.sql

9 导出一个数据库中一个表的数据（不包含结构）

mysqldump -t dbname1 tablename -uroot -p > tablename.sql

10 导出一个数据库中多个表的数据（不包含结构）

mysqldump -t -B dbname1 --tables tablename1 tablename2 -uroot -p > tablename.sql

11 导出一个数据库中一个表的结构以及数据

mysqldump dbname1 tablename -uroot -p > tablename.sql

12 导出一个数据库中多个表的结构以及数据

mysqldump -B dbname1 --tables tablename1 tablename2 -uroot -p > tablename.sql
```


### 导出数据：

1. 导出数据和表结构：
> mysqldump -u用户名 -p密码 数据库名 > 数据库名.sql
```
mysqldump -uroot -p abc > abc.sql   # 敲回车后会提示输入密码
```

2. 只导出表结构:
> mysqldump -u用户名 -p密码 -d 数据库名 > 数据库名.sql

```
mysqldump -uroot -p -d abc > abc.sql
```

### 导入数据

> 导入数据需要进入到具体的数据中

方法一：

```
1.选择数据库
mysql>use abc;

2.设置数据库编码
mysql>set names utf8;

3.导入数据（注意sql文件的路径）
mysql>source /data/abc.sql;
```

方法二：
> mysql -u用户名 -p密码 数据库名 < 数据库名.sql

```
mysql -umysqluser -p abc < abc.sql
```

建议使用第二种方法。
