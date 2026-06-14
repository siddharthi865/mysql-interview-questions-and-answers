# Set 7

| S.No. | Question                                                                                                                                                            |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you insert multiple rows using a single `INSERT` statement?](#question-1-how-do-you-insert-multiple-rows-using-a-single-insert-statement)                   |
| 2.    | [What is the difference between `INSERT` and `INSERT IGNORE`?](#question-2-what-is-the-difference-between-insert-and-insert-ignore)                                 |
| 3.    | [How do you update all rows in a table at once?](#question-3-how-do-you-update-all-rows-in-a-table-at-once)                                                         |
| 4.    | [What happens if you omit the `WHERE` clause in an `UPDATE` or `DELETE` query?](#question-4-what-happens-if-you-omit-the-where-clause-in-an-update-or-delete-query) |
| 5.    | [What is the use of the `AS` keyword in MySQL?](#question-5-what-is-the-use-of-the-as-keyword-in-mysql)                                                             |
| 6.    | [How do you alias a column in a `SELECT` query?](#question-6-how-do-you-alias-a-column-in-a-select-query)                                                           |
| 7.    | [How can you concatenate two strings in MySQL?](#question-7-how-can-you-concatenate-two-strings-in-mysql)                                                           |
| 8.    | [How do you find records that begin or end with a certain character?](#question-8-how-do-you-find-records-that-begin-or-end-with-a-certain-character)               |
| 9.    | [What is the difference between `AND` and `OR` operators?](#question-9-what-is-the-difference-between-and-and-or-operators)                                         |
| 10.   | [How do you perform arithmetic operations in MySQL queries?](#question-10-how-do-you-perform-arithmetic-operations-in-mysql-queries)                                |
| 11.   | [How can you extract the year from a date column?](#question-11-how-can-you-extract-the-year-from-a-date-column)                                                    |
| 12.   | [What is the difference between `DATE_FORMAT()` and `STR_TO_DATE()`?](#question-12-what-is-the-difference-between-date_format-and-str_to_date)                      |
| 13.   | [How do you round numbers in MySQL?](#question-13-how-do-you-round-numbers-in-mysql)                                                                                |
| 14.   | [How can you check for `NULL` values in a query?](#question-14-how-can-you-check-for-null-values-in-a-query)                                                        |
| 15.   | [How do you replace `NULL` values with a default value?](#question-15-how-do-you-replace-null-values-with-a-default-value)                                          |
| 16.   | [What are composite primary keys and when should they be used?](#question-16-what-are-composite-primary-keys-and-when-should-they-be-used)                          |
| 17.   | [What is the purpose of the `ON DELETE SET NULL` constraint?](#question-17-what-is-the-purpose-of-the-on-delete-set-null-constraint)                                |
| 18.   | [How can you enforce uniqueness across multiple columns?](#question-18-how-can-you-enforce-uniqueness-across-multiple-columns)                                      |
| 19.   | [What is referential integrity?](#question-19-what-is-referential-integrity)                                                                                        |
| 20.   | [How do you retrieve records in random order?](#question-20-how-do-you-retrieve-records-in-random-order)                                                            |

## Question 1. How do you insert multiple rows using a single `INSERT` statement?

### **How do you insert multiple rows using a single `INSERT` statement? (MySQL Interview Answer)**

### **Definition**

In MySQL, you can insert **multiple rows in a single `INSERT` statement** by specifying multiple sets of values separated by commas.

This is more efficient than executing multiple individual `INSERT` statements because MySQL parses, optimizes, and commits the operation only once (unless transaction handling changes the behavior).

**Syntax:**

```sql
INSERT INTO table_name (column1, column2, ...)
VALUES
(value1_row1, value2_row1, ...),
(value1_row2, value2_row2, ...),
(value1_row3, value2_row3, ...);
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
```

---

# Example: Insert Multiple Employees

```sql
INSERT INTO employees
(name, department, salary, hire_date, manager_id)
VALUES
('Alice', 'Engineering', 75000, '2020-01-15', NULL),
('Bob', 'Engineering', 80000, '2019-03-10', 1),
('Charlie', 'HR', 60000, '2021-07-22', NULL),
('David', 'Finance', 70000, '2022-04-12', 2);
```

---

# Line-by-Line Explanation

### Line 1

```sql
INSERT INTO employees
```

Specifies the table where data will be inserted.

---

### Line 2

```sql
(name, department, salary, hire_date, manager_id)
```

Lists the columns that will receive values.

Since `id` is `AUTO_INCREMENT`, it is omitted.

---

### Remaining Lines

```sql
VALUES
('Alice', ...),
('Bob', ...),
('Charlie', ...),
('David', ...);
```

Each pair of parentheses represents **one row**.

MySQL inserts all four rows in a single operation.

---

# Result

| id  | name    | department  | salary | hire_date  | manager_id |
| --- | ------- | ----------- | ------ | ---------- | ---------- |
| 1   | Alice   | Engineering | 75000  | 2020-01-15 | NULL       |
| 2   | Bob     | Engineering | 80000  | 2019-03-10 | 1          |
| 3   | Charlie | HR          | 60000  | 2021-07-22 | NULL       |
| 4   | David   | Finance     | 70000  | 2022-04-12 | 2          |

---

# Why Use Multi-Row Inserts?

Instead of:

```sql
INSERT INTO employees (...) VALUES (...);

INSERT INTO employees (...) VALUES (...);

INSERT INTO employees (...) VALUES (...);
```

Use:

```sql
INSERT INTO employees (...)
VALUES
(...),
(...),
(...);
```

This reduces:

- SQL parsing overhead
- Network round trips between the application and MySQL server
- Transaction overhead (especially with autocommit enabled)

As a result, multi-row inserts are generally much faster for bulk data loading.

---

# Example with Explicit IDs

```sql
INSERT INTO employees
(id, name, department, salary, hire_date, manager_id)
VALUES
(10, 'Emma', 'Sales', 65000, '2023-01-10', NULL),
(11, 'Frank', 'Sales', 68000, '2023-02-15', 10);
```

Use this only when you intentionally want to provide primary key values instead of relying on `AUTO_INCREMENT`.

---

# Example Inside a Transaction

```sql
START TRANSACTION;

INSERT INTO employees
(name, department, salary, hire_date)
VALUES
('Grace', 'Marketing', 72000, '2024-01-10'),
('Henry', 'Marketing', 71000, '2024-01-15');

COMMIT;
```

If an error occurs before `COMMIT`, you can roll back the transaction:

```sql
ROLLBACK;
```

This ensures that either all rows are inserted or none are, preserving data consistency.

---

# Performance Advantages

Suppose you need to insert **10,000 rows**.

### Poor Approach

```sql
10,000 separate INSERT statements
```

Problems:

- 10,000 SQL parses
- 10,000 network requests
- 10,000 commits (with autocommit)

---

### Better Approach

```sql
INSERT INTO table VALUES
(...),
(...),
...
```

Advantages:

- One SQL statement
- Fewer network round trips
- Reduced parsing and optimization overhead
- Better overall throughput

This is why multi-row inserts are commonly used in ETL processes, data migrations, and batch imports.

---

# Common Pitfalls

### 1. Column Count Must Match

❌ Incorrect:

```sql
INSERT INTO employees(name, department)
VALUES
('Alice', 'Engineering'),
('Bob');
```

The second row has fewer values than specified columns, resulting in an error.

✔ Correct:

```sql
INSERT INTO employees(name, department)
VALUES
('Alice', 'Engineering'),
('Bob', 'Engineering');
```

---

### 2. Data Type Mismatch

```sql
('Alice', 'Engineering', 'ABC')
```

If `salary` is numeric, `'ABC'` is invalid and may cause an error (depending on the SQL mode).

---

### 3. Primary Key Conflicts

If you explicitly insert duplicate primary key values:

```sql
(1, 'Alice', ...)
(1, 'Bob', ...)
```

MySQL returns a duplicate key error unless you use statements such as `INSERT IGNORE` or `INSERT ... ON DUPLICATE KEY UPDATE`.

---

### 4. Very Large Statements

Although MySQL supports multi-row inserts, extremely large statements may exceed the server's `max_allowed_packet` limit. For very large datasets, insert rows in batches (for example, 1,000–10,000 rows per statement, depending on row size and server configuration).

---

# Optimization Tips

- Use multi-row `INSERT` instead of many single-row inserts for better performance.
- Wrap bulk inserts in a transaction when appropriate to reduce commit overhead.
- Omit `AUTO_INCREMENT` columns unless you need explicit values.
- Use `LOAD DATA` for very large imports (millions of rows), as it is typically much faster than `INSERT`.

---

# Related MySQL Features

- `INSERT IGNORE` — Skip rows that would violate certain constraints.
- `INSERT ... ON DUPLICATE KEY UPDATE` — Insert a row or update it if a duplicate key exists.
- `LOAD DATA` — High-performance bulk loading from files.
- `AUTO_INCREMENT` — Automatically generates unique primary key values.

---

# Interview Tips

If asked, **"Why is inserting multiple rows in one `INSERT` statement faster?"**, explain that:

- MySQL parses and optimizes the statement only once.
- It reduces network communication between the client and server.
- It decreases transaction overhead by grouping inserts into a single operation.
- It is the recommended approach for batch inserts and improves overall performance.

## Question 2. What is the difference between `INSERT` and `INSERT IGNORE`?

## Question 3. How do you update all rows in a table at once?

## Question 4. What happens if you omit the `WHERE` clause in an `UPDATE` or `DELETE` query?

## Question 5. What is the use of the `AS` keyword in MySQL?

## Question 6. How do you alias a column in a `SELECT` query?

## Question 7. How can you concatenate two strings in MySQL?

## Question 8. How do you find records that begin or end with a certain character?

## Question 9. What is the difference between `AND` and `OR` operators?

## Question 10. How do you perform arithmetic operations in MySQL queries?

## Question 11. How can you extract the year from a date column?

## Question 12. What is the difference between `DATE_FORMAT()` and `STR_TO_DATE()`?

## Question 13. How do you round numbers in MySQL?

## Question 14. How can you check for `NULL` values in a query?

## Question 15. How do you replace `NULL` values with a default value?

## Question 16. What are composite primary keys and when should they be used?

## Question 17. What is the purpose of the `ON DELETE SET NULL` constraint?

## Question 18. How can you enforce uniqueness across multiple columns?

## Question 19. What is referential integrity?

## Question 20. How do you retrieve records in random order?
