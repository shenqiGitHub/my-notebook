1.3.1  隔离级别

每一种级别都规定了一个事物中所做的修改，哪些在事务内和事物间是可见的，哪些是不可见的



覆盖索引



extra 

using where 

Using where过滤元组和执行计划是否走全表扫描或走索引查找没有关系。如上测试所示，Using where: 仅仅表示MySQL服务器在收到存储引擎返回的记录后进行“后过滤”（Post-filter）。



using index

表示直接访问索引就能够获取到所需要的数据（索引覆盖），不需要通过索引回表。





'optimizer_switch', index_merge=on,index_merge_union=on,index_merge_sort_union=on,index_merge_intersection=on,engine_condition_pushdown=on,index_condition_pushdown=on,mrr=on,mrr_cost_based=on,block_nested_loop=on,batched_key_access=off,materialization=on,semijoin=on,loosescan=on,firstmatch=on,duplicateweedout=on,subquery_materialization_cost_based=on,use_index_extensions=off,condition_fanout_filter=on,derived_merge=on