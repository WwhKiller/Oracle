# 实验一：分析SQL执行计划，执行SQL语句的优化指导

## 实验内容：
	对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。首先运行和分析教材中的样例：本训练任务目的是查询两个部
	门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。
	设计自己的查询语句，并作相应的分析，查询语句不能太简单。

### 查询1：
```SQL
	set autotrace on
   	SELECT d.department_name，count(e.job_id)as "部门总人数"，
	avg(e.salary)as "平均工资"
	from hr.departments d，hr.employees e
	where d.department_id = e.department_id
	and d.department_name in ('IT'，'Sales')
	GROUP BY department_name;<br>
```
### 结果1：
```SQL
DEPARTMENT_NAME                     部门总人数       平均工资

IT                                      5       5760
Sales                                  34 8955.88235

Explain Plan


PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           

Plan hash value: 1370531240
 

| Id  | Operation                     | Name              | Rows  | Bytes | Cost (%CPU)| Time     |

|   0 | SELECT STATEMENT              |                   |     2 |    46 |     2   (0)| 00:00:01 |
|   1 |  SORT GROUP BY NOSORT         |                   |     2 |    46 |     2   (0)| 00:00:01 |
|   2 |   NESTED LOOPS                |                   |    19 |   437 |     2   (0)| 00:00:01 |
|   3 |    NESTED LOOPS               |                   |    20 |   437 |     2   (0)| 00:00:01 |
|   4 |     INLIST ITERATOR           |                   |       |       |            |          |
|*  5 |      INDEX RANGE SCAN         | IDX$$_006F0001    |     2 |    32 |     1   (0)| 00:00:01 |

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           

|*  6 |     INDEX RANGE SCAN          | EMP_DEPARTMENT_IX |    10 |       |     0   (0)| 00:00:01 |
|   7 |    TABLE ACCESS BY INDEX ROWID| EMPLOYEES         |    10 |    70 |     1   (0)| 00:00:01 |

 
Predicate Information (identified by operation id):

 
   5 - access("D"."DEPARTMENT_NAME"='IT' OR "D"."DEPARTMENT_NAME"='Sales')
   6 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")

Statistics

              26  Requests to/from client
              25  SQL*Net roundtrips to/from client
               6  buffer is not pinned count
              76  buffer is pinned count
             671  bytes received via SQL*Net from client
           47138  bytes sent via SQL*Net to client
               2  calls to get snapshot scn: kcmgss
               2  calls to kcmgcs
               5  consistent gets
               5  consistent gets from cache
               5  consistent gets pin
               5  consistent gets pin (fastpath)
               2  execute count
               4  index scans kdiixs1
           40960  logical read bytes from cache
               3  no work - consistent read gets
              25  non-idle wait count
               2  opened cursors cumulative
               1  opened cursors current
               2  parse count (total)
               1  session cursor cache hits
               5  session logical reads
               1  sorts (memory)
            1178  sorts (rows)
              39  table fetch by rowid
              26  user calls
```
### 查询2：
```SQL
	 set autotrace on
	SELECT d.department_name，count(e.job_id)as "部门总人数"，
	avg(e.salary)as "平均工资"
	FROM hr.departments d，hr.employees e
	WHERE d.department_id = e.department_id
	GROUP BY department_name
	HAVING d.department_name in ('IT'，'Sales');
```
### 结果2：
```SQL
DEPARTMENT_NAME                     部门总人数       平均工资

IT                                      5       5760
Sales                                  34 8955.88235
Explain Plan
PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                      
Plan hash value: 2694570928 
|id  | Operation            | Name           | Rows  | Bytes | Cost (%CPU)| Time     |

|   0 | SELECT STATEMENT     |                |     1 |    23 |     5  (20)| 00:00:01 |
|*  1 |  FILTER              |                |       |       |            |          |
|   2 |   HASH GROUP BY      |                |     1 |    23 |     5  (20)| 00:00:01 |
|*  3 |    HASH JOIN         |                |   106 |  2438 |     4   (0)| 00:00:01 |
|   4 |     INDEX FULL SCAN  | IDX$$_006F0001 |    27 |   432 |     1   (0)| 00:00:01 |
|   5 |     TABLE ACCESS FULL| EMPLOYEES      |   107 |   749 |     3   (0)| 00:00:01 |
PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           


 
Predicate Information (identified by operation id):

 
   1 - filter("DEPARTMENT_NAME"='IT' OR "DEPARTMENT_NAME"='Sales')
   3 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")

Statistics

               1  CPU used by this session
               1  CPU used when call started
               1  DB time
              26  Requests to/from client
              25  SQL*Net roundtrips to/from client
               1  buffer is not pinned count
             669  bytes received via SQL*Net from client
           47140  bytes sent via SQL*Net to client
               2  calls to get snapshot scn: kcmgss
               4  calls to kcmgcs
               8  consistent gets
               8  consistent gets from cache
               8  consistent gets pin
               8  consistent gets pin (fastpath)
               1  cursor authentications
               2  execute count
               1  index scans kdiixs1
           65536  logical read bytes from cache
               5  no work - consistent read gets
              25  non-idle wait count
               2  opened cursors cumulative
               1  opened cursors current
               2  parse count (total)
               1  session cursor cache hits
               8  session logical reads
               1  sorts (memory)
            1178  sorts (rows)
               5  table scan blocks gotten
             107  table scan disk non-IMC rows gotten
             107  table scan rows gotten
               1  table scans (short tables)
              26  user calls
```
### 查询1执行计划：
	查询1的SQL语句是直接从hr.departments d和hr.employees e两个表中查询出"部门总人数"和"平均工资",
	然后通过where子句进行约束限制,最后通department_name过department_name来排列显示。
### 查询2执行计划：
	查询2的SQL语句是直接从hr.departments d和hr.employees e两个表中查询出"部门总人数"和"平均工资",
	然后通过where子句进行约束限制然后又通过department_name来排列显示,最后通过having in 来过滤显示内容。
### 分析查询1和查询2的SQL语句谁较优： 
			     （1）查询1的Cost(%CPU)普遍比查询2的Cost（%CPU）消耗低
			     （2）通过查询时间可以看出查询1的SQL语句比查询2的SQL的SQL语句查询时间低
			     （3）因为Oracle存在缓存的机制，多运行几次后依旧是查询1语句较优    	
				
### 最优查询：	
	
			     	
		
	
