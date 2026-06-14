# Set 10

| S.No. | Question                                                                                                                                                                                                                           |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you perform a full outer join in MySQL (since it's not supported natively)?](#question-1-how-do-you-perform-a-full-outer-join-in-mysql-since-its-not-supported-natively)                                                   |
| 2.    | [What is a recursive CTE (Common Table Expression)?](#question-2-what-is-a-recursive-cte-common-table-expression)                                                                                                                  |
| 3.    | [How do you write a recursive query using `WITH RECURSIVE`?](#question-3-how-do-you-write-a-recursive-query-using-with-recursive)                                                                                                  |
| 4.    | [What is a window function? Give examples](#question-4-what-is-a-window-function-give-examples)                                                                                                                                    |
| 5.    | [How does `ROW_NUMBER()` differ from `RANK()` and `DENSE_RANK()`?](#question-5-how-does-row_number-differ-from-rank-and-dense_rank)                                                                                                |
| 6.    | [How do you calculate running totals in MySQL?](#question-6-how-do-you-calculate-running-totals-in-mysql)                                                                                                                          |
| 7.    | [How do you detect and resolve deadlocks in MySQL?](#question-7-how-do-you-detect-and-resolve-deadlocks-in-mysql)                                                                                                                  |
| 8.    | [What are wait locks and metadata locks?](#question-8-what-are-wait-locks-and-metadata-locks)                                                                                                                                      |
| 9.    | [How does MySQL handle phantom reads?](#question-9-how-does-mysql-handle-phantom-reads)                                                                                                                                            |
| 10.   | [How do you identify which queries are locking tables?](#question-10-how-do-you-identify-which-queries-are-locking-tables)                                                                                                         |
| 11.   | [What is query profiling in MySQL?](#question-11-what-is-query-profiling-in-mysql)                                                                                                                                                 |
| 12.   | [What is the `performance_schema` used for?](#question-12-what-is-the-performance_schema-used-for)                                                                                                                                 |
| 13.   | [How do you monitor slow queries in MySQL 8?](#question-13-how-do-you-monitor-slow-queries-in-mysql-8)                                                                                                                             |
| 14.   | [What is query execution plan caching?](#question-14-what-is-query-execution-plan-caching)                                                                                                                                         |
| 15.   | [How do you force MySQL to use a specific index?](#question-15-how-do-you-force-mysql-to-use-a-specific-index)                                                                                                                     |
| 16.   | [How do you use the `ANALYZE TABLE` command?](#question-16-how-do-you-use-the-analyze-table-command)                                                                                                                               |
| 17.   | [How do you migrate a MySQL database from one server to another?](#question-17-how-do-you-migrate-a-mysql-database-from-one-server-to-another)                                                                                     |
| 18.   | [What are GTIDs (Global Transaction Identifiers) in replication?](#question-18-what-are-gtids-global-transaction-identifiers-in-replication)                                                                                       |
| 19.   | [How does MySQL handle circular replication?](#question-19-how-does-mysql-handle-circular-replication)                                                                                                                             |
| 20.   | [What new features were introduced in MySQL 8.0 (like CTEs, Window Functions, Roles, JSON Enhancements, etc.)?](#question-20-what-new-features-were-introduced-in-mysql-80-like-ctes-window-functions-roles-json-enhancements-etc) |

## Question 1. How do you perform a full outer join in MySQL (since it’s not supported natively)?

# How do you perform a FULL OUTER JOIN in MySQL (since it’s not supported natively)?

## Interview Answer

A **FULL OUTER JOIN** returns **all rows from both tables**:

- Matching rows from both tables are combined.
- Rows that exist **only in the left table** are included.
- Rows that exist **only in the right table** are also included.
- Missing values are filled with `NULL`.

Unlike databases such as PostgreSQL, SQL Server, and Oracle, **MySQL does not provide a native `FULL OUTER JOIN` operator**. Instead, we simulate it by combining:

1. a `LEFT JOIN`
2. a `RIGHT JOIN`
3. `UNION` (or `UNION ALL` with filtering)

---

## Sample Schema

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

CREATE TABLE departments (
    department VARCHAR(50) PRIMARY KEY,
    location VARCHAR(100)
);

INSERT INTO departments VALUES
('Engineering', 'New York'),
('Finance', 'London'),
('Marketing', 'Chicago');
```

---

# Data

### employees

| id  | name    | department  |
| --- | ------- | ----------- |
| 1   | Alice   | Engineering |
| 2   | Bob     | Engineering |
| 3   | Charlie | HR          |

### departments

| department  | location |
| ----------- | -------- |
| Engineering | New York |
| Finance     | London   |
| Marketing   | Chicago  |

Notice:

- **HR** exists only in `employees`.
- **Finance** and **Marketing** exist only in `departments`.

A FULL OUTER JOIN should return **all four departments**.

---

# Method 1 (Most Common): LEFT JOIN + RIGHT JOIN + UNION

```sql
SELECT
    e.name,
    e.department,
    d.location
FROM employees e
LEFT JOIN departments d
ON e.department = d.department

UNION

SELECT
    e.name,
    e.department,
    d.location
FROM employees e
RIGHT JOIN departments d
ON e.department = d.department;
```

---

## Line-by-line Explanation

### First query

```sql
SELECT ...
FROM employees e
LEFT JOIN departments d
ON e.department = d.department
```

Returns

- All employees
- Matching department if found
- Otherwise NULL

Result

| Employee | Department  | Location |
| -------- | ----------- | -------- |
| Alice    | Engineering | New York |
| Bob      | Engineering | New York |
| Charlie  | HR          | NULL     |

---

### Second query

```sql
RIGHT JOIN departments d
```

Returns

- All departments
- Matching employee if available

Result

| Employee | Department  | Location |
| -------- | ----------- | -------- |
| Alice    | Engineering | New York |
| Bob      | Engineering | New York |
| NULL     | NULL        | London   |
| NULL     | NULL        | Chicago  |

---

### UNION

`UNION` removes duplicate matching rows.

Final output

| Employee | Department  | Location |
| -------- | ----------- | -------- |
| Alice    | Engineering | New York |
| Bob      | Engineering | New York |
| Charlie  | HR          | NULL     |
| NULL     | Finance     | London   |
| NULL     | Marketing   | Chicago  |

This is equivalent to a FULL OUTER JOIN.

---

# Method 2 (Recommended for Large Tables): LEFT JOIN + UNION ALL

Using `UNION` forces MySQL to remove duplicates, which requires sorting or hashing and can be slower on large datasets.

A more efficient approach is:

```sql
SELECT
    e.name,
    e.department,
    d.location
FROM employees e
LEFT JOIN departments d
ON e.department = d.department

UNION ALL

SELECT
    e.name,
    e.department,
    d.location
FROM employees e
RIGHT JOIN departments d
ON e.department = d.department
WHERE e.department IS NULL;
```

---

## Why `WHERE e.department IS NULL`?

The second query returns only the rows that were **not already returned** by the first `LEFT JOIN`.

So:

- matching rows are skipped
- only right-only rows remain

This avoids duplicate elimination while producing the same result.

---

# Visual Representation

```
Employees                Departments

Engineering   ●────────────● Engineering
HR            ●
                         ● Finance
                         ● Marketing
```

FULL OUTER JOIN returns:

```
Engineering  ✓
HR           ✓
Finance      ✓
Marketing    ✓
```

---

# UNION vs UNION ALL

| UNION                      | UNION ALL                                                             |
| -------------------------- | --------------------------------------------------------------------- |
| Removes duplicates         | Keeps duplicates                                                      |
| Requires sorting/hash step | No duplicate removal                                                  |
| Usually slower             | Usually faster                                                        |
| Simpler query              | Needs filtering (`WHERE ... IS NULL`) when simulating FULL OUTER JOIN |

---

# Performance Considerations

For large tables:

- `UNION ALL` is generally faster than `UNION` because it avoids duplicate elimination.
- Ensure indexes exist on the join columns (for example, `department`) to improve join performance.
- Use `EXPLAIN` to verify the execution plan and confirm that indexes are being used.

Example:

```sql
EXPLAIN
SELECT
    e.name,
    e.department,
    d.location
FROM employees e
LEFT JOIN departments d
ON e.department = d.department

UNION ALL

SELECT
    e.name,
    e.department,
    d.location
FROM employees e
RIGHT JOIN departments d
ON e.department = d.department
WHERE e.department IS NULL;
```

---

# Common Pitfalls

### 1. Assuming MySQL supports `FULL OUTER JOIN`

This is incorrect:

```sql
SELECT *
FROM employees
FULL OUTER JOIN departments;
```

MySQL will return a syntax error because `FULL OUTER JOIN` is not supported.

---

### 2. Using `UNION ALL` without filtering

```sql
LEFT JOIN
UNION ALL
RIGHT JOIN
```

This duplicates all matching rows.

Always filter the second query:

```sql
WHERE e.department IS NULL
```

---

### 3. Forgetting indexes

Without indexes on the join columns, MySQL may perform full table scans, which can significantly degrade performance on large tables.

---

# Real-World Use Cases

- Comparing customers and orders (including customers without orders and orders without matching customers)
- Data reconciliation between two systems
- ETL validation and migration checks
- Auditing records across two databases

---

# Interview Tip

If asked, **"Does MySQL support FULL OUTER JOIN?"**, a strong answer is:

> "No. MySQL doesn't support `FULL OUTER JOIN` natively. The standard workaround is to combine a `LEFT JOIN` and a `RIGHT JOIN` using `UNION`, or preferably `UNION ALL` with a filter on unmatched rows to avoid the overhead of duplicate elimination. This approach reproduces the behavior of a FULL OUTER JOIN."

## Question 2. What is a recursive CTE (Common Table Expression)?

## Question 3. How do you write a recursive query using `WITH RECURSIVE`?

## Question 4. What is a window function? Give examples

## Question 5. How does `ROW_NUMBER()` differ from `RANK()` and `DENSE_RANK()`?

## Question 6. How do you calculate running totals in MySQL?

## Question 7. How do you detect and resolve deadlocks in MySQL?

## Question 8. What are wait locks and metadata locks?

## Question 9. How does MySQL handle phantom reads?

## Question 10. How do you identify which queries are locking tables?

## Question 11. What is query profiling in MySQL?

## Question 12. What is the `performance_schema` used for?

## Question 13. How do you monitor slow queries in MySQL 8?

## Question 14. What is query execution plan caching?

## Question 15. How do you force MySQL to use a specific index?

## Question 16. How do you use the `ANALYZE TABLE` command?

## Question 17. How do you migrate a MySQL database from one server to another?

## Question 18. What are GTIDs (Global Transaction Identifiers) in replication?

## Question 19. How does MySQL handle circular replication?

## Question 20. What new features were introduced in MySQL 8.0 (like CTEs, Window Functions, Roles, JSON Enhancements, etc.)?
