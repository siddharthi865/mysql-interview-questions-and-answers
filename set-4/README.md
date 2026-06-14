# Set 4

| S.No. | Question                                                                                                                                                                           |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between `COALESCE()` and `IFNULL()`?](#question-1-what-is-the-difference-between-coalesce-and-ifnull)                                                      |
| 2.    | [What is the difference between `UNION` and `UNION ALL`?](#question-2-what-is-the-difference-between-union-and-union-all)                                                          |
| 3.    | [What is the difference between `CHAR_LENGTH()` and `LENGTH()`?](#question-3-what-is-the-difference-between-char_length-and-length)                                                |
| 4.    | [What is a temporary table in MySQL?](#question-4-what-is-a-temporary-table-in-mysql)                                                                                              |
| 5.    | [What is the difference between logical and physical backup?](#question-5-what-is-the-difference-between-logical-and-physical-backup)                                              |
| 6.    | [How do you backup and restore a MySQL database?](#question-6-how-do-you-backup-and-restore-a-mysql-database)                                                                      |
| 7.    | [How can you optimize a slow query?](#question-7-how-can-you-optimize-a-slow-query)                                                                                                |
| 8.    | [What is the `EXPLAIN` statement used for?](#question-8-what-is-the-explain-statement-used-for)                                                                                    |
| 9.    | [What is query caching and how does it improve performance?](#question-9-what-is-query-caching-and-how-does-it-improve-performance)                                                |
| 10.   | [What are the common causes of MySQL performance issues?](#question-10-what-are-the-common-causes-of-mysql-performance-issues)                                                     |
| 11.   | [What is partitioning in MySQL?](#question-11-what-is-partitioning-in-mysql)                                                                                                       |
| 12.   | [What are different types of table partitioning available?](#question-12-what-are-different-types-of-table-partitioning-available)                                                 |
| 13.   | [How does indexing work internally in MySQL?](#question-13-how-does-indexing-work-internally-in-mysql)                                                                             |
| 14.   | [What are composite indexes and when should you use them?](#question-14-what-are-composite-indexes-and-when-should-you-use-them)                                                   |
| 15.   | [How does the query optimizer decide which index to use?](#question-15-how-does-the-query-optimizer-decide-which-index-to-use)                                                     |
| 16.   | [What are covering indexes?](#question-16-what-are-covering-indexes)                                                                                                               |
| 17.   | [What is the difference between clustered and non-clustered indexes?](#question-17-what-is-the-difference-between-clustered-and-non-clustered-indexes)                             |
| 18.   | [What is full-text search in MySQL?](#question-18-what-is-full-text-search-in-mysql)                                                                                               |
| 19.   | [How do you implement full-text indexing?](#question-19-how-do-you-implement-full-text-indexing)                                                                                   |
| 20.   | [What is the difference between `REPLACE INTO` and `INSERT ON DUPLICATE KEY UPDATE`?](#question-20-what-is-the-difference-between-replace-into-and-insert-on-duplicate-key-update) |

## Question 1. What is the difference between `COALESCE()` and `IFNULL()`?

## Difference Between `COALESCE()` and `IFNULL()` in MySQL

### Short Answer (Interview Definition)

Both `COALESCE()` and `IFNULL()` are used to handle `NULL` values in MySQL, but they differ in flexibility:

- **`IFNULL(expr1, expr2)`**: Returns `expr2` if `expr1` is `NULL`; otherwise returns `expr1`.
- **`COALESCE(expr1, expr2, expr3, ...)`**: Returns the **first non-NULL value** from a list of expressions.

`COALESCE()` is part of the SQL standard and can accept multiple arguments, whereas `IFNULL()` is a MySQL-specific function that accepts only two arguments.

---

# 1. `IFNULL()` Syntax

```sql
IFNULL(expression, replacement_value)
```

### Example

```sql
SELECT IFNULL(NULL, 'Not Available') AS result;
```

### Output

```text
result
--------------
Not Available
```

### How It Works

1. Checks the first argument (`NULL`).
2. Since it is `NULL`, MySQL returns the second argument (`'Not Available'`).

---

# 2. `COALESCE()` Syntax

```sql
COALESCE(value1, value2, value3, ...)
```

### Example

```sql
SELECT COALESCE(NULL, NULL, 'Available', 'Default') AS result;
```

### Output

```text
result
---------
Available
```

### How It Works

1. Checks first value → `NULL`
2. Checks second value → `NULL`
3. Finds third value → `'Available'`
4. Returns it immediately.

---

# Using the Sample Employee Table

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

Assume some employees don't have managers:

```sql
SELECT
    name,
    IFNULL(manager_id, 0) AS manager
FROM employees;
```

### Result

```text
Alice    0
Bob      1
Charlie  0
```

Here:

- `manager_id = NULL` → returns `0`
- Otherwise returns actual manager ID.

---

# Example Using `COALESCE()`

Suppose we want to display:

1. Manager ID if available
2. Department if manager is NULL
3. "No Info" if both are NULL

```sql
SELECT
    name,
    COALESCE(
        CAST(manager_id AS CHAR),
        department,
        'No Info'
    ) AS display_value
FROM employees;
```

### Explanation

For each row:

```text
manager_id → first choice
department → second choice
'No Info' → fallback
```

The first non-NULL value is returned.

---

# Key Differences

| Feature             | IFNULL()                                | COALESCE()               |
| ------------------- | --------------------------------------- | ------------------------ |
| Number of arguments | Exactly 2                               | 2 or more                |
| SQL Standard        | No (MySQL-specific)                     | Yes                      |
| Returns             | First argument if not NULL, else second | First non-NULL argument  |
| Flexibility         | Limited                                 | More flexible            |
| Portability         | Lower                                   | Higher                   |
| Common Use          | Simple NULL replacement                 | Multiple fallback values |

---

# Practical Example

### Using `IFNULL()`

```sql
SELECT
    name,
    IFNULL(salary, 0) AS salary
FROM employees;
```

If salary is NULL:

```text
NULL → 0
```

---

### Using `COALESCE()`

```sql
SELECT
    name,
    COALESCE(
        salary,
        50000,
        0
    ) AS effective_salary
FROM employees;
```

Logic:

```text
salary available? → use it
otherwise → 50000
otherwise → 0
```

---

# Performance Consideration

For two arguments:

```sql
IFNULL(col, 'N/A')
```

and

```sql
COALESCE(col, 'N/A')
```

the performance difference is negligible in modern MySQL versions.

Choose based on readability and requirements:

- Simple replacement → `IFNULL()`
- Multiple fallback values → `COALESCE()`

---

# Interview Tip

A common interview answer:

> "`IFNULL()` is a MySQL-specific function that accepts only two arguments and replaces a NULL value with a specified alternative. `COALESCE()` is a SQL-standard function that can accept multiple arguments and returns the first non-NULL value. For portability and multiple fallback options, `COALESCE()` is generally preferred."

---

# Optimization / Best Practice

Prefer:

```sql
COALESCE(email, phone, 'No Contact')
```

over nested `IFNULL()` calls:

```sql
IFNULL(email,
       IFNULL(phone, 'No Contact'))
```

because it is cleaner, easier to read, and follows the SQL standard.

## Question 2. What is the difference between `UNION` and `UNION ALL`?

## Question 3. What is the difference between `CHAR_LENGTH()` and `LENGTH()`?

## Question 4. What is a temporary table in MySQL?

## Question 5. What is the difference between logical and physical backup?

## Question 6. How do you backup and restore a MySQL database?

## Question 7. How can you optimize a slow query?

## Question 8. What is the `EXPLAIN` statement used for?

## Question 9. What is query caching and how does it improve performance?

## Question 10. What are the common causes of MySQL performance issues?

## Question 11. What is partitioning in MySQL?

## Question 12. What are different types of table partitioning available?

## Question 13. How does indexing work internally in MySQL?

## Question 14. What are composite indexes and when should you use them?

## Question 15. How does the query optimizer decide which index to use?

## Question 16. What are covering indexes?

## Question 17. What is the difference between clustered and non-clustered indexes?

## Question 18. What is full-text search in MySQL?

## Question 19. How do you implement full-text indexing?

## Question 20. What is the difference between `REPLACE INTO` and `INSERT ON DUPLICATE KEY UPDATE`?
