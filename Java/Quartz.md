# Quartz

```java
JobDetail job = newJob(HelloJob.class)
    .withIdentity("job1", "group1")
    .build();

// Trigger the job to run now, and then repeat every 40 seconds
Trigger trigger = newTrigger()
    .withIdentity("trigger1", "group1")
    .startNow()
    .withSchedule(simpleSchedule()
    .withIntervalInSeconds(40)
    .repeatForever())
    .build();

// Tell quartz to schedule the job using our trigger
scheduler.scheduleJob(job, trigger);

```
## JobStores[3]

* JobStores:
	* RAMJobStore
	* JDBCJobStore
	* TerracottaJobStore
	
### RAMJobStore
* RAMJobStore is the simplest JobStore to use, it is also the most performant (in terms of CPU time). RAMJobStore gets its name in the obvious way: it keeps all of its data in RAM.
* Configuring Quartz to use RAMJobStore
    * org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore

### JDBCJobStore
* Configuring Quartz to use JobStoreTx
    * org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
* Configuring JDBCJobStore to use a DriverDelegate
    * org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
* Configuring JDBCJobStore with the Table Prefix
    * org.quartz.jobStore.tablePrefix = QRTZ_ 
    * tables: e.g. “QRTZ_TRIGGERS”, and “QRTZ_JOB_DETAIL”
* Configuring JDBCJobStore with the name of the DataSource to use
    * org.quartz.jobStore.dataSource = ${myDS}
    
#### Tables of JDBCJobStore[4][5]

|Table Name          |Description                                 |
|--------------------|--------------------------------------------|
|QRTZ_TRIGGERS       |Stores information about configured triggers|
|QRTZ_SIMPLE_TRIGGERS|Stores simple triggers, including repeat count, internal, and number of times triggered|
|QRTZ_JOB_DETAILS    |Stores detailed information for every configured Job|
...

### TerracottaJobStore
* TerracottaJobStore provides a means for scaling and robustness without the use of a database. 
* TerracottaJobStore can be ran clustered or non-clustered, and in either case provides a storage medium for your job data that is persistent between application restarts, because the data is stored in the Terracotta server. 
* It’s performance is much better than using a database via JDBCJobStore (about an order of magnitude better), but fairly slower than RAMJobStore.
* Configuring Quartz to use TerracottaJobStore
    * org.quartz.jobStore.class = org.terracotta.quartz.TerracottaJobStore
    * org.quartz.jobStore.tcConfigUrl = localhost:9510
		
## Automatically Retry Failed Jobs in Quartz[2]

*  Retry continuously until the job succeeds
    * Throw a JobExecutionException with a flag to tell the scheduler to fire it again when the execution job fails.
    ```java
    public void execute(JobExecutionContext context) throws JobExecutionException {
       try{
           //do something
       }
       catch(Exception e){ 
           JobExecutionException e2 = new JobExecutionException(e);
           //fire it again
           e2.refireImmediately();
           throw e2;
       }
    }
    ``` 
* Retry n times and then disable the job.
    * Use a retryCounter

* Retry after specific time period.
    *  Create another job with specific start time when the execution job fails.

## References
* [1][Java Quartz Best Practices Example](https://examples.javacodegeeks.com/enterprise-java/quartz/java-quartz-best-practices-example/)
* [2][Automatically Retry Failed Jobs in Quartz](http://fahdshariff.blogspot.com/2010/12/automatically-retry-failed-jobs-in.html)
* [3][Quartz Tutorials - Lesson 9: Job Stores](http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-09.html)
* [4][What does each table for quartz scheduler signify?](https://stackoverflow.com/questions/48557255/what-does-each-table-for-quartz-scheduler-signify/52048642)
* [5][tables_mysql_innodb.sql](https://github.com/quartz-scheduler/quartz/blob/master/quartz-core/src/main/resources/org/quartz/impl/jdbcjobstore/tables_mysql_innodb.sql)