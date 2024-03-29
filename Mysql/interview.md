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
操作配置：

首先binlog需要在`my.ini`配置文件中打开，对应的是`log-bin=mysql-bin`这一行配置，打开后进行数据写操作就会产生`mysql-bin.000001`这个文件，后缀是递增的一个号。

![image](https://i.imgur.com/nUaMRQy.png)

该文件不能直接打开，可以通过登录mysql后，执行`show binlog events`来查看对应的sql。

![image](https://i.imgur.com/A80CnAU.png)

也可以通过`mysqlbinlog`指令来将其转换成sql：
```shell
mysqlbinlog --no-defaults mysql/data/mysql-bin.000001 > output.sql
```
可以添加`--start-datetime="2023-06-30 00:00:00" --stop-datetime="2023-06-30 08:00:00"`这种参数来圈定固定的时间范围，或者使用`--start-position=190 --stop-position=888`来确定位置，position就是上面sql截图中每一句sql都有的一个偏移量。`--database=test`则可以指定某个db。

  一般可以通过`mysqldump`和`binlog`相结合的方式来实现线上数据的备份，这样可以使得mysql数据可以快速恢复到任意时间点。例如每天执行一次`mysqldump`同时带上`--flush-logs`标志，这样每天会生成一份全量数据库的sql文件，同时切换`binlog`文件到一个新的文件，来打印新的一天的sql。想要恢复到3号10点10分，那么可以先用3号0点的`dump.sql`快速恢复到0点，然后再用3号的`binlog`通过指定`--stop-datetime`恢复到具体的10点10分。这样比纯用`binlog`的速度快。

https://github.com/sunwu51/notebook/blob/master/README.md
