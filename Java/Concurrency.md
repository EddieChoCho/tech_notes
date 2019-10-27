### The volatile keyword:

* The volatile keyword also ensures visibility across the application. If you declare a field to be volatile, this means that as soon as a write occurs for that field, all reads will see the change. 
* This is true even if local caches are involved—volatile fields are immediately written through to main memory, and reads occur from main memory.(and are not cached).
* Volatile also restricts compiler reordering of accesses during optimization.

#### Volatile vs Synchronization:

* If multiple tasks are accessing a field, that field should be volatile; otherwise, the field should only be accessed via synchronization.
* Synchronization also causes flushing to main memory, so if a field is completely guarded by synchronized methods or blocks, it is not necessary to make it volatile.

#### Scenarios that volatile won’t work:
	
* When the value of a field depends on its previous value (such as incrementing a counter).
* TBD


## References
* [Thinking in Java](https://www.amazon.com/Thinking-Java-4th-Bruce-Eckel/dp/0131872486)
