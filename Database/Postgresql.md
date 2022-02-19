# Chapter 2. The SQL Language

* 2.4. Populating a Table With Rows
    * You could also have used COPY to load large amounts of data from flat-text files. This is usually faster because
      the COPY command is optimized for this application while allowing less flexibility than INSERT. An example would
      be:
  ```sql
  COPY weather FROM '/home/user/weather.txt';
  ```
    * Where the file name for the source file must be available on the machine running the backend process, not the
      client.

* 2.7. Aggregate Functions
    * Work around of using aggregate functions for where condition - using sub-query
        ```sql
        SELECT city FROM weather WHERE temp_lo = max(temp_lo);     --WRONG
        ```
        * This restriction exists because the WHERE clause determines which rows will be included in the aggregate
          calculation; so obviously it has to be evaluated before aggregate functions are computed.
        * Work around by using sub-query:
        ```sql
        SELECT city FROM weather WHERE temp_lo = (SELECT max(temp_lo) FROM weather);
        ```
    * Aggregates are also very useful in combination with GROUP BY clauses.
      ```sql
        SELECT city, max(temp_lo) FROM weather GROUP BY city;
      ```
    * We can filter these grouped rows using HAVING:
      ```sql
      SELECT city, max(temp_lo) FROM weather GROUP BY city HAVING max(temp_lo) < 40;
      ```
    * The interaction between aggregates and SQL's WHERE and HAVING clauses.
        * WHERE selects input rows before groups and aggregates are computed (thus, it controls which rows go into the
          aggregate computation).
            * Thus, the WHERE clause must not contain aggregate functions.
        * HAVING selects group rows after groups and aggregates are computed.
            * On the other hand, the HAVING clause always contains aggregate functions.
            * You are allowed to write a HAVING clause that doesn't use aggregates, but it's seldom useful. The same
              condition could be used more efficiently at the WHERE stage.

# Chapter 3. Advanced Features

* 3.2. Views
    * You can create a view over the query, which gives a name to the query that you can refer to like an ordinary
      table:
  ```sql
  CREATE VIEW myview AS
    SELECT name, temp_lo, temp_hi, prcp, date, location
        FROM weather, cities
        WHERE city = name;

  SELECT * FROM myview;
  ```
    * Views allow you to encapsulate the details of the structure of your tables, which might change as your application
      evolves, behind consistent interfaces.

* 3.4. Transactions
    * In PostgreSQL, a transaction is set up by surrounding the SQL commands of the transaction with BEGIN and COMMIT
      commands.
      ```sql
      BEGIN;
      UPDATE accounts SET balance = balance - 100.00 WHERE name = 'Alice';
      -- etc etc
      COMMIT; -- or ROLLBACK;
      ```
    * PostgreSQL actually treats every SQL statement as being executed within a transaction. If you do not issue a BEGIN
      command, then each individual statement has an implicit BEGIN and (if successful) COMMIT wrapped around it.
    * A group of statements surrounded by BEGIN and COMMIT is sometimes called a transaction block.
    * `SAVEPOINT` and `ROLLBACK TO`
        * It's possible to control the statements in a transaction in a more granular fashion through the use of
          savepoints.
        * Savepoints allow you to selectively discard parts of the transaction, while committing the rest. After
          defining a savepoint with SAVEPOINT, you can if needed roll back to the savepoint with ROLLBACK TO.
        * After rolling back to a savepoint, it continues to be defined, so you can roll back to it several times.
        * Conversely, if you are sure you won't need to roll back to a particular savepoint again, it can be released,
          so the system can free some resources.
            * Keep in mind that either releasing or rolling back to a savepoint will automatically release all
              savepoints that were defined after it.
        * All this is happening within the transaction block, so none of it is visible to other database sessions.
        * e.g., suppose we debit $100.00 from Alice's account, and credit Bob's account, only to find later that we
          should have credited Wally's account. We could do it using savepoints like this:
            ```sql
            BEGIN;
            UPDATE accounts SET balance = balance - 100.00 WHERE name = 'Alice';
            SAVEPOINT my_savepoint; 
            UPDATE accounts SET balance = balance + 100.00 WHERE name = 'Bob';
            -- oops ... forget that and use Wally's account
            ROLLBACK TO my_savepoint;
            UPDATE accounts SET balance = balance + 100.00
            WHERE name = 'Wally';
            COMMIT;
            ```

## The Information Schema

* List all the created view
    ```sql
        SELECT * FROM information_schema.views;
    ```
* List information of columns of a table
    ```sql
        SELECT column_name, column_default, is_nullable, data_type 
        FROM information_schema.columns 
        WHERE table_name = 'customer';
    ```
* List all the foreign keys of a table
    ```sql
        SELECT 
            tc.table_schema, tc.constraint_name, tc.table_name, kcu.column_name, 
            ccu.table_schema AS foreign_table_schema, 
            ccu.table_name AS foreign_table_name, 
            ccu.column_name AS foreign_column_name 
        FROM
            information_schema.table_constraints AS tc
        JOIN information_schema.key_column_usage AS kcu
            ON tc.constraint_name = kcu.constraint_name
            AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu
            ON ccu.constraint_name = tc.constraint_name
            AND ccu.table_schema = tc.table_schema
        WHERE tc.constraint_type = 'FOREIGN KEY' AND tc.table_name='customer';
    ```

## System Catalogs

* List all the indexes of a table
    ```sql
        SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'customer';
    ```

# References

* [PostgreSQL 14.2 Documentation](https://www.postgresql.org/docs/14/index.html)
