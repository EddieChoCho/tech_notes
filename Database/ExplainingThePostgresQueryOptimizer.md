# Explaining the Postgres Query Optimizer

* What Decisions Does the Optimizer Have to Make?
  * Scan Method
  * Join Method
  * Join Order

* Which Scan Method?
  * Sequential Scan
  * Bitmap Index Scan
  * Index Scan

* TBD...

* Which Join Method?
  * Nested Loop
    * With Inner Sequential Scan
    * With Inner Index Scan
  * Hash Join
  * Merge Join

* Pseudocode for Nested Loop Join with Inner Sequential Scan
  ```
  for (i = 0; i < length(outer); i++)
    for (j = 0; j < length(inner); j++)
      if (outer[i] == inner[j])
        output(outer[i], inner[j]);
  ```

* Pseudocode for Hash Join
  ```
  for (j = 0; j < length(inner); j++)
    hash_key = hash(inner[j]);
    append(hash_store[hash_key], inner[j]);
  for (i = 0; i < length(outer); i++)
    hash_key = hash(outer[i]);
    for (j = 0; j < length(hash_store[hash_key]); j++)
      if (outer[i] == hash_store[hash_key][j])
        output(outer[i], inner[j]);
  ```
* Pseudocode for Merge Join
  ```
  sort(outer);
  sort(inner);
  i = 0;
  j = 0;
  save_j = 0;
  while (i < length(outer))
    if (outer[i] == inner[j])
      output(outer[i], inner[j]);
    if (outer[i] >= inner[j] && j < length(inner))
      j++;
      if (outer[i] > inner[j])
        save_j = j;
    else
      i++;
      j = save_j;
  ```

## References

* [Explaining the Postgres Query Optimizer by Bruce Momjian](https://momjian.us/main/presentations/performance.html#optimizer)
