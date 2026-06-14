# Set 16

| S.No. | Question                                                                                                                                                              |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between SQL, MySQL, and MariaDB?](#question-1-what-is-the-difference-between-sql-mysql-and-mariadb)                                           |
| 2.    | [What are some popular GUI tools for managing MySQL databases?](#question-2-what-are-some-popular-gui-tools-for-managing-mysql-databases)                             |
| 3.    | [What are system databases in MySQL (like `mysql`, `information_schema`, etc.)?](#question-3-what-are-system-databases-in-mysql-like-mysql-information_schema-etc)    |
| 4.    | [What is the difference between `information_schema` and `performance_schema`?](#question-4-what-is-the-difference-between-information_schema-and-performance_schema) |
| 5.    | [What is a catalog in MySQL?](#question-5-what-is-a-catalog-in-mysql)                                                                                                 |
| 6.    | [What are the different types of MySQL statements?](#question-6-what-are-the-different-types-of-mysql-statements)                                                     |
| 7.    | [What is a constraint? List types of constraints](#question-7-what-is-a-constraint-list-types-of-constraints)                                                         |
| 8.    | [How do you define a NOT NULL constraint during table creation?](#question-8-how-do-you-define-a-not-null-constraint-during-table-creation)                           |
| 9.    | [How can you add or remove constraints after table creation?](#question-9-how-can-you-add-or-remove-constraints-after-table-creation)                                 |
| 10.   | [What happens if you try to violate a UNIQUE constraint?](#question-10-what-happens-if-you-try-to-violate-a-unique-constraint)                                        |
| 11.   | [Can a table have multiple primary keys?](#question-11-can-a-table-have-multiple-primary-keys)                                                                        |
| 12.   | [Can a primary key be composite?](#question-12-can-a-primary-key-be-composite)                                                                                        |
| 13.   | [How do you define a composite key?](#question-13-how-do-you-define-a-composite-key)                                                                                  |
| 14.   | [What is the difference between primary key and foreign key constraints?](#question-14-what-is-the-difference-between-primary-key-and-foreign-key-constraints)        |
| 15.   | [What is the purpose of the `CHECK` constraint?](#question-15-what-is-the-purpose-of-the-check-constraint)                                                            |
| 16.   | [How can you list all constraints applied to a table?](#question-16-how-can-you-list-all-constraints-applied-to-a-table)                                              |
| 17.   | [What are the different types of SQL joins supported in MySQL?](#question-17-what-are-the-different-types-of-sql-joins-supported-in-mysql)                            |
| 18.   | [Explain with an example how a `LEFT JOIN` differs from an `INNER JOIN`](#question-18-explain-with-an-example-how-a-left-join-differs-from-an-inner-join)             |
| 19.   | [How can you combine rows from two tables that have no related column?](#question-19-how-can-you-combine-rows-from-two-tables-that-have-no-related-column)            |
| 20.   | [What is a `NATURAL JOIN` and how is it different from `USING`?](#question-20-what-is-a-natural-join-and-how-is-it-different-from-using)                              |

## Question 1. What is the difference between SQL, MySQL, and MariaDB?

## Difference Between SQL, MySQL, and MariaDB (Interview Answer)

This is one of the most common interview questions. The key is to understand that **SQL is a language**, while **MySQL and MariaDB are database management systems (DBMSs)** that implement SQL.

| Feature       | SQL                             | MySQL                                         | MariaDB                                       |
| ------------- | ------------------------------- | --------------------------------------------- | --------------------------------------------- |
| Type          | Query language                  | Relational Database Management System (RDBMS) | Open-source fork of MySQL                     |
| Purpose       | Used to interact with databases | Stores, manages, and retrieves data           | Stores, manages, and retrieves data           |
| Developed By  | ANSI/ISO Standard               | Originally MySQL AB, later Oracle             | MariaDB Foundation (forked by MySQL creators) |
| Open Source   | Standard specification          | Community + Enterprise editions               | Fully open source                             |
| Stores Data?  | ❌ No                           | ✅ Yes                                        | ✅ Yes                                        |
| SQL Support   | Defines syntax                  | Implements SQL                                | Implements SQL                                |
| Compatibility | Works across many databases     | MySQL-specific implementation                 | Highly compatible with MySQL                  |

---

# 1. What is SQL?

**SQL (Structured Query Language)** is the **standard language used to communicate with relational databases**.

It is **not a database**.

SQL allows you to:

- Create databases and tables
- Insert data
- Retrieve data
- Update records
- Delete records
- Manage users and permissions

Example:

```sql
SELECT name, salary
FROM employees
WHERE department = 'Engineering';
```

### Explanation

```sql
SELECT name, salary
```

Returns only the `name` and `salary` columns.

```sql
FROM employees
```

Reads data from the `employees` table.

```sql
WHERE department = 'Engineering';
```

Filters rows belonging to the Engineering department.

SQL syntax is standardized by ANSI/ISO, although each database vendor adds its own extensions.

---

# 2. What is MySQL?

**MySQL** is an **open-source Relational Database Management System (RDBMS)** that uses SQL as its query language.

It provides:

- Data storage
- Query execution
- Indexing
- Transactions
- Replication
- Backup
- Security
- User management

MySQL supports storage engines such as:

- InnoDB (default)
- MEMORY
- MyISAM (legacy)
- CSV
- ARCHIVE

Example:

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

### Explanation

```sql
CREATE DATABASE interview_db;
```

Creates a new database.

```sql
USE interview_db;
```

Makes it the active database.

```sql
CREATE TABLE employees (...);
```

Creates the `employees` table.

MySQL stores the data physically on disk and executes SQL statements against it.

---

# 3. What is MariaDB?

**MariaDB** is an **open-source fork of MySQL**, created by the original MySQL developers after Oracle acquired MySQL.

The goal was to maintain a community-driven, fully open-source database while staying largely compatible with MySQL.

MariaDB supports:

- SQL
- Transactions
- Replication
- Views
- Stored procedures
- Triggers
- Window functions
- Common Table Expressions (CTEs)
- JSON-related functions (implementation differs from MySQL)

Most MySQL applications can run on MariaDB with little or no code changes.

Example:

```sql
SELECT *
FROM employees
ORDER BY salary DESC;
```

This query works in both MySQL and MariaDB.

---

# Sample Schema

We'll use the following schema for examples:

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

-- Sample data
INSERT INTO employees (name, department, salary, hire_date, manager_id) VALUES
('Alice', 'Engineering', 75000, '2020-01-15', NULL),
('Bob', 'Engineering', 80000, '2019-03-10', 1),
('Charlie', 'HR', 60000, '2021-07-22', NULL);
```

### Line-by-line Explanation

```sql
CREATE DATABASE interview_db;
```

Creates a new database.

```sql
USE interview_db;
```

Selects the database for subsequent commands.

```sql
CREATE TABLE employees (...);
```

Creates an `employees` table with:

- `id` as an auto-incrementing primary key
- `name` and `department` as text fields
- `salary` as a decimal value
- `hire_date` as a date
- `manager_id` to reference a manager (if applicable)

```sql
INSERT INTO employees (...)
```

Inserts three sample employee records into the table.

---

# Real-World Analogy

Think of a library:

- **SQL** is the language you use to ask the librarian for books.
- **MySQL** is the library building that stores and manages the books.
- **MariaDB** is another library built from the same original design as MySQL, with additional community-driven improvements while remaining very similar.

---

# MySQL vs MariaDB

| Feature             | MySQL                                                      | MariaDB                                                                                                       |
| ------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Creator             | Oracle                                                     | Original MySQL developers / MariaDB Foundation                                                                |
| License             | Community + Commercial editions                            | Fully open source                                                                                             |
| Compatibility       | Native MySQL                                               | Highly compatible with MySQL                                                                                  |
| Storage Engines     | InnoDB (default), MyISAM, MEMORY, etc.                     | InnoDB (or XtraDB in older versions), Aria (default replacement for MyISAM), MyRocks (optional), MEMORY, etc. |
| Performance         | Excellent general-purpose performance                      | Often includes additional optimizer enhancements and extra storage engine options                             |
| JSON                | Native `JSON` data type with binary storage and validation | `JSON` is an alias for `LONGTEXT` with validation functions rather than MySQL's native binary JSON storage    |
| Enterprise Features | Oracle Enterprise Edition                                  | Community-driven features                                                                                     |

---

# Query Optimization (`EXPLAIN`)

In MySQL, you can inspect how a query will be executed using `EXPLAIN`:

```sql
EXPLAIN
SELECT *
FROM employees
WHERE department = 'Engineering';
```

### Why use `EXPLAIN`?

- Shows whether indexes are used.
- Identifies full table scans.
- Helps optimize slow queries.
- Displays the execution plan before running the query.

---

# Common Interview Pitfalls

1. **Saying SQL is a database**
   - Incorrect. SQL is a language.

2. **Saying MySQL and SQL are the same**
   - Incorrect. MySQL is software that implements SQL.

3. **Thinking MariaDB is a different language**
   - Incorrect. MariaDB also uses SQL.

4. **Assuming MySQL and MariaDB are identical**
   - They share a common origin and remain largely compatible, but they have diverged over time with different features, optimizations, storage engines, and implementation details (such as JSON handling).

---

# Best Practices

- Learn **standard SQL** first, as it is portable across database systems.
- Understand **MySQL-specific features** (e.g., storage engines, `EXPLAIN`, replication, binary JSON).
- Be aware of compatibility differences if migrating between MySQL and MariaDB.
- Use `EXPLAIN` to analyze query performance before optimizing.

---

# Latest MySQL Notes

According to the latest MySQL documentation:

- MySQL supports the SQL standard while also providing MySQL-specific extensions.
- The default storage engine is **InnoDB**, which supports ACID-compliant transactions, row-level locking, and foreign keys.
- MySQL provides a native `JSON` data type with efficient binary storage and indexing support through generated columns and functional indexes.

## Question 2. What are some popular GUI tools for managing MySQL databases?

## Question 3. What are system databases in MySQL (like `mysql`, `information_schema`, etc.)?

## Question 4. What is the difference between `information_schema` and `performance_schema`?

## Question 5. What is a catalog in MySQL?

## Question 6. What are the different types of MySQL statements?

## Question 7. What is a constraint? List types of constraints

## Question 8. How do you define a NOT NULL constraint during table creation?

## Question 9. How can you add or remove constraints after table creation?

## Question 10. What happens if you try to violate a UNIQUE constraint?

## Question 11. Can a table have multiple primary keys?

## Question 12. Can a primary key be composite?

## Question 13. How do you define a composite key?

## Question 14. What is the difference between primary key and foreign key constraints?

## Question 15. What is the purpose of the `CHECK` constraint?

## Question 16. How can you list all constraints applied to a table?

## Question 17. What are the different types of SQL joins supported in MySQL?

## Question 18. Explain with an example how a `LEFT JOIN` differs from an `INNER JOIN`

## Question 19. How can you combine rows from two tables that have no related column?

## Question 20. What is a `NATURAL JOIN` and how is it different from `USING`?
