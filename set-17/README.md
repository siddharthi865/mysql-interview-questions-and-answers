# Set 17

| S.No. | Question                                                                                                                                                   |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you find all the columns common between two tables?](#question-1-how-do-you-find-all-the-columns-common-between-two-tables)                        |
| 2.    | [What are aliases and why are they used?](#question-2-what-are-aliases-and-why-are-they-used)                                                              |
| 3.    | [Can we use aliases in the `WHERE` clause?](#question-3-can-we-use-aliases-in-the-where-clause)                                                            |
| 4.    | [What is a scalar subquery?](#question-4-what-is-a-scalar-subquery)                                                                                        |
| 5.    | [What is a row subquery?](#question-5-what-is-a-row-subquery)                                                                                              |
| 6.    | [Can we use subqueries inside the `FROM` clause?](#question-6-can-we-use-subqueries-inside-the-from-clause)                                                |
| 7.    | [What is a correlated subquery and when should you use it?](#question-7-what-is-a-correlated-subquery-and-when-should-you-use-it)                          |
| 8.    | [What is the difference between `=` and `IS` when comparing `NULL` values?](#question-8-what-is-the-difference-between--and-is-when-comparing-null-values) |
| 9.    | [How do you count only distinct values from a column?](#question-9-how-do-you-count-only-distinct-values-from-a-column)                                    |
| 10.   | [What is the use of the `EXISTS` keyword?](#question-10-what-is-the-use-of-the-exists-keyword)                                                             |
| 11.   | [How do you calculate total and average in a single query?](#question-11-how-do-you-calculate-total-and-average-in-a-single-query)                         |
| 12.   | [What does `ROLLUP` do in a `GROUP BY` query?](#question-12-what-does-rollup-do-in-a-group-by-query)                                                       |
| 13.   | [What is the difference between `INNER QUERY` and `OUTER QUERY`?](#question-13-what-is-the-difference-between-inner-query-and-outer-query)                 |
| 14.   | [How can you find the total number of columns in a table?](#question-14-how-can-you-find-the-total-number-of-columns-in-a-table)                           |
| 15.   | [How can you find all tables containing a specific column name?](#question-15-how-can-you-find-all-tables-containing-a-specific-column-name)               |
| 16.   | [How can you find which queries are consuming the most resources?](#question-16-how-can-you-find-which-queries-are-consuming-the-most-resources)           |
| 17.   | [What is a query execution plan?](#question-17-what-is-a-query-execution-plan)                                                                             |
| 18.   | [How can you view an execution plan in MySQL Workbench?](#question-18-how-can-you-view-an-execution-plan-in-mysql-workbench)                               |
| 19.   | [What are derived indexes and why are they important?](#question-19-what-are-derived-indexes-and-why-are-they-important)                                   |
| 20.   | [How can you determine if MySQL used an index for a query?](#question-20-how-can-you-determine-if-mysql-used-an-index-for-a-query)                         |

## Question 1. How do you find all the columns common between two tables?

## Interview Answer: How do you find all the columns common between two tables in MySQL?

### Definition

To find **common columns between two tables**, you compare their column names stored in MySQL's **`INFORMATION_SCHEMA.COLUMNS`** metadata table.

This is a common interview question because it tests your knowledge of:

- Database metadata
- `INFORMATION_SCHEMA`
- Set operations using SQL
- Dynamic schema analysis

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

CREATE TABLE managers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    department VARCHAR(50),
    hire_date DATE,
    bonus DECIMAL(10,2)
);
```

Notice:

| employees  | managers   |
| ---------- | ---------- |
| id         | id         |
| name       | name       |
| department | department |
| salary     | bonus      |
| hire_date  | hire_date  |
| manager_id | -          |

Common columns are:

- id
- name
- department
- hire_date

---

# Method 1 (Most Common): Join `INFORMATION_SCHEMA.COLUMNS`

```sql
SELECT c1.COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS c1
JOIN INFORMATION_SCHEMA.COLUMNS c2
    ON c1.COLUMN_NAME = c2.COLUMN_NAME
WHERE c1.TABLE_SCHEMA = 'interview_db'
  AND c1.TABLE_NAME = 'employees'
  AND c2.TABLE_SCHEMA = 'interview_db'
  AND c2.TABLE_NAME = 'managers';
```

---

## Output

| COLUMN_NAME |
| ----------- |
| id          |
| name        |
| department  |
| hire_date   |

---

# Line-by-Line Explanation

```sql
FROM INFORMATION_SCHEMA.COLUMNS c1
```

Reads metadata for the first table.

---

```sql
JOIN INFORMATION_SCHEMA.COLUMNS c2
```

Reads metadata for the second table.

---

```sql
ON c1.COLUMN_NAME = c2.COLUMN_NAME
```

Matches columns having the same name.

---

```sql
TABLE_SCHEMA = 'interview_db'
```

Restricts the search to your database.

---

```sql
TABLE_NAME = 'employees'
```

Chooses the first table.

---

```sql
TABLE_NAME = 'managers'
```

Chooses the second table.

---

# Method 2: Using `INTERSECT` (MySQL 8.0.31+)

Recent MySQL versions support the `INTERSECT` set operator, making this query more concise.

```sql
SELECT COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'interview_db'
  AND TABLE_NAME = 'employees'

INTERSECT

SELECT COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'interview_db'
  AND TABLE_NAME = 'managers';
```

### Output

```
id
name
department
hire_date
```

---

# Finding Common Columns Across Multiple Tables

Example:

```sql
SELECT COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'interview_db'
  AND TABLE_NAME IN ('employees', 'managers', 'projects')
GROUP BY COLUMN_NAME
HAVING COUNT(DISTINCT TABLE_NAME) = 3;
```

This returns only the columns present in **all three tables**.

---

# Real-World Use Cases

Developers and DBAs use this technique when:

- Comparing two table structures
- Validating schemas before migration
- Building dynamic SQL
- Creating generic reporting tools
- Preparing joins automatically
- Checking compatibility between production and staging databases

---

# Why `INFORMATION_SCHEMA`?

`INFORMATION_SCHEMA` is a system schema that stores metadata about database objects.

Useful metadata tables include:

| Table                     | Purpose                   |
| ------------------------- | ------------------------- |
| `COLUMNS`                 | Column information        |
| `TABLES`                  | Table information         |
| `STATISTICS`              | Indexes                   |
| `KEY_COLUMN_USAGE`        | Primary and foreign keys  |
| `REFERENTIAL_CONSTRAINTS` | Foreign key relationships |

---

# Performance Considerations

- `INFORMATION_SCHEMA` queries are generally lightweight because they access metadata rather than table data.
- Always filter by `TABLE_SCHEMA` and `TABLE_NAME` to avoid scanning metadata for all databases.
- For very large environments with many schemas and tables, selective filtering improves performance.

---

# Common Pitfalls

### 1. Forgetting the schema name

```sql
WHERE TABLE_NAME='employees'
```

If multiple databases contain a table named `employees`, the results may include columns from unintended schemas.

Always include:

```sql
TABLE_SCHEMA='interview_db'
```

---

### 2. Comparing only names

Two columns may share the same name but have different data types.

Example:

```text
employees.id     INT
orders.id        BIGINT
```

If you also need matching data types, include them in the join condition:

```sql
SELECT c1.COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS c1
JOIN INFORMATION_SCHEMA.COLUMNS c2
  ON c1.COLUMN_NAME = c2.COLUMN_NAME
 AND c1.DATA_TYPE = c2.DATA_TYPE
WHERE c1.TABLE_SCHEMA = 'interview_db'
  AND c1.TABLE_NAME = 'employees'
  AND c2.TABLE_SCHEMA = 'interview_db'
  AND c2.TABLE_NAME = 'managers';
```

---

# Optimization Tips

- Use `TABLE_SCHEMA` and `TABLE_NAME` filters to reduce metadata scanning.
- If compatibility matters, compare additional attributes such as `DATA_TYPE`, `CHARACTER_MAXIMUM_LENGTH`, `IS_NULLABLE`, or `COLUMN_TYPE`.
- Prefer `INTERSECT` for readability on MySQL 8.0.31+; for older versions, the self-join approach is fully compatible.

---

# Related MySQL Features

- `INFORMATION_SCHEMA.COLUMNS`
- `GROUP BY`
- `HAVING`
- `INTERSECT` (MySQL 8.0.31+)
- Metadata queries

## Question 2. What are aliases and why are they used?

## Question 3. Can we use aliases in the `WHERE` clause?

## Question 4. What is a scalar subquery?

## Question 5. What is a row subquery?

## Question 6. Can we use subqueries inside the `FROM` clause?

## Question 7. What is a correlated subquery and when should you use it?

## Question 8. What is the difference between `=` and `IS` when comparing `NULL` values?

## Question 9. How do you count only distinct values from a column?

## Question 10. What is the use of the `EXISTS` keyword?

## Question 11. How do you calculate total and average in a single query?

## Question 12. What does `ROLLUP` do in a `GROUP BY` query?

## Question 13. What is the difference between `INNER QUERY` and `OUTER QUERY`?

## Question 14. How can you find the total number of columns in a table?

## Question 15. How can you find all tables containing a specific column name?

## Question 16. How can you find which queries are consuming the most resources?

## Question 17. What is a query execution plan?

## Question 18. How can you view an execution plan in MySQL Workbench?

## Question 19. What are derived indexes and why are they important?

## Question 20. How can you determine if MySQL used an index for a query?
