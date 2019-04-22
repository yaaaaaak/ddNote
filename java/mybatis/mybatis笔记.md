1. Invalid bound statement (not found)可能的原因。

   - 扫描不到你的interface，或者xml目录不对
   - xml的namespace不对

2. Mybatis缓存
   - 一级缓存
     - 默认开启，同sqlSession查询同一个sql时，从缓存获取，默认最多1024个
     - 如果有增删改的commit，则会清空所有一级缓存
   - 二级缓存
     - 可以同mapper跨sqlSession共用缓存，需要主动开启，并且mapper指定select useCache=true
     - 

3. Mybatis事务

   - 使用原生java.sql.Connection的事务提交
   - 提供事务实现的接口，但是不做实现，交给其他组件（容器）去实现
   - 整体来说mybatis事务只做了简单包装，将这一层抛给其他搭配组件去实现

4. 各组件一览（图来自CSDN的[《深入理解mybatis原理》 MyBatis的架构设计以及实例分析](<https://blog.csdn.net/luanlouis/article/details/40422941>)）

   ![](20141028140852531.png)

5. 源码还算好读，而且流程上图已经表述得比较清晰了，这里就不过多解读了

