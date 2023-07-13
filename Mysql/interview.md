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

![binlog的示例流程图](https://i.imgur.com/YbMkQSh.png)
