# Set 20

| S.No. | Question                                                                                                                                                                                     |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you log query execution time using triggers or events?](#question-1-how-do-you-log-query-execution-time-using-triggers-or-events)                                                    |
| 2.    | [How do you store historical data using triggers?](#question-2-how-do-you-store-historical-data-using-triggers)                                                                              |
| 3.    | [What are virtual columns and when are they used?](#question-3-what-are-virtual-columns-and-when-are-they-used)                                                                              |
| 4.    | [How do you define a generated column that stores JSON data?](#question-4-how-do-you-define-a-generated-column-that-stores-json-data)                                                        |
| 5.    | [What are the differences between virtual and stored generated columns?](#question-5-what-are-the-differences-between-virtual-and-stored-generated-columns)                                  |
| 6.    | [How can you create a computed column using an expression?](#question-6-how-can-you-create-a-computed-column-using-an-expression)                                                            |
| 7.    | [What is the `WITH CHECK OPTION` in a view?](#question-7-what-is-the-with-check-option-in-a-view)                                                                                            |
| 8.    | [What is an updatable view and what are its restrictions?](#question-8-what-is-an-updatable-view-and-what-are-its-restrictions)                                                              |
| 9.    | [How do you make a non-updatable view updatable?](#question-9-how-do-you-make-a-non-updatable-view-updatable)                                                                                |
| 10.   | [How do you handle recursive data structures (like organizational hierarchy) using CTEs?](#question-10-how-do-you-handle-recursive-data-structures-like-organizational-hierarchy-using-ctes) |
| 11.   | [How can you use `LAG()` and `LEAD()` window functions for comparisons?](#question-11-how-can-you-use-lag-and-lead-window-functions-for-comparisons)                                         |
| 12.   | [How can you find gaps in a sequence of numbers using SQL?](#question-12-how-can-you-find-gaps-in-a-sequence-of-numbers-using-sql)                                                           |
| 13.   | [How do you pivot rows into columns in MySQL?](#question-13-how-do-you-pivot-rows-into-columns-in-mysql)                                                                                     |
| 14.   | [How do you unpivot columns into rows?](#question-14-how-do-you-unpivot-columns-into-rows)                                                                                                   |
| 15.   | [What is the difference between OLTP and OLAP workloads?](#question-15-what-is-the-difference-between-oltp-and-olap-workloads)                                                               |
| 16.   | [How can you design MySQL databases for OLAP workloads?](#question-16-how-can-you-design-mysql-databases-for-olap-workloads)                                                                 |
| 17.   | [What is MySQL HeatWave (used in Oracle Cloud)?](#question-17-what-is-mysql-heatwave-used-in-oracle-cloud)                                                                                   |
| 18.   | [How can you integrate MySQL with Kafka or other message queues?](#question-18-how-can-you-integrate-mysql-with-kafka-or-other-message-queues)                                               |
| 19.   | [What is MySQL Router and when would you use it?](#question-19-what-is-mysql-router-and-when-would-you-use-it)                                                                               |
| 20.   | [How can you scale MySQL horizontally using replication or clustering?](#question-20-how-can-you-scale-mysql-horizontally-using-replication-or-clustering)                                   |

## Question 1. How do you log query execution time using triggers or events?

## Interview Answer: How do you log query execution time using triggers or events?

### Short Answer

**You cannot directly log query execution time using MySQL triggers.** Triggers execute **inside the same transaction** as the triggering SQL statement and do not have access to the execution start/end time of the statement.

However, you can achieve query execution logging using:

1. **Performance Schema** (recommended)
2. **MySQL Slow Query Log**
3. **General Query Log** (for debugging)
4. **Events** that periodically collect execution statistics from Performance Schema
5. **Application-level logging** (most common in production)

---

# Why Triggers Cannot Measure Query Execution Time

A trigger executes:

- Before or after an `INSERT`
- Before or after an `UPDATE`
- Before or after a `DELETE`

It **does not know**:

- When the SQL statement started
- How long the optimizer took
- How long locks waited
- Total execution duration

For example:

```sql
UPDATE employees
SET salary = salary + 1000
WHERE department = 'Engineering';
```

Execution order:

```
Query Starts
      │
Optimizer
      │
Find Rows
      │
BEFORE UPDATE Trigger
      │
Modify Row
      │
AFTER UPDATE Trigger
      │
Commit
```

The trigger only runs during the statement execution. It cannot calculate:

```
End Time - Start Time
```

because it never receives the query start timestamp.

---

# Recommended Method 1: Performance Schema (Best Practice)

Enable the **Performance Schema**, which records execution statistics for SQL statements.

Example:

```sql
SELECT
    EVENT_ID,
    SQL_TEXT,
    TIMER_WAIT / 1000000000000 AS execution_seconds
FROM performance_schema.events_statements_history
ORDER BY EVENT_ID DESC;
```

### Explanation

`events_statements_history`

- Stores recently executed SQL statements.

`SQL_TEXT`

- Shows the executed query.

`TIMER_WAIT`

- Stores execution time in picoseconds.

Convert to seconds:

```sql
TIMER_WAIT / 1000000000000
```

Example output:

| SQL                      | Time    |
| ------------------------ | ------- |
| SELECT \* FROM employees | 0.002 s |
| UPDATE employees ...     | 0.011 s |
| DELETE FROM employees    | 0.004 s |

---

# Recommended Method 2: Create an Event That Logs Query Times

Suppose you want to save execution statistics periodically.

## Step 1: Create a Log Table

```sql
CREATE TABLE query_execution_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    query_text TEXT,
    execution_seconds DECIMAL(10,6),
    logged_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Step 2: Create an Event

```sql
CREATE EVENT log_query_execution
ON SCHEDULE EVERY 1 MINUTE
DO
INSERT INTO query_execution_log
(
    query_text,
    execution_seconds
)
SELECT
    SQL_TEXT,
    TIMER_WAIT / 1000000000000
FROM performance_schema.events_statements_history
WHERE SQL_TEXT IS NOT NULL;
```

### Explanation

Every minute:

- Reads recent statements
- Captures SQL text
- Captures execution time
- Inserts them into your custom table

This is a common approach for building a lightweight audit/history table.

---

# Enable the Event Scheduler

```sql
SET GLOBAL event_scheduler = ON;
```

Without enabling the scheduler, events will not execute.

---

# Recommended Method 3: Slow Query Log

This is the most common production solution.

Enable it:

```sql
SET GLOBAL slow_query_log = 'ON';

SET GLOBAL long_query_time = 1;
```

Meaning:

- Any query taking **more than 1 second** is logged.

Example log:

```
# Query_time: 3.251
SELECT *
FROM employees
WHERE department='Engineering';
```

Advantages:

- Minimal configuration
- Built into MySQL
- Low overhead compared to general logging
- Ideal for performance tuning

---

# Recommended Method 4: General Query Log

Enable:

```sql
SET GLOBAL general_log = 'ON';
```

Logs:

```
Connect
SELECT ...
UPDATE ...
DELETE ...
```

However:

- It logs **every query**
- It **does not record execution duration**
- It has significant performance overhead and is mainly useful for debugging rather than continuous production monitoring.

---

# Recommended Method 5: Application Logging

Many production applications measure execution time around database calls.

Example (pseudo-code):

```text
start = current_time()

execute SQL

end = current_time()

execution_time = end - start

save_to_log()
```

Advantages:

- Includes:
  - Network latency
  - Connection wait time
  - Database execution

- Simple to implement
- Works across different databases

---

# Why Not Use Triggers?

Suppose you write:

```sql
CREATE TRIGGER before_update_employee
BEFORE UPDATE
ON employees
FOR EACH ROW
BEGIN
    -- No function exists to retrieve the overall query start time.
END;
```

The trigger has access to:

- `OLD`
- `NEW`
- Local variables

But **not**:

- Query duration
- Optimizer execution time
- Lock wait time
- Parsing time
- Statement start timestamp

Therefore, triggers are **not suitable** for measuring SQL execution time.

---

# Performance Considerations

- **Performance Schema**: Best choice for detailed, low-overhead statement timing and diagnostics.
- **Slow Query Log**: Excellent for identifying long-running queries in production.
- **General Query Log**: Use only for troubleshooting because it logs every statement and can noticeably impact performance.
- **Events**: Suitable for periodically archiving or summarizing Performance Schema data.
- **Triggers**: Should not be used for execution-time logging since they lack visibility into statement duration.

---

# Interview Tip

If asked, **"Can triggers log execution time?"**, a strong answer is:

> "No. MySQL triggers execute as part of a DML statement but have no access to the statement's overall execution duration or start time. To monitor query execution time, I would use Performance Schema or the Slow Query Log. If I need historical reporting, I can schedule a MySQL Event to periodically copy execution statistics from Performance Schema into a custom logging table."

This demonstrates an understanding of MySQL's execution model and appropriate monitoring tools.

---

# MySQL-Specific Notes

- **Performance Schema** is the preferred mechanism for statement execution metrics and is available in modern MySQL versions.
- **Event Scheduler** must be enabled (`event_scheduler = ON`) before scheduled events will run.
- For query optimization, combine execution-time monitoring with `EXPLAIN` or `EXPLAIN ANALYZE` to understand execution plans and identify bottlenecks.

## Question 2. How do you store historical data using triggers?

## Question 3. What are virtual columns and when are they used?

## Question 4. How do you define a generated column that stores JSON data?

## Question 5. What are the differences between virtual and stored generated columns?

## Question 6. How can you create a computed column using an expression?

## Question 7. What is the `WITH CHECK OPTION` in a view?

## Question 8. What is an updatable view and what are its restrictions?

## Question 9. How do you make a non-updatable view updatable?

## Question 10. How do you handle recursive data structures (like organizational hierarchy) using CTEs?

## Question 11. How can you use `LAG()` and `LEAD()` window functions for comparisons?

## Question 12. How can you find gaps in a sequence of numbers using SQL?

## Question 13. How do you pivot rows into columns in MySQL?

## Question 14. How do you unpivot columns into rows?

## Question 15. What is the difference between OLTP and OLAP workloads?

## Question 16. How can you design MySQL databases for OLAP workloads?

## Question 17. What is MySQL HeatWave (used in Oracle Cloud)?

## Question 18. How can you integrate MySQL with Kafka or other message queues?

## Question 19. What is MySQL Router and when would you use it?

## Question 20. How can you scale MySQL horizontally using replication or clustering?
