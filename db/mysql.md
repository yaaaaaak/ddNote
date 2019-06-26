##### 事务基础

##### ACID

即原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）。

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

实际上是在innoDB存储上的每一列多几个字段的数据：

- DATA_TRX_ID
  - 最新更新的事务id，每多一个事务+1
  - 下称创建id
- DATA_ROLL_PTR
  - undo log的记录id，需要rollback时通过这个找回旧数据
  - 下称删除id
- DB_ROW_ID
  - 索引相关
  - 下称索引id
- DELETE BIT
  - 删除标志位，标记是否要在commit时删除

增删改查对应操作：

- insert：创建id=当前事务id，无删除id
- delete：删除id=当前事务id，创建不变
- update：复制当前行，旧行删除id=当前事务id=新行创建id，旧行标记为删除旧行，执行无问题后删除旧行
- select：读取创建id<=当前事务id，且删除id为空或大于当前事务id的数据
  - 分为快照读和当前读
    - 快照读：读取快照版本，即历史版本
    - 当前读：读取最新版本
  - 单独的查询为快照读
  - 增删改、select lock in share mode或者select for update为当前读

理想中的mvvc ：

- 每行数据都存在一个版本，每次数据更新时都更新该版本
- 修改时Copy出当前版本随意修改，各个事务之间无干扰
- 保存时比较版本号，如果成功（commit），则覆盖原记录；失败则放弃copy（rollback）

innoDB的mvvc：

- 事务以排他锁的形式修改原始数据
- 把修改前的数据存放于undo log，通过回滚指针与主数据关联
- 修改成功（commit）啥都不做，失败则恢复undo log中的数据（rollback）

综上可知，其实innoDB的mvvc并非真实意义上的理想mvvc，只是实现了非阻塞读。

而理想mvvc有其实际实现上的问题。比如事务t1修改row1成功，但是修改row2失败，而事务t2在此期间修改了row1并提交了，如果回滚，那么t2的修改就被干扰了，导致t2违反acid。

###### 主键和非索引的本质区别

b树上存储的数据不同。主键是存data，非主键索引存的是主键id。

