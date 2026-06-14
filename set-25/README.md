# Set 25

| S.No. | Question                                                                                                                                                                                   |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [What is partition pruning in MySQL?](#question-1-what-is-partition-pruning-in-mysql)                                                                                                      |
| 2.    | [What is the difference between horizontal and vertical partitioning?](#question-2-what-is-the-difference-between-horizontal-and-vertical-partitioning)                                    |
| 3.    | [How do you split a large table into smaller partitions?](#question-3-how-do-you-split-a-large-table-into-smaller-partitions)                                                              |
| 4.    | [How does MySQL decide which partition to query?](#question-4-how-does-mysql-decide-which-partition-to-query)                                                                              |
| 5.    | [What are hash partitions in MySQL?](#question-5-what-are-hash-partitions-in-mysql)                                                                                                        |
| 6.    | [What are list partitions?](#question-6-what-are-list-partitions)                                                                                                                          |
| 7.    | [How can you move data between partitions?](#question-7-how-can-you-move-data-between-partitions)                                                                                          |
| 8.    | [What are virtual columns in MySQL?](#question-8-what-are-virtual-columns-in-mysql)                                                                                                        |
| 9.    | [What is a generated column, and how is it used?](#question-9-what-is-a-generated-column-and-how-is-it-used)                                                                               |
| 10.   | [How can you encrypt a specific column in MySQL?](#question-10-how-can-you-encrypt-a-specific-column-in-mysql)                                                                             |
| 11.   | [What are the pros and cons of table encryption?](#question-11-what-are-the-pros-and-cons-of-table-encryption)                                                                             |
| 12.   | [How can you secure MySQL connections using SSL/TLS?](#question-12-how-can-you-secure-mysql-connections-using-ssltls)                                                                      |
| 13.   | [How can you detect SQL injection vulnerabilities in MySQL applications?](#question-13-how-can-you-detect-sql-injection-vulnerabilities-in-mysql-applications)                             |
| 14.   | [What is `sql_mode`, and how does it affect behavior?](#question-14-what-is-sql_mode-and-how-does-it-affect-behavior)                                                                      |
| 15.   | [What does the `ONLY_FULL_GROUP_BY` mode do?](#question-15-what-does-the-only_full_group_by-mode-do)                                                                                       |
| 16.   | [How can you enforce strict SQL mode for a database?](#question-16-how-can-you-enforce-strict-sql-mode-for-a-database)                                                                     |
| 17.   | [What are JSON functions available in MySQL 8?](#question-17-what-are-json-functions-available-in-mysql-8)                                                                                 |
| 18.   | [How can you store and query hierarchical data in MySQL?](#question-18-how-can-you-store-and-query-hierarchical-data-in-mysql)                                                             |
| 19.   | [What are the most common performance bottlenecks in MySQL production systems?](#question-19-what-are-the-most-common-performance-bottlenecks-in-mysql-production-systems)                 |
| 20.   | [How would you migrate a MySQL database from one server to another with zero downtime?](#question-20-how-would-you-migrate-a-mysql-database-from-one-server-to-another-with-zero-downtime) |

## Question 1. What is partition pruning in MySQL?

# Partition Pruning in MySQL

## Definition

**Partition pruning** is a **query optimization technique** in MySQL where the optimizer scans **only the required partitions** instead of scanning every partition in a partitioned table.

In other words:

> **If the `WHERE` condition identifies which partition(s) contain the required data, MySQL ignores all other partitions, significantly improving query performance.**

This is one of the biggest advantages of table partitioning.

---

# Why is Partition Pruning Important?

Imagine a sales table partitioned by year.

Instead of searching:

```
2020
2021
2022
2023
2024
2025
```

If your query is:

```sql
WHERE sale_date = '2025-04-15'
```

MySQL immediately knows the row exists only in the **2025 partition**, so it reads only that partition.

Without pruning:

```
Scan all partitions
```

With pruning:

```
Scan only 2025 partition
```

This greatly reduces:

- Disk I/O
- CPU usage
- Query execution time

---

# Example Table

Using the sample database:

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
```

Now suppose we create a partitioned table based on the hire year.

```sql
CREATE TABLE employees_partitioned (
    id INT,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE,
    manager_id INT,
    PRIMARY KEY (id, hire_date)
)
PARTITION BY RANGE (YEAR(hire_date)) (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION pmax VALUES LESS THAN MAXVALUE
);
```

---

# Sample Data

```sql
INSERT INTO employees_partitioned
VALUES
(1,'Alice','Engineering',75000,'2020-03-10',NULL),
(2,'Bob','Engineering',82000,'2021-08-15',1),
(3,'Charlie','HR',60000,'2022-05-20',NULL),
(4,'David','Finance',68000,'2023-01-15',2);
```

---

# Query That Uses Partition Pruning

```sql
SELECT *
FROM employees_partitioned
WHERE hire_date = '2022-05-20';
```

### What happens internally?

The optimizer calculates:

```
YEAR('2022-05-20') = 2022
```

So it knows only:

```
Partition p2022
```

needs to be searched.

Instead of

```
p2020
p2021
p2022
p2023
pmax
```

it accesses only

```
p2022
```

---

# Another Example

```sql
SELECT *
FROM employees_partitioned
WHERE hire_date BETWEEN
'2021-01-01'
AND
'2022-12-31';
```

Only these partitions are read:

```
p2021
p2022
```

Remaining partitions are skipped.

---

# Verify Using `EXPLAIN`

```sql
EXPLAIN
SELECT *
FROM employees_partitioned
WHERE hire_date = '2022-05-20';
```

The execution plan should indicate only the relevant partition (for example, `p2022`) is accessed, confirming that partition pruning has been applied.

---

# When Does MySQL Perform Partition Pruning?

Partition pruning works when the optimizer can determine the relevant partition(s) from the query predicate.

Examples that typically allow pruning:

```sql
WHERE hire_date = '2022-05-20'
```

```sql
WHERE hire_date BETWEEN
'2021-01-01'
AND
'2022-12-31'
```

```sql
WHERE YEAR(hire_date) = 2022
```

_(The effectiveness depends on how the table is partitioned and whether the predicate can be evaluated to identify specific partitions.)_

---

# When Partition Pruning Does NOT Work

## 1. Non-sargable expressions

```sql
WHERE DATE_ADD(hire_date, INTERVAL 1 DAY) =
'2022-05-21'
```

Since the partitioning column is wrapped inside a function, the optimizer may not be able to determine the target partition efficiently.

---

## 2. Unrelated columns

```sql
WHERE department = 'HR'
```

The table is partitioned on `hire_date`, not `department`.

MySQL must search every partition.

---

## 3. Predicates that cannot be resolved to partitions

Complex conditions that prevent MySQL from identifying which partition contains the rows can disable pruning.

---

# How Partition Pruning Improves Performance

Without partition pruning:

```
Read Partition 2020
Read Partition 2021
Read Partition 2022
Read Partition 2023
Read Partition MAX
```

With partition pruning:

```
Read Partition 2022
```

This results in:

- Less disk I/O
- Faster scans
- Lower CPU utilization
- Better response time for large partitioned tables

---

# Best Practices

- Partition tables using columns that are frequently used in `WHERE` clauses.
- Use predicates that directly reference the partitioning column whenever possible.
- Verify pruning with `EXPLAIN` or `EXPLAIN ANALYZE`.
- Avoid unnecessary expressions on the partitioning column that may prevent pruning.
- Remember that partitioning complements proper indexing; it is **not** a replacement for indexes.

---

# Common Pitfalls

- Partitioning on one column while filtering mostly on another.
- Assuming partitioning alone guarantees better performance.
- Using non-sargable predicates on the partitioning column.
- Creating too many partitions, which increases metadata and management overhead.

---

# Interview Tip

A concise interview answer:

> **Partition pruning is a MySQL optimization that allows the query optimizer to access only the partitions that can contain the requested rows instead of scanning every partition. It works when the `WHERE` clause can be mapped to the partitioning expression, reducing disk I/O, CPU usage, and query execution time. It is especially beneficial for large partitioned tables.**

## Question 2. What is the difference between horizontal and vertical partitioning?

## Question 3. How do you split a large table into smaller partitions?

## Question 4. How does MySQL decide which partition to query?

## Question 5. What are hash partitions in MySQL?

## Question 6. What are list partitions?

## Question 7. How can you move data between partitions?

## Question 8. What are virtual columns in MySQL?

## Question 9. What is a generated column, and how is it used?

## Question 10. How can you encrypt a specific column in MySQL?

## Question 11. What are the pros and cons of table encryption?

## Question 12. How can you secure MySQL connections using SSL/TLS?

## Question 13. How can you detect SQL injection vulnerabilities in MySQL applications?

## Question 14. What is `sql_mode`, and how does it affect behavior?

## Question 15. What does the `ONLY_FULL_GROUP_BY` mode do?

## Question 16. How can you enforce strict SQL mode for a database?

## Question 17. What are JSON functions available in MySQL 8?

## Question 18. How can you store and query hierarchical data in MySQL?

## Question 19. What are the most common performance bottlenecks in MySQL production systems?

## Question 20. How would you migrate a MySQL database from one server to another with zero downtime?
