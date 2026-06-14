# Set 9

| S.No. | Question                                                                                                                                                   |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How can you find all tables that contain a particular column name?](#question-1-how-can-you-find-all-tables-that-contain-a-particular-column-name)        |
| 2.    | [What are MySQL functions to handle strings (like `SUBSTRING`, `REPLACE`)?](#question-2-what-are-mysql-functions-to-handle-strings-like-substring-replace) |
| 3.    | [How can you convert uppercase letters to lowercase in MySQL?](#question-3-how-can-you-convert-uppercase-letters-to-lowercase-in-mysql)                    |
| 4.    | [How can you get the length of a string column?](#question-4-how-can-you-get-the-length-of-a-string-column)                                                |
| 5.    | [What are scalar functions in MySQL?](#question-5-what-are-scalar-functions-in-mysql)                                                                      |
| 6.    | [What is a derived table?](#question-6-what-is-a-derived-table)                                                                                            |
| 7.    | [What is the difference between a temporary table and a permanent one?](#question-7-what-is-the-difference-between-a-temporary-table-and-a-permanent-one)  |
| 8.    | [How do you drop a temporary table automatically after a session ends?](#question-8-how-do-you-drop-a-temporary-table-automatically-after-a-session-ends)  |
| 9.    | [How do you copy a table without copying data?](#question-9-how-do-you-copy-a-table-without-copying-data)                                                  |
| 10.   | [How do you copy a table including both structure and data?](#question-10-how-do-you-copy-a-table-including-both-structure-and-data)                       |
| 11.   | [What are the internal components of the InnoDB storage engine?](#question-11-what-are-the-internal-components-of-the-innodb-storage-engine)               |
| 12.   | [What is the purpose of the redo log and undo log in MySQL?](#question-12-what-is-the-purpose-of-the-redo-log-and-undo-log-in-mysql)                       |
| 13.   | [How does MySQL ensure data durability during a crash?](#question-13-how-does-mysql-ensure-data-durability-during-a-crash)                                 |
| 14.   | [What is the difference between logical and physical row storage?](#question-14-what-is-the-difference-between-logical-and-physical-row-storage)           |
| 15.   | [What is a B-Tree index and how does it work?](#question-15-what-is-a-b-tree-index-and-how-does-it-work)                                                   |
| 16.   | [What is a hash index and when is it used?](#question-16-what-is-a-hash-index-and-when-is-it-used)                                                         |
| 17.   | [What is index selectivity and why is it important?](#question-17-what-is-index-selectivity-and-why-is-it-important)                                       |
| 18.   | [What happens when an index becomes fragmented?](#question-18-what-happens-when-an-index-becomes-fragmented)                                               |
| 19.   | [What is the `OPTIMIZE TABLE` command used for?](#question-19-what-is-the-optimize-table-command-used-for)                                                 |
| 20.   | [What are the different types of joins supported by MySQL?](#question-20-what-are-the-different-types-of-joins-supported-by-mysql)                         |

## Question 1. How can you find all tables that contain a particular column name?

## Interview Answer: How can you find all tables that contain a particular column name?

### Definition

In MySQL, you can find all tables that contain a specific column by querying the **`INFORMATION_SCHEMA.COLUMNS`** system table.

`INFORMATION_SCHEMA` is a metadata database that stores information about all databases, tables, columns, indexes, constraints, views, and other database objects. It is commonly used by DBAs and developers to inspect database structures without accessing the actual table data.

---

# Basic Syntax

```sql
SELECT TABLE_SCHEMA,
       TABLE_NAME,
       COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE COLUMN_NAME = 'manager_id';
```

### Output Example

| TABLE_SCHEMA | TABLE_NAME | COLUMN_NAME |
| ------------ | ---------- | ----------- |
| interview_db | employees  | manager_id  |
| company_db   | managers   | manager_id  |

This query searches every database on the MySQL server.

---

# Search Within a Specific Database

Usually, interviewers expect you to search inside one database.

```sql
SELECT TABLE_NAME,
       COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'interview_db'
  AND COLUMN_NAME = 'manager_id';
```

### Result

| TABLE_NAME | COLUMN_NAME |
| ---------- | ----------- |
| employees  | manager_id  |

---

# Using the Sample Database

Suppose we have:

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

CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50),
    manager_id INT
);

CREATE TABLE projects (
    project_id INT PRIMARY KEY,
    project_name VARCHAR(100)
);
```

Now search for the column **`manager_id`**.

```sql
SELECT TABLE_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'interview_db'
  AND COLUMN_NAME = 'manager_id';
```

### Output

```
employees
departments
```

Because both tables contain that column.

---

# Explanation Line by Line

```sql
SELECT TABLE_NAME
```

Returns the table names.

```sql
FROM INFORMATION_SCHEMA.COLUMNS
```

Reads metadata from the system catalog.

```sql
WHERE TABLE_SCHEMA = 'interview_db'
```

Limits the search to one database.

```sql
AND COLUMN_NAME = 'manager_id';
```

Finds only columns with that exact name.

---

# Searching with Wildcards

Sometimes you don't remember the complete column name.

Example:

```sql
SELECT TABLE_NAME,
       COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'interview_db'
  AND COLUMN_NAME LIKE '%manager%';
```

Possible Output

| TABLE_NAME  | COLUMN_NAME  |
| ----------- | ------------ |
| employees   | manager_id   |
| departments | manager_id   |
| payroll     | manager_name |

---

# Find Every Occurrence Across All Databases

```sql
SELECT TABLE_SCHEMA,
       TABLE_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE COLUMN_NAME = 'employee_id';
```

Useful for DBAs managing multiple databases.

---

# Practical Real-World Uses

Developers frequently query `INFORMATION_SCHEMA.COLUMNS` to:

- Identify which tables use a specific column before making schema changes.
- Analyze database dependencies during refactoring.
- Verify whether a column exists before running migration scripts.
- Locate audit or foreign key columns (such as `created_by`, `updated_at`, or `manager_id`) across many tables.

---

# Performance Considerations

- `INFORMATION_SCHEMA` contains **metadata**, not actual table data, so these queries are generally much faster than scanning tables.
- Always filter by `TABLE_SCHEMA` when possible to reduce the amount of metadata searched.
- On servers with many databases and tables, searching the entire `INFORMATION_SCHEMA` can still take noticeable time.

---

# MySQL-Specific Notes

- `INFORMATION_SCHEMA.COLUMNS` is a read-only metadata view maintained by MySQL.
- The search is **case-insensitive** for column names on most systems, depending on the server's collation and configuration.
- You can retrieve additional metadata such as `DATA_TYPE`, `COLUMN_TYPE`, `IS_NULLABLE`, `COLUMN_DEFAULT`, and `ORDINAL_POSITION` from the same view.
- In modern MySQL (8.x and later), the data dictionary backs the metadata exposed through `INFORMATION_SCHEMA`, improving consistency and metadata management compared to older versions.

---

# Extended Example

To retrieve more information about matching columns:

```sql
SELECT TABLE_NAME,
       COLUMN_NAME,
       DATA_TYPE,
       COLUMN_TYPE,
       IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'interview_db'
  AND COLUMN_NAME = 'manager_id';
```

Sample Output

| TABLE_NAME  | COLUMN_NAME | DATA_TYPE | COLUMN_TYPE | IS_NULLABLE |
| ----------- | ----------- | --------- | ----------- | ----------- |
| employees   | manager_id  | int       | int         | YES         |
| departments | manager_id  | int       | int         | YES         |

This is particularly useful when checking whether columns with the same name also share compatible data types.

## Question 2. What are MySQL functions to handle strings (like `SUBSTRING`, `REPLACE`)?

## Question 3. How can you convert uppercase letters to lowercase in MySQL?

## Question 4. How can you get the length of a string column?

## Question 5. What are scalar functions in MySQL?

## Question 6. What is a derived table?

## Question 7. What is the difference between a temporary table and a permanent one?

## Question 8. How do you drop a temporary table automatically after a session ends?

## Question 9. How do you copy a table without copying data?

## Question 10. How do you copy a table including both structure and data?

## Question 11. What are the internal components of the InnoDB storage engine?

## Question 12. What is the purpose of the redo log and undo log in MySQL?

## Question 13. How does MySQL ensure data durability during a crash?

## Question 14. What is the difference between logical and physical row storage?

## Question 15. What is a B-Tree index and how does it work?

## Question 16. What is a hash index and when is it used?

## Question 17. What is index selectivity and why is it important?

## Question 18. What happens when an index becomes fragmented?

## Question 19. What is the `OPTIMIZE TABLE` command used for?

## Question 20. What are the different types of joins supported by MySQL?
