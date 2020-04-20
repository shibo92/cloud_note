### Extra内容(按优化效率从高到底排序) 
 1. Using Index : select 查询的列用到了索引（索引覆盖查询），且where用到了索引，且索引为前导列
 2. Using where Using index : 查询的列被索引覆盖，但是where条件是索引列之一，但不是前导列
 3. Using Where : select 查询列没有用到索引， where用到了索引，但不是前导列 
 4. eg. 表employees中有复合索引(first_name, last_name)
	 ```sql
		explain SELECT t.gender FROM employees_min t where t.last_name = 'bo';
		-- 结果
        # id, select_type, table, partitions, type, possible_keys, key, key_len, ref, rows, filtered, Extra
        '1', 'SIMPLE', 't', NULL, 'ALL', NULL, NULL, NULL, NULL, '10', '10.00', 'Using where'

		-- 优化
  		explain SELECT t.gender FROM employees_min t 
        join
        (
        	select t1.emp_no from employees_min t1 where t1.last_name = 'bo'
        ) as t2 on t.emp_no = t2.emp_no
		-- 结果
		'1', 'SIMPLE', 't1', NULL, 'index', 'PRIMARY', 'idx_first_last', '34', NULL, '10', '10.00', 'Using where; Using index'
        
	 ```
