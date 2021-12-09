# Partial Results
## Querying Top-N Rows

* For efficient execution, the ranking must be done with a pipelined order by.
* Inform the database whenever you don’t need all rows.
* The database can only optimize a query for a partial result if it knows this from the beginning.
```
SELECT *
  FROM (
       SELECT *
         FROM sales
        ORDER BY sale_date DESC
       )
 WHERE rownum <= 10
```

* COUNT STOPKEY operation: That means the database recognized the top-N syntax.
* A pipelined top-N query doesn’t need to read and sort the entire result set.

-------------------------------------------------------------
| Operation                     | Name        | Rows | Cost |
-------------------------------------------------------------
| SELECT STATEMENT              |             |   10 |    9 |
|  COUNT STOPKEY                |             |      |      |
|   VIEW                        |             |   10 |    9 |
|    TABLE ACCESS BY INDEX ROWID| SALES       | 1004K|    9 |
|     INDEX FULL SCAN DESCENDING| SALES_DT_PR |   10 |    3 |
-------------------------------------------------------------

* If there is no suitable index on SALE_DATE for a pipelined order by, the database must read and sort the entire table. 
--------------------------------------------------
| Operation               | Name  | Rows |  Cost |
--------------------------------------------------
| SELECT STATEMENT        |       |   10 | 59558 |
|  COUNT STOPKEY          |       |      |       |
|   VIEW                  |       | 1004K| 59558 |
|    SORT ORDER BY STOPKEY|       | 1004K| 59558 |
|     TABLE ACCESS FULL   | SALES | 1004K|  9246 |
--------------------------------------------------
