# 目录
1. 存储引擎对比
2. 底层是如何组织存储的
3. 索引
4. buffer Pool
5. bin log
6. redo log
7. undo log
8. 事务的隔离级别
9. mvcc
10. 锁
11. explain
12. 多机

# 1.存储引擎对比
| 特性 | InnoDB | MyISAM |
| :----- | :----- | :----- |
| 事务 | 支持 | 不支持 |
| 外键 | 支持| 不支持|
| 锁级别 | 行级锁 | 表级锁 |
| MVCC | 支持 | 不支持 |
| 全文索引 | 不支持 | 支持 |
| 在线热备份 | 支持 | 不支持 |


# 5.bin log
Binlog是记录所有数据库表结构变更以及表数据修改的二进制日志，主要用于数据复制和备份、主从同步和数据恢复，写入时机是事务提交的时候。文件的记录模式有STATEMENT、ROW、MIXED三种.

![Binlog写入时机的示例图](https://i.imgur.com/4MVEr63.png)

STATAMENT记录每次执行的sql命令，日志小，但是有些sql有毒，比如now()可能导致恢复的时候与之前数据不一致。针对这种函数也有解决方案，如下是一个binlog截图，先设置当前时间戳，然后在运行now()这句sql。
![binlog中特殊方法now()的处理](https://i.imgur.com/gOCFhdH.png "now()在binlog中的处理方法")
ROW记录每一行被改的数据，能还原所有的细节，但是如果有加一列这种操作，导致全表都会生成binlog。
MIXED则是两者结合。
实操：

首先binlog需要在`my.ini`配置文件中打开，对应的是`log-bin=mysql-bin`这一行配置，打开后进行数据写操作就会产生`mysql-bin.000001`这个文件，后缀是递增的一个号。

![image](https://i.imgur.com/nUaMRQy.png)

该文件不能直接打开，可以通过登录mysql后，执行`show binlog events`来查看对应的sql。

![image](https://i.imgur.com/A80CnAU.png)

也可以通过`mysqlbinlog`指令来将其转换成sql：
```shell
mysqlbinlog --no-defaults mysql/data/mysql-bin.000001 > output.sql
```
