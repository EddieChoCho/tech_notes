# Data Concurrency and Consistency

### Tips

#### Get Current Isolation Level

```sql
-- When Transaction is in Progress:
SELECT s.sid,
       s.serial#,
       CASE BITAND(t.flag, POWER(2, 28))
           WHEN 0 THEN 'READ COMMITTED'
           ELSE 'SERIALIZABLE'
           END AS isolation_level
FROM v$transaction t
         JOIN v$session s ON t.addr = s.taddr AND s.sid = sys_context('USERENV', 'SID');
```

#### [Check DML locks](https://docs.oracle.com/en/database/oracle/oracle-database/21/refrn/DBA_DML_LOCKS.html#GUID-B806FD0A-4F10-41A9-81AA-A33E20D048C2)

```sql
select *
from DBA_DML_LOCKS;
-- DBA_DML_LOCKS lists all DML locks held in the database and all outstanding requests for a DML lock.
```

* Columns
    * SESSION_ID:    Session holding or acquiring the lock
    * OWNER: Owner of the lock(not null)
    * NAME: Name of the lock(not null)
    * MODE_HELD: The type of lock held.
        * The value could be SS, SX, S, SSX, or NONE(lock requested but not yet obtained)
    * MODE_REQUESTED: Lock request type.
        * The value could be SS, SX, S, SSX, or NONE(Lock identifier obtained; lock not held or requested)
    * LAST_CONVERT: The last convert
    * BLOCKING_OTHERS: Blocking others

# Others

### [Hierarchical Queries](https://docs.oracle.com/cd/B19306_01/server.102/b14200/queries003.htm)

* start with connect by prior

### [Maximum number of values in SQL IN operator list](http://www.dba-oracle.com/t_maximum_number_of_sql_in_list_values.htm)

* Oracle allows up to 1,000 IN list values in a SQL statement. However, using a long IN list may not be the most
  efficient way of processing such a query.
* Instead, don't write your SQL with a long "IN" Lost. Instead, write the list to a global temporary table and join into
  the GTT:
    * [Using global temporary tables](http://www.dba-oracle.com/t_sql_rewrite_temporary_tables.htm)
    * [global temporary tables](http://www.dba-oracle.com/t_global_temporary_tables.htm)

* You can also load the IN List" values into a PL/SQL collection and process this array as-if it was an Oracle table.
* You also can use Guava's Lists.partition(List, int) method:
    ```
    List<Table> result = ... 
    for (List<Table> partition : Lists.partition(paramtersMoreThan1000, 100)) { 
        // do something with partition 
    }
    ```

## References

* [Data Concurrency and Consistency](https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-concurrency-and-consistency.html#GUID-E8CBA9C5-58E3-460F-A82A-850E0152E95C)