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
3. 

