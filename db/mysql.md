##### 事务基础

###### 三种数据读取异常

- 脏读
  - 事务B更新数据d
  - 事务A获取数据d
    - 此时数据d已被修改但是未提交
  - 事务B回滚
  - 事务A获取到的是一段脏数据

- 不可重复读
  - 事务A读到了数据d
  - 事务B更新了数据d并提交
  - 事务A再次获取d，结果与前一次不一致

- 幻读
  - 事务A按照索引条件查数据，符合记录数为n
  - 事务B添加m条数据，数据符合事务A的索引查询条件
  - 事务A再次查询条件，复核记录数为n+m



###### 隔离级别

- SERIALIZABLE
  - 串行化。事务串行执行，效率低，但是能避开所有数据读取异常

- REPEATABLE READ
  - 可重复读。永远只读版本<=当前事务id的数据，不会出现脏读和不可重复读，但是会有幻读
  - MySQL innoDB引擎默认隔离级别

- READ COMMITTED 
  - 提交读。只读已提交事务的数据，不会有脏读，但是会有幻读不可重复读
  - 大多数主流数据库的默认事务级别

- READ UNCOMMITTED
  - 未提交读。可以读到其他事务修改未提交的数据，会有脏读、不可重复读和幻读



###### 查看隔离级别

- mysql默认隔离级别为repeatable read
- 查看当前会话隔离级别
  - select @@tx_isolation;

- 查看系统当前隔离级别
  - select @@global.tx_isolation;



##### 事务原理(MVCC)

mysql事务实现的核心是mvcc(multi version concurrency control)，多版本并发控制。

mvcc是基于repeatable read隔离级别的基础上实现的。

