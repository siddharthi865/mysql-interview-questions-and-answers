# Set 14

| S.No. | Question                                                                                                                                                                                 |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How can you check fragmentation in a table?](#question-1-how-can-you-check-fragmentation-in-a-table)                                                                                    |
| 2.    | [What is the use of the `OPTIMIZER_TRACE` feature?](#question-2-what-is-the-use-of-the-optimizer_trace-feature)                                                                          |
| 3.    | [How can you determine which queries are blocked?](#question-3-how-can-you-determine-which-queries-are-blocked)                                                                          |
| 4.    | [What is the difference between `SHOW PROCESSLIST` and `INFORMATION_SCHEMA.PROCESSLIST`?](#question-4-what-is-the-difference-between-show-processlist-and-information_schemaprocesslist) |
| 5.    | [How can you see which queries are currently running?](#question-5-how-can-you-see-which-queries-are-currently-running)                                                                  |
| 6.    | [What are the advantages of using prepared statements?](#question-6-what-are-the-advantages-of-using-prepared-statements)                                                                |
| 7.    | [What is the difference between implicit and explicit commits?](#question-7-what-is-the-difference-between-implicit-and-explicit-commits)                                                |
| 8.    | [How can you identify unused indexes?](#question-8-how-can-you-identify-unused-indexes)                                                                                                  |
| 9.    | [How do you drop duplicate indexes?](#question-9-how-do-you-drop-duplicate-indexes)                                                                                                      |
| 10.   | [How can you convert MyISAM tables to InnoDB?](#question-10-how-can-you-convert-myisam-tables-to-innodb)                                                                                 |
| 11.   | [What is the internal structure of an InnoDB page?](#question-11-what-is-the-internal-structure-of-an-innodb-page)                                                                       |
| 12.   | [How does MySQL handle redo logs and undo segments?](#question-12-how-does-mysql-handle-redo-logs-and-undo-segments)                                                                     |
| 13.   | [What is doublewrite buffering in InnoDB?](#question-13-what-is-doublewrite-buffering-in-innodb)                                                                                         |
| 14.   | [How does adaptive hash indexing work?](#question-14-how-does-adaptive-hash-indexing-work)                                                                                               |
| 15.   | [What is the InnoDB buffer pool?](#question-15-what-is-the-innodb-buffer-pool)                                                                                                           |
| 16.   | [How does the buffer pool improve performance?](#question-16-how-does-the-buffer-pool-improve-performance)                                                                               |
| 17.   | [How can you monitor buffer pool usage?](#question-17-how-can-you-monitor-buffer-pool-usage)                                                                                             |
| 18.   | [How do you increase the InnoDB buffer pool size?](#question-18-how-do-you-increase-the-innodb-buffer-pool-size)                                                                         |
| 19.   | [What are the different MySQL log files and their purposes?](#question-19-what-are-the-different-mysql-log-files-and-their-purposes)                                                     |
| 20.   | [What is binary logging used for?](#question-20-what-is-binary-logging-used-for)                                                                                                         |

## Question 1. How can you check fragmentation in a table?

# How can you check fragmentation in a table in MySQL?

## Interview-Friendly Answer

**Table fragmentation** in MySQL refers to **unused or inefficiently organized storage space** that accumulates after frequent `DELETE`, `UPDATE`, or `ALTER TABLE` operations. Fragmentation can increase disk usage and, in some storage engines, negatively impact I/O performance.

To check fragmentation, MySQL provides information through:

- `SHOW TABLE STATUS`
- `INFORMATION_SCHEMA.TABLES`
- Storage engine-specific metadata (especially for `InnoDB`)

> **Important:** In modern MySQL (latest versions), fragmentation affects storage engines differently:
>
> - **InnoDB:** Internal free space is managed automatically. Fragmentation mainly matters when you want to reclaim disk space or reorganize data.
> - **MyISAM:** Fragmentation is more visible and can significantly affect performance because deleted rows leave gaps.

---

# Sample Schema

```sql
CREATE DATABASE interview_db;
USE interview_db;

CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE,
    manager_id INT
);

INSERT INTO employees (name, department, salary, hire_date, manager_id)
VALUES
('Alice','Engineering',75000,'2020-01-15',NULL),
('Bob','Engineering',80000,'2019-03-10',1),
('Charlie','HR',60000,'2021-07-22',NULL);
```

---

# Method 1: Using `SHOW TABLE STATUS`

```sql
SHOW TABLE STATUS LIKE 'employees';
```

### Sample Output

| Name      | Rows | Data_length | Index_length | Data_free |
| --------- | ---- | ----------- | ------------ | --------- |
| employees | 3    | 16384       | 16384        | 1048576   |

### Explanation

- `Data_length`
  - Size of table data.

- `Index_length`
  - Size occupied by indexes.

- `Data_free`
  - Amount of allocated but unused space.
  - This is the primary indicator of fragmentation.

If `Data_free` is large relative to the table size, fragmentation may exist.

---

# Method 2: Using INFORMATION_SCHEMA

```sql
SELECT
    TABLE_NAME,
    ENGINE,
    DATA_LENGTH,
    INDEX_LENGTH,
    DATA_FREE
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'interview_db'
  AND TABLE_NAME = 'employees';
```

### Explanation

This query retrieves:

- Storage engine
- Data size
- Index size
- Free space available

This method is useful when checking multiple tables.

---

# Method 3: Check All Fragmented Tables

```sql
SELECT
    TABLE_NAME,
    ROUND(DATA_FREE / 1024 / 1024, 2) AS free_space_mb,
    ROUND(DATA_LENGTH / 1024 / 1024, 2) AS data_size_mb
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'interview_db'
ORDER BY DATA_FREE DESC;
```

This helps identify which tables have the most unused space.

---

# Understanding `DATA_FREE`

Suppose:

```
DATA_LENGTH = 500 MB
INDEX_LENGTH = 100 MB
DATA_FREE = 250 MB
```

This means:

- Actual data occupies **500 MB**
- Indexes occupy **100 MB**
- **250 MB is allocated but currently unused**

That unused space may have resulted from:

- Large DELETE operations
- Updates causing row relocation
- Table rebuilds
- Dropped indexes

---

# Reclaiming Fragmented Space

Once fragmentation is identified, you can reorganize the table.

## For InnoDB

```sql
OPTIMIZE TABLE employees;
```

Internally, for `InnoDB`, this rebuilds the table (similar to `ALTER TABLE ... FORCE`), reclaiming unused space and defragmenting the data.

---

## For MyISAM

```sql
OPTIMIZE TABLE employees;
```

For `MyISAM`, this physically defragments the table and updates index statistics.

---

# Line-by-Line Explanation

```sql
OPTIMIZE TABLE employees;
```

- `OPTIMIZE TABLE`
  - Reorganizes the table storage.

- `employees`
  - The table to optimize.

Effects:

- Reclaims unused disk space.
- Defragments data pages.
- Rebuilds indexes.
- Refreshes optimizer statistics.

---

# Performance Considerations

- `OPTIMIZE TABLE` can lock the table or require additional disk space during the rebuild, depending on the storage engine and operation.
- Avoid running it frequently on busy production systems unless there is a clear benefit.
- Large `DATA_FREE` values do not always indicate a performance problem, especially for `InnoDB`, which can reuse free space for future inserts.

---

# Real-World Example

Imagine an `orders` table:

- 100 million rows
- 40 million old rows deleted
- Disk size remains nearly unchanged

Running:

```sql
SHOW TABLE STATUS LIKE 'orders';
```

might show:

```
Data_length : 18 GB
Data_free   : 7 GB
```

Running:

```sql
OPTIMIZE TABLE orders;
```

can rebuild the table and reclaim much of that unused disk space.

---

# Common Pitfalls

1. **Assuming `DATA_FREE` always means poor performance**
   - For `InnoDB`, free space is often reused automatically.

2. **Running `OPTIMIZE TABLE` too often**
   - It is an expensive operation and may not provide measurable benefits if free space will soon be reused.

3. **Ignoring storage engine differences**
   - Fragmentation behaves differently in `InnoDB` and `MyISAM`.

4. **Not scheduling maintenance**
   - Rebuilding large tables during peak hours can affect application availability.

---

# Optimization Tips

- Use `SHOW TABLE STATUS` or query `INFORMATION_SCHEMA.TABLES` periodically to monitor `DATA_FREE`.
- Only optimize tables when there is significant unused space or after large data purges.
- For `InnoDB`, rely on automatic space reuse for routine workloads and reserve `OPTIMIZE TABLE` for reclaiming disk space or reorganizing heavily fragmented tables.
- Use `EXPLAIN` to diagnose slow queries separately; fragmentation is only one potential factor affecting performance.

## Question 2. What is the use of the `OPTIMIZER_TRACE` feature?

## Question 3. How can you determine which queries are blocked?

## Question 4. What is the difference between `SHOW PROCESSLIST` and `INFORMATION_SCHEMA.PROCESSLIST`?

## Question 5. How can you see which queries are currently running?

## Question 6. What are the advantages of using prepared statements?

## Question 7. What is the difference between implicit and explicit commits?

## Question 8. How can you identify unused indexes?

## Question 9. How do you drop duplicate indexes?

## Question 10. How can you convert MyISAM tables to InnoDB?

## Question 11. What is the internal structure of an InnoDB page?

## Question 12. How does MySQL handle redo logs and undo segments?

## Question 13. What is doublewrite buffering in InnoDB?

## Question 14. How does adaptive hash indexing work?

## Question 15. What is the InnoDB buffer pool?

## Question 16. How does the buffer pool improve performance?

## Question 17. How can you monitor buffer pool usage?

## Question 18. How do you increase the InnoDB buffer pool size?

## Question 19. What are the different MySQL log files and their purposes?

## Question 20. What is binary logging used for?
