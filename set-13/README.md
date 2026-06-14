# Set 13

| S.No. | Question                                                                                                                                                               |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you update one table based on another table's data?](#question-1-how-do-you-update-one-table-based-on-another-tables-data)                                     |
| 2.    | [How do you delete records from one table based on another table's condition?](#question-2-how-do-you-delete-records-from-one-table-based-on-another-tables-condition) |
| 3.    | [How do you swap values between two columns in a single query?](#question-3-how-do-you-swap-values-between-two-columns-in-a-single-query)                              |
| 4.    | [How can you find rows that have at least one `NULL` value in any column?](#question-4-how-can-you-find-rows-that-have-at-least-one-null-value-in-any-column)          |
| 5.    | [How can you find all foreign key relationships of a table?](#question-5-how-can-you-find-all-foreign-key-relationships-of-a-table)                                    |
| 6.    | [How can you find all indexes on a particular table?](#question-6-how-can-you-find-all-indexes-on-a-particular-table)                                                  |
| 7.    | [How do you rename an index in MySQL?](#question-7-how-do-you-rename-an-index-in-mysql)                                                                                |
| 8.    | [How do you disable and re-enable foreign key checks temporarily?](#question-8-how-do-you-disable-and-re-enable-foreign-key-checks-temporarily)                        |
| 9.    | [What is the difference between `RESTRICT`, `CASCADE`, and `SET NULL`?](#question-9-what-is-the-difference-between-restrict-cascade-and-set-null)                      |
| 10.   | [How can you check table constraints in MySQL?](#question-10-how-can-you-check-table-constraints-in-mysql)                                                             |
| 11.   | [What is the difference between static and dynamic tables?](#question-11-what-is-the-difference-between-static-and-dynamic-tables)                                     |
| 12.   | [How can you monitor query execution time?](#question-12-how-can-you-monitor-query-execution-time)                                                                     |
| 13.   | [What is a slow query log?](#question-13-what-is-a-slow-query-log)                                                                                                     |
| 14.   | [How can you enable or disable the slow query log?](#question-14-how-can-you-enable-or-disable-the-slow-query-log)                                                     |
| 15.   | [How do you use `EXPLAIN ANALYZE` to check query performance?](#question-15-how-do-you-use-explain-analyze-to-check-query-performance)                                 |
| 16.   | [How can you find the number of active connections to a MySQL server?](#question-16-how-can-you-find-the-number-of-active-connections-to-a-mysql-server)               |
| 17.   | [How can you terminate a specific MySQL connection?](#question-17-how-can-you-terminate-a-specific-mysql-connection)                                                   |
| 18.   | [How can you calculate the size of each table in a database?](#question-18-how-can-you-calculate-the-size-of-each-table-in-a-database)                                 |
| 19.   | [What is the difference between a heap table and a normal table?](#question-19-what-is-the-difference-between-a-heap-table-and-a-normal-table)                         |
| 20.   | [How can you compress a table in MySQL?](#question-20-how-can-you-compress-a-table-in-mysql)                                                                           |

## Question 1. How do you update one table based on another table’s data?

## Updating One Table Based on Another Table’s Data in MySQL

### Interview Answer

Updating one table based on another table's data is a very common operation in MySQL. It is typically done using an **`UPDATE` statement with a `JOIN`**.

This allows you to:

- Synchronize data between tables.
- Copy values from one table to another.
- Update records based on matching rows in another table.
- Perform bulk updates efficiently without writing multiple queries.

Unlike some other databases, **MySQL supports `UPDATE ... JOIN` directly**, making it straightforward to update one table using another table's values.

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

INSERT INTO employees (name, department, salary, hire_date, manager_id)
VALUES
('Alice','Engineering',75000,'2020-01-15',NULL),
('Bob','Engineering',80000,'2019-03-10',1),
('Charlie','HR',60000,'2021-07-22',NULL);
```

Suppose we have another table containing revised salaries.

```sql
CREATE TABLE salary_updates (
    employee_id INT,
    new_salary DECIMAL(10,2)
);

INSERT INTO salary_updates
VALUES
(1, 78000),
(2, 85000);
```

---

# Method 1: UPDATE with INNER JOIN (Most Common)

```sql
UPDATE employees e
JOIN salary_updates s
ON e.id = s.employee_id
SET e.salary = s.new_salary;
```

### Line-by-Line Explanation

```sql
UPDATE employees e
```

- Specifies the table to update.
- `e` is an alias for `employees`.

---

```sql
JOIN salary_updates s
```

- Joins the second table containing new values.

---

```sql
ON e.id = s.employee_id
```

- Matches employees with their corresponding update record.

---

```sql
SET e.salary = s.new_salary;
```

- Copies the salary from the second table into the first table.

---

## Before Update

### employees

| id  | name    | salary |
| --- | ------- | ------ |
| 1   | Alice   | 75000  |
| 2   | Bob     | 80000  |
| 3   | Charlie | 60000  |

### salary_updates

| employee_id | new_salary |
| ----------- | ---------- |
| 1           | 78000      |
| 2           | 85000      |

---

## After Update

| id  | name    | salary |
| --- | ------- | ------ |
| 1   | Alice   | 78000  |
| 2   | Bob     | 85000  |
| 3   | Charlie | 60000  |

Charlie remains unchanged because there was no matching row.

---

# Method 2: Update Using Multiple Columns

```sql
UPDATE employees e
JOIN employee_changes c
ON e.id = c.employee_id
SET
    e.salary = c.salary,
    e.department = c.department;
```

Multiple columns can be updated simultaneously.

---

# Method 3: Update Using a Subquery

Sometimes a JOIN isn't necessary.

Example:

```sql
UPDATE employees
SET salary = (
    SELECT MAX(new_salary)
    FROM salary_updates
    WHERE employee_id = employees.id
)
WHERE id IN (
    SELECT employee_id
    FROM salary_updates
);
```

This works when the subquery returns **exactly one value** for each row.

---

# Method 4: Update Using a LEFT JOIN

Suppose missing matches should retain their original value:

```sql
UPDATE employees e
LEFT JOIN salary_updates s
ON e.id = s.employee_id
SET e.salary = IFNULL(s.new_salary, e.salary);
```

Here:

- Matching rows receive the new salary.
- Non-matching rows keep their existing salary.

---

# Real-World Example

Suppose an HR department uploads a new salary sheet.

Instead of updating employees one by one:

```text
employees
          ▲
          │
          │ JOIN
          ▼
salary_updates
```

A single query updates thousands of employee records.

This is much faster and less error-prone than executing individual `UPDATE` statements.

---

# Performance Considerations

For large tables:

### Create indexes on join columns

```sql
CREATE INDEX idx_employee_id
ON salary_updates(employee_id);
```

Also ensure the target table's join column (`employees.id`) is indexed (it is, since it's the primary key).

Without indexes, MySQL performs full table scans, making updates significantly slower.

---

# Use `EXPLAIN` to Analyze the Join

Although `EXPLAIN` does not execute an `UPDATE`, it can be used with the equivalent `SELECT` to inspect the join plan:

```sql
EXPLAIN
SELECT *
FROM employees e
JOIN salary_updates s
ON e.id = s.employee_id;
```

Look for:

- `type` (`ref`, `eq_ref`, etc.)
- `possible_keys`
- `key`
- `rows`

Efficient plans should use indexes on the join columns.

---

# Potential Pitfalls

### 1. Missing JOIN Condition

```sql
UPDATE employees
JOIN salary_updates
SET salary = new_salary;
```

Without an `ON` clause, every row may be matched with every row, producing unintended results.

---

### 2. Duplicate Matching Rows

If `salary_updates` contains multiple rows for the same employee, a target row can match more than once, leading to unpredictable updates. Ensure the join keys are unique or deduplicate the source data before updating.

---

### 3. Updating Too Many Rows

Always preview the affected rows first:

```sql
SELECT *
FROM employees e
JOIN salary_updates s
ON e.id = s.employee_id;
```

Once verified, execute the `UPDATE`.

---

# Optimization Tips

- Index all join columns.
- Update only required rows.
- Test with a `SELECT` before running the `UPDATE`.
- Run large updates inside a transaction when appropriate.

Example:

```sql
START TRANSACTION;

UPDATE employees e
JOIN salary_updates s
ON e.id = s.employee_id
SET e.salary = s.new_salary;

COMMIT;
```

If something goes wrong before committing:

```sql
ROLLBACK;
```

---

# Interview Summary

> In MySQL, the standard way to update one table based on another table is by using **`UPDATE ... JOIN`**. The target table is joined with the source table using a matching condition, and the `SET` clause assigns values from the source table to the target table. This approach is efficient for bulk updates, provided the join columns are properly indexed and each target row matches at most one source row. Always verify the join with a `SELECT` first and consider using transactions for large or critical updates.

## Question 2. How do you delete records from one table based on another table’s condition?

## Question 3. How do you swap values between two columns in a single query?

## Question 4. How can you find rows that have at least one `NULL` value in any column?

## Question 5. How can you find all foreign key relationships of a table?

## Question 6. How can you find all indexes on a particular table?

## Question 7. How do you rename an index in MySQL?

## Question 8. How do you disable and re-enable foreign key checks temporarily?

## Question 9. What is the difference between `RESTRICT`, `CASCADE`, and `SET NULL`?

## Question 10. How can you check table constraints in MySQL?

## Question 11. What is the difference between static and dynamic tables?

## Question 12. How can you monitor query execution time?

## Question 13. What is a slow query log?

## Question 14. How can you enable or disable the slow query log?

## Question 15. How do you use `EXPLAIN ANALYZE` to check query performance?

## Question 16. How can you find the number of active connections to a MySQL server?

## Question 17. How can you terminate a specific MySQL connection?

## Question 18. How can you calculate the size of each table in a database?

## Question 19. What is the difference between a heap table and a normal table?

## Question 20. How can you compress a table in MySQL?
