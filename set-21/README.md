# Set 21

| S.No. | Question                                                                                                                                                             |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is a query execution plan in MySQL?](#question-1-what-is-a-query-execution-plan-in-mysql)                                                                      |
| 2.    | [How can you analyze and improve a slow query?](#question-2-how-can-you-analyze-and-improve-a-slow-query)                                                            |
| 3.    | [What is the role of the `EXPLAIN` keyword in MySQL?](#question-3-what-is-the-role-of-the-explain-keyword-in-mysql)                                                  |
| 4.    | [How does the query optimizer in MySQL choose an execution plan?](#question-4-how-does-the-query-optimizer-in-mysql-choose-an-execution-plan)                        |
| 5.    | [What are optimizer hints in MySQL, and when are they used?](#question-5-what-are-optimizer-hints-in-mysql-and-when-are-they-used)                                   |
| 6.    | [How can you force MySQL to use a specific index?](#question-6-how-can-you-force-mysql-to-use-a-specific-index)                                                      |
| 7.    | [What is the difference between `STRAIGHT_JOIN` and `JOIN`?](#question-7-what-is-the-difference-between-straight_join-and-join)                                      |
| 8.    | [What are temporary tables, and when are they created internally?](#question-8-what-are-temporary-tables-and-when-are-they-created-internally)                       |
| 9.    | [How does MySQL handle subquery optimization?](#question-9-how-does-mysql-handle-subquery-optimization)                                                              |
| 10.   | [What is the difference between a correlated and a non-correlated subquery?](#question-10-what-is-the-difference-between-a-correlated-and-a-non-correlated-subquery) |
| 11.   | [How does MySQL handle `UNION` vs `UNION ALL` performance-wise?](#question-11-how-does-mysql-handle-union-vs-union-all-performance-wise)                             |
| 12.   | [What is a covering index, and how does it improve query performance?](#question-12-what-is-a-covering-index-and-how-does-it-improve-query-performance)              |
| 13.   | [How can you detect and remove redundant indexes?](#question-13-how-can-you-detect-and-remove-redundant-indexes)                                                     |
| 14.   | [What is a composite index, and when should you use one?](#question-14-what-is-a-composite-index-and-when-should-you-use-one)                                        |
| 15.   | [What are the drawbacks of having too many indexes?](#question-15-what-are-the-drawbacks-of-having-too-many-indexes)                                                 |
| 16.   | [What is an index cardinality, and how is it calculated?](#question-16-what-is-an-index-cardinality-and-how-is-it-calculated)                                        |
| 17.   | [How does MySQL use the query cache (if enabled)?](#question-17-how-does-mysql-use-the-query-cache-if-enabled)                                                       |
| 18.   | [How can you disable the query cache for a specific query?](#question-18-how-can-you-disable-the-query-cache-for-a-specific-query)                                   |
| 19.   | [How does MySQL optimize joins between large tables?](#question-19-how-does-mysql-optimize-joins-between-large-tables)                                               |
| 20.   | [What are the limitations of the MySQL optimizer?](#question-20-what-are-the-limitations-of-the-mysql-optimizer)                                                     |

## Question 1. What is a query execution plan in MySQL?

# What is a Query Execution Plan in MySQL?

## Interview Answer

A **query execution plan** is the strategy that the **MySQL Optimizer** chooses to execute an SQL query efficiently. It describes **how MySQL will access tables, use indexes, perform joins, sort data, and retrieve the requested rows**.

Before executing any query, MySQL analyzes different possible execution methods and selects the one with the **lowest estimated cost** based on available indexes, table statistics, and optimizer rules.

The execution plan is commonly viewed using the `EXPLAIN` statement.

---

# Why is a Query Execution Plan Important?

A query execution plan helps developers and DBAs understand:

- Which indexes are being used
- Whether a full table scan is occurring
- Join order of tables
- Join algorithms used
- Estimated number of rows scanned
- Whether sorting or temporary tables are required
- Overall query efficiency

It is the **first tool used when diagnosing slow queries**.

---

# How MySQL Generates an Execution Plan

When you execute:

```sql
SELECT *
FROM employees
WHERE department = 'Engineering';
```

MySQL internally performs the following steps:

```
SQL Query
      │
      ▼
Parser
      │
      ▼
Optimizer
      │
Chooses:
• Index?
• Join order?
• Scan type?
• Sorting?
      │
      ▼
Execution Plan
      │
      ▼
Storage Engine (InnoDB)
      │
      ▼
Results
```

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

INSERT INTO employees (name, department, salary, hire_date, manager_id) VALUES
('Alice', 'Engineering', 75000, '2020-01-15', NULL),
('Bob', 'Engineering', 80000, '2019-03-10', 1),
('Charlie', 'HR', 60000, '2021-07-22', NULL);
```

---

# Viewing the Execution Plan

```sql
EXPLAIN
SELECT *
FROM employees
WHERE department = 'Engineering';
```

---

# Example Output

```
+----+-------------+-----------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table     | type | possible_keys | key  | key_len | rows | filtered | Extra |
+----+-------------+-----------+------+---------------+------+---------+------+------+-------------+
| 1  | SIMPLE      | employees | ALL  | NULL          | NULL | NULL    | 3    | 33.33 | Using where |
+----+-------------+-----------+------+---------------+------+---------+------+------+-------------+
```

---

# Explanation of Each Column

## 1. id

Execution order of the query.

```
1
```

Usually starts from 1.

---

## 2. select_type

Type of query.

Examples:

- SIMPLE
- PRIMARY
- SUBQUERY
- DERIVED
- UNION

Example:

```
SIMPLE
```

Means it is a simple SELECT statement.

---

## 3. table

The table currently being accessed.

```
employees
```

---

## 4. type (Very Important)

Shows how MySQL accesses the table.

Common access methods (best → worst):

| Type   | Meaning          |
| ------ | ---------------- |
| system | Single-row table |
| const  | One matching row |
| eq_ref | One row per join |
| ref    | Index lookup     |
| range  | Index range scan |
| index  | Full index scan  |
| ALL    | Full table scan  |

Example:

```
ALL
```

Means MySQL scans the entire table.

---

## 5. possible_keys

Indexes that **could** be used.

Example:

```
idx_department
```

---

## 6. key

The index actually chosen.

Example:

```
idx_department
```

If NULL, no index is used.

---

## 7. rows

Estimated number of rows MySQL expects to examine.

Example:

```
100000
```

Lower is generally better.

---

## 8. filtered

Estimated percentage of rows that pass the filter.

Example:

```
20%
```

---

## 9. Extra

Additional execution information.

Common values:

```
Using where
Using index
Using filesort
Using temporary
Using index condition
```

---

# Improving the Plan with an Index

Create an index:

```sql
CREATE INDEX idx_department
ON employees(department);
```

Now run:

```sql
EXPLAIN
SELECT *
FROM employees
WHERE department = 'Engineering';
```

Possible output:

```
type: ref
key: idx_department
rows: 2
Extra: Using index condition
```

### What Changed?

Before:

```
Full table scan

ALL
Rows scanned: 100000
```

After:

```
Index lookup

ref
Rows scanned: 2
```

Huge performance improvement.

---

# `EXPLAIN ANALYZE` (MySQL 8.0.18+)

Unlike `EXPLAIN`, which shows the optimizer's estimates, `EXPLAIN ANALYZE` actually executes the query and reports runtime statistics such as actual rows processed and timing.

```sql
EXPLAIN ANALYZE
SELECT *
FROM employees
WHERE department = 'Engineering';
```

This helps compare the optimizer's estimates with real execution behavior and is especially useful for diagnosing inaccurate statistics or inefficient plans.

---

# Common Reasons for Poor Execution Plans

- Missing indexes
- Using functions on indexed columns
- Leading wildcard searches (e.g., `LIKE '%abc'`)
- Data type mismatches
- Outdated table statistics
- Returning unnecessary columns with `SELECT *`
- Complex joins or subqueries that prevent efficient index use

---

# Optimization Tips

- Create indexes on frequently filtered and joined columns.
- Use `EXPLAIN` before optimizing a slow query.
- Use `EXPLAIN ANALYZE` to validate the actual execution behavior.
- Keep table statistics up to date so the optimizer can make better decisions.
- Avoid operations that disable index usage, such as wrapping indexed columns in functions or using leading wildcards.
- Retrieve only the columns you need instead of using `SELECT *`.

---

# MySQL-Specific Notes

- `EXPLAIN` shows the optimizer's estimated execution plan without running the query.
- `EXPLAIN ANALYZE` (available in MySQL 8.0.18 and later) executes the query and reports actual timing and row counts.
- The `type` column is one of the most important indicators of access efficiency; values like `const`, `eq_ref`, `ref`, and `range` are generally preferable to `ALL`.
- The optimizer chooses a plan based on available indexes, table statistics, and cost estimates.

---

# Common Pitfalls

- Assuming `EXPLAIN` reflects actual runtime behavior—it only shows estimates.
- Ignoring the `rows` estimate; large values often indicate expensive scans.
- Focusing only on whether an index is used, rather than whether it is the _best_ index.
- Over-indexing tables, which can slow down `INSERT`, `UPDATE`, and `DELETE` operations.

## Question 2. How can you analyze and improve a slow query?

## Question 3. What is the role of the `EXPLAIN` keyword in MySQL?

## Question 4. How does the query optimizer in MySQL choose an execution plan?

## Question 5. What are optimizer hints in MySQL, and when are they used?

## Question 6. How can you force MySQL to use a specific index?

## Question 7. What is the difference between `STRAIGHT_JOIN` and `JOIN`?

## Question 8. What are temporary tables, and when are they created internally?

## Question 9. How does MySQL handle subquery optimization?

## Question 10. What is the difference between a correlated and a non-correlated subquery?

## Question 11. How does MySQL handle `UNION` vs `UNION ALL` performance-wise?

## Question 12. What is a covering index, and how does it improve query performance?

## Question 13. How can you detect and remove redundant indexes?

## Question 14. What is a composite index, and when should you use one?

## Question 15. What are the drawbacks of having too many indexes?

## Question 16. What is an index cardinality, and how is it calculated?

## Question 17. How does MySQL use the query cache (if enabled)?

## Question 18. How can you disable the query cache for a specific query?

## Question 19. How does MySQL optimize joins between large tables?

## Question 20. What are the limitations of the MySQL optimizer?
