# Set 19

| S.No. | Question                                                                                                                                                                       |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [How do you find the total size of all databases in a MySQL instance?](#question-1-how-do-you-find-the-total-size-of-all-databases-in-a-mysql-instance)                        |
| 2.    | [How can you check MySQL server uptime?](#question-2-how-can-you-check-mysql-server-uptime)                                                                                    |
| 3.    | [What are user-defined functions (UDFs) in MySQL?](#question-3-what-are-user-defined-functions-udfs-in-mysql)                                                                  |
| 4.    | [How can you create and install a custom UDF?](#question-4-how-can-you-create-and-install-a-custom-udf)                                                                        |
| 5.    | [What is the difference between a function and a stored procedure?](#question-5-what-is-the-difference-between-a-function-and-a-stored-procedure)                              |
| 6.    | [How do you debug stored procedures?](#question-6-how-do-you-debug-stored-procedures)                                                                                          |
| 7.    | [How do you handle exceptions in stored procedures?](#question-7-how-do-you-handle-exceptions-in-stored-procedures)                                                            |
| 8.    | [What is the use of the `SIGNAL` and `RESIGNAL` statements?](#question-8-what-is-the-use-of-the-signal-and-resignal-statements)                                                |
| 9.    | [What is the purpose of the `DECLARE HANDLER` statement?](#question-9-what-is-the-purpose-of-the-declare-handler-statement)                                                    |
| 10.   | [What are the differences between deterministic and non-deterministic functions?](#question-10-what-are-the-differences-between-deterministic-and-non-deterministic-functions) |
| 11.   | [What is a trigger and what are its types?](#question-11-what-is-a-trigger-and-what-are-its-types)                                                                             |
| 12.   | [How can you create a trigger that fires before insert or after delete?](#question-12-how-can-you-create-a-trigger-that-fires-before-insert-or-after-delete)                   |
| 13.   | [Can a trigger call another trigger?](#question-13-can-a-trigger-call-another-trigger)                                                                                         |
| 14.   | [What are cascading triggers and why should they be used carefully?](#question-14-what-are-cascading-triggers-and-why-should-they-be-used-carefully)                           |
| 15.   | [How do you disable and re-enable triggers temporarily?](#question-15-how-do-you-disable-and-re-enable-triggers-temporarily)                                                   |
| 16.   | [How can you prevent recursion in triggers?](#question-16-how-can-you-prevent-recursion-in-triggers)                                                                           |
| 17.   | [What is a stored event in MySQL?](#question-17-what-is-a-stored-event-in-mysql)                                                                                               |
| 18.   | [How do you schedule a stored event?](#question-18-how-do-you-schedule-a-stored-event)                                                                                         |
| 19.   | [What is the difference between a cron job and a MySQL event?](#question-19-what-is-the-difference-between-a-cron-job-and-a-mysql-event)                                       |
| 20.   | [How do you enable or disable the event scheduler?](#question-20-how-do-you-enable-or-disable-the-event-scheduler)                                                             |

## Question 1. How do you find the total size of all databases in a MySQL instance?

## Interview Answer: How do you find the total size of all databases in a MySQL instance?

The **total size of all databases** in a MySQL instance can be determined by querying the **`information_schema.TABLES`** system table.

The `information_schema.TABLES` table stores metadata about every table, including:

- **`DATA_LENGTH`** → Size of the table data
- **`INDEX_LENGTH`** → Size of all indexes
- **`DATA_FREE`** → Unused allocated space (optional, not included in actual table size)

To calculate the total storage used by all databases, sum the `DATA_LENGTH` and `INDEX_LENGTH` columns.

---

# Sample Database

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
('Alice', 'Engineering', 75000, '2020-01-15', NULL),
('Bob', 'Engineering', 80000, '2019-03-10', 1),
('Charlie', 'HR', 60000, '2021-07-22', NULL);
```

---

# Method 1: Total Size of All Databases

```sql
SELECT
    ROUND(SUM(DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024, 2) AS total_size_mb
FROM information_schema.TABLES;
```

### Example Output

| total_size_mb |
| ------------- |
| 845.67        |

---

## Line-by-Line Explanation

```sql
SUM(DATA_LENGTH + INDEX_LENGTH)
```

Adds together:

- Table data
- Index storage

for every table in every database.

---

```sql
/1024/1024
```

Converts bytes into megabytes.

---

```sql
ROUND(...,2)
```

Rounds the result to two decimal places.

---

# Method 2: Size of Each Database

This is a very common interview question.

```sql
SELECT
    TABLE_SCHEMA AS database_name,
    ROUND(SUM(DATA_LENGTH + INDEX_LENGTH)/1024/1024,2) AS size_mb
FROM information_schema.TABLES
GROUP BY TABLE_SCHEMA
ORDER BY size_mb DESC;
```

### Example Output

| database_name | size_mb |
| ------------- | ------: |
| sales         |  530.42 |
| hr            |  210.33 |
| interview_db  |    0.04 |
| mysql         |   35.80 |

This helps identify which databases consume the most storage.

---

# Method 3: Size in GB

```sql
SELECT
    ROUND(SUM(DATA_LENGTH + INDEX_LENGTH)/1024/1024/1024,2) AS total_size_gb
FROM information_schema.TABLES;
```

---

# Excluding System Databases

Often, you only want the size of user-created databases.

```sql
SELECT
    ROUND(SUM(DATA_LENGTH + INDEX_LENGTH)/1024/1024,2) AS user_db_size_mb
FROM information_schema.TABLES
WHERE TABLE_SCHEMA NOT IN (
    'mysql',
    'information_schema',
    'performance_schema',
    'sys'
);
```

---

# Understanding the Columns

| Column         | Meaning                                                           |
| -------------- | ----------------------------------------------------------------- |
| `TABLE_SCHEMA` | Database name                                                     |
| `TABLE_NAME`   | Table name                                                        |
| `DATA_LENGTH`  | Size of table data (bytes)                                        |
| `INDEX_LENGTH` | Size of indexes (bytes)                                           |
| `DATA_FREE`    | Allocated but currently unused space (can indicate fragmentation) |

---

# Real-World Use Cases

- Monitor overall MySQL storage usage.
- Identify databases consuming the most disk space.
- Plan storage capacity before disk exhaustion.
- Track database growth over time.
- Estimate backup sizes and maintenance windows.

---

# Performance Notes

- `information_schema.TABLES` provides metadata maintained by MySQL, making these queries efficient because they do not scan actual table data.
- For large instances with many tables, querying metadata may still take some time, but it is far less expensive than inspecting every row.
- Storage values are approximate for some storage engines and can vary slightly depending on the engine and MySQL version.

---

# Optimization Tips

- Use `GROUP BY TABLE_SCHEMA` to generate per-database storage reports.
- Exclude system schemas when you're only interested in application databases.
- If you need to analyze storage at the table level, include `TABLE_NAME` in the query.
- If you suspect significant unused allocated space, examine `DATA_FREE` alongside `DATA_LENGTH` and `INDEX_LENGTH`.

---

# MySQL-Specific Notes

- `DATA_LENGTH` and `INDEX_LENGTH` are reported in bytes.
- For **InnoDB** tables, these values are estimates based on metadata and may not exactly match the operating system's reported file sizes.
- In installations using a shared InnoDB tablespace (for example, when `innodb_file_per_table` is disabled), table-level sizes may not directly correspond to individual disk files.
- These queries are compatible with the latest MySQL 8.x releases and rely on the `information_schema.TABLES` metadata.

---

# Common Pitfalls

- **Forgetting indexes:** Summing only `DATA_LENGTH` underestimates total storage because index space is omitted.
- **Including system schemas:** This inflates the reported size if you're only interested in user databases.
- **Assuming exact disk usage:** Metadata values may differ slightly from filesystem usage due to storage engine implementation details, compression, and shared tablespaces.

## Question 2. How can you check MySQL server uptime?

## Question 3. What are user-defined functions (UDFs) in MySQL?

## Question 4. How can you create and install a custom UDF?

## Question 5. What is the difference between a function and a stored procedure?

## Question 6. How do you debug stored procedures?

## Question 7. How do you handle exceptions in stored procedures?

## Question 8. What is the use of the `SIGNAL` and `RESIGNAL` statements?

## Question 9. What is the purpose of the `DECLARE HANDLER` statement?

## Question 10. What are the differences between deterministic and non-deterministic functions?

## Question 11. What is a trigger and what are its types?

## Question 12. How can you create a trigger that fires before insert or after delete?

## Question 13. Can a trigger call another trigger?

## Question 14. What are cascading triggers and why should they be used carefully?

## Question 15. How do you disable and re-enable triggers temporarily?

## Question 16. How can you prevent recursion in triggers?

## Question 17. What is a stored event in MySQL?

## Question 18. How do you schedule a stored event?

## Question 19. What is the difference between a cron job and a MySQL event?

## Question 20. How do you enable or disable the event scheduler?
