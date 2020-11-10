### [Hierarchical Queries](https://docs.oracle.com/cd/B19306_01/server.102/b14200/queries003.htm)
* start with connect by prior

### [Maximum number of values in SQL IN operator list](http://www.dba-oracle.com/t_maximum_number_of_sql_in_list_values.htm)
* Oracle allows up to 1,000 IN list values in a SQL statement.  However, using a long IN list may not be the most efficient way of processing such a query.
* Instead, don't write your SQL with a long "IN" Lost. Instead, write the list to a global temporary table and join into the GTT:
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
    

