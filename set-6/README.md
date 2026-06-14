# Set 6

| S.No. | Question                                                                                                                                             |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between SQL and MySQL Workbench?](#question-1-what-is-the-difference-between-sql-and-mysql-workbench)                        |
| 2.    | [How do you connect to a MySQL server using the command line?](#question-2-how-do-you-connect-to-a-mysql-server-using-the-command-line)              |
| 3.    | [What is the default username in MySQL after installation?](#question-3-what-is-the-default-username-in-mysql-after-installation)                    |
| 4.    | [How do you change a MySQL user password?](#question-4-how-do-you-change-a-mysql-user-password)                                                      |
| 5.    | [How can you list all tables inside a specific database?](#question-5-how-can-you-list-all-tables-inside-a-specific-database)                        |
| 6.    | [How can you check the version of MySQL installed?](#question-6-how-can-you-check-the-version-of-mysql-installed)                                    |
| 7.    | [How do you rename a table in MySQL?](#question-7-how-do-you-rename-a-table-in-mysql)                                                                |
| 8.    | [How do you rename a column in a table?](#question-8-how-do-you-rename-a-column-in-a-table)                                                          |
| 9.    | [How do you change the data type of a column?](#question-9-how-do-you-change-the-data-type-of-a-column)                                              |
| 10.   | [How do you add a new column to an existing table?](#question-10-how-do-you-add-a-new-column-to-an-existing-table)                                   |
| 11.   | [How do you remove a column from a table?](#question-11-how-do-you-remove-a-column-from-a-table)                                                     |
| 12.   | [What does the `SHOW CREATE TABLE` command do?](#question-12-what-does-the-show-create-table-command-do)                                             |
| 13.   | [How can you see the structure of a table?](#question-13-how-can-you-see-the-structure-of-a-table)                                                   |
| 14.   | [What is the difference between `INT` and `BIGINT` data types?](#question-14-what-is-the-difference-between-int-and-bigint-data-types)               |
| 15.   | [What is the difference between `FLOAT` and `DOUBLE`?](#question-15-what-is-the-difference-between-float-and-double)                                 |
| 16.   | [What is the default storage engine in MySQL 8?](#question-16-what-is-the-default-storage-engine-in-mysql-8)                                         |
| 17.   | [How do you check which storage engine a table uses?](#question-17-how-do-you-check-which-storage-engine-a-table-uses)                               |
| 18.   | [What is the purpose of the `DEFAULT` keyword in a table definition?](#question-18-what-is-the-purpose-of-the-default-keyword-in-a-table-definition) |
| 19.   | [What is the purpose of the `ENUM` data type?](#question-19-what-is-the-purpose-of-the-enum-data-type)                                               |
| 20.   | [What is the difference between `TIMESTAMP` and `DATETIME`?](#question-20-what-is-the-difference-between-timestamp-and-datetime)                     |

## Question 1. What is the difference between SQL and MySQL Workbench?

This is a very common MySQL interview question. The key is to understand that **SQL is a language**, while **MySQL Workbench is a software application (GUI tool)** used to work with MySQL databases.

---

# Difference Between SQL and MySQL Workbench

| SQL                                                                                                                        | MySQL Workbench                                                                          |
| -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| SQL stands for **Structured Query Language**.                                                                              | MySQL Workbench is a **graphical database management tool (GUI)** developed by Oracle.   |
| It is a **language** used to interact with relational databases.                                                           | It is a **desktop application** used to develop, administer, and manage MySQL databases. |
| SQL consists of commands like `SELECT`, `INSERT`, `UPDATE`, and `DELETE`.                                                  | Workbench provides an editor where you write and execute SQL queries.                    |
| SQL is standardized (ANSI/ISO) and supported by many databases such as MySQL, PostgreSQL, SQL Server, and Oracle Database. | MySQL Workbench is designed specifically for MySQL-compatible databases.                 |
| SQL can be executed through many tools, command-line clients, applications, or APIs.                                       | Workbench is one of those tools that executes SQL against a MySQL server.                |
| SQL itself has no graphical interface.                                                                                     | Workbench offers a graphical interface for database development and administration.      |

---

# Think of It Like This

Imagine you want to write a document:

- **SQL** = the English language
- **MySQL Workbench** = Microsoft Word

You write English in Microsoft Word.

Similarly,

You write **SQL queries** inside **MySQL Workbench**.

---

# What is SQL?

SQL is the language used to:

- Create databases
- Create tables
- Insert data
- Retrieve data
- Update records
- Delete records
- Control permissions
- Manage transactions

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

INSERT INTO employees (name, department, salary, hire_date, manager_id)
VALUES
('Alice','Engineering',75000,'2020-01-15',NULL),
('Bob','Engineering',80000,'2019-03-10',1),
('Charlie','HR',60000,'2021-07-22',NULL);
```

### Explanation

```sql
CREATE DATABASE interview_db;
```

Creates a new database.

```sql
USE interview_db;
```

Selects the database for subsequent operations.

```sql
CREATE TABLE employees (...);
```

Creates the `employees` table.

```sql
INSERT INTO employees (...)
VALUES (...);
```

Adds sample employee records.

These are **SQL commands**. They can be executed in:

- MySQL Workbench
- MySQL Command-Line Client
- Application code (Java, Python, PHP, etc.)
- Other database tools that support SQL

---

# What is MySQL Workbench?

MySQL Workbench is a graphical application that helps developers and database administrators interact with a MySQL server.

It provides features such as:

- SQL Editor
- Database Modeling (ER Diagrams)
- Query Execution
- Performance Analysis
- User Management
- Backup and Restore
- Server Administration
- Data Import and Export
- Schema Designer
- Visual Explain Plans

---

# Example Workflow in MySQL Workbench

1. Open MySQL Workbench.
2. Connect to your MySQL server.
3. Open a new SQL editor tab.
4. Type the following SQL:

```sql
SELECT * FROM employees;
```

5. Click the **Execute** (lightning bolt) button.
6. View the results in the result grid.

Here:

- `SELECT * FROM employees;` is **SQL**.
- MySQL Workbench is the tool that sends this query to the MySQL server and displays the results.

---

# SQL vs MySQL vs MySQL Workbench

Many beginners confuse these three terms.

| SQL                       | MySQL                             | MySQL Workbench                          |
| ------------------------- | --------------------------------- | ---------------------------------------- |
| Query language            | Database Management System (DBMS) | GUI tool for MySQL                       |
| Used to write queries     | Stores and manages data           | Used to manage MySQL databases visually  |
| Standard language         | Software                          | Application                              |
| Works with many databases | Executes SQL queries              | Lets users write and execute SQL queries |

---

# Real-World Example

Suppose a company stores employee information.

- **MySQL Server** stores the employee data.
- **SQL** is used to query or update that data.
- **MySQL Workbench** is the interface developers use to connect to the server, write SQL, and manage the database.

For example:

```sql
SELECT name, salary
FROM employees
WHERE department = 'Engineering';
```

**Line-by-line explanation:**

```sql
SELECT name, salary
```

Retrieves only the `name` and `salary` columns.

```sql
FROM employees
```

Reads data from the `employees` table.

```sql
WHERE department = 'Engineering';
```

Filters the results to employees in the Engineering department.

You can execute this query in MySQL Workbench, but the query itself is standard SQL.

---

# Key MySQL-Specific Features in Workbench

While SQL is the language, MySQL Workbench provides MySQL-specific capabilities, including:

- **Visual Explain** for analyzing query execution plans (built on MySQL's `EXPLAIN` functionality).
- Schema synchronization.
- Database reverse engineering.
- Forward engineering from ER diagrams.
- Server status monitoring.
- Performance dashboards.
- User and privilege management.

These features make development and administration easier but are not part of the SQL language itself.

---

# Common Interview Pitfalls

- Saying **SQL and MySQL Workbench are the same** — they are not.
- Thinking MySQL Workbench is the database — the database is the MySQL Server.
- Believing SQL only works in Workbench — SQL can be executed from many clients and applications.
- Confusing MySQL (the DBMS) with MySQL Workbench (the GUI tool).

---

# Interview Answer (2-Minute Version)

> **SQL (Structured Query Language)** is the standard language used to create, retrieve, update, and delete data in relational databases. It is supported by many database systems such as MySQL, PostgreSQL, SQL Server, and Oracle Database.
>
> **MySQL Workbench** is a graphical application developed by Oracle for working with MySQL databases. It provides an SQL editor, database modeling, administration tools, performance analysis, backup and restore, and visual schema management. SQL is the language you write, while MySQL Workbench is one of the tools you use to write and execute SQL against a MySQL server.

> **Latest MySQL Note:** MySQL Workbench complements the latest MySQL Server releases by providing features such as SQL editing, Visual Explain, data modeling, and server administration. The SQL syntax and server behavior are defined by the MySQL Server and documented in the official MySQL Reference Manual.

## Question 2. How do you connect to a MySQL server using the command line?

## Question 3. What is the default username in MySQL after installation?

## Question 4. How do you change a MySQL user password?

## Question 5. How can you list all tables inside a specific database?

## Question 6. How can you check the version of MySQL installed?

## Question 7. How do you rename a table in MySQL?

## Question 8. How do you rename a column in a table?

## Question 9. How do you change the data type of a column?

## Question 10. How do you add a new column to an existing table?

## Question 11. How do you remove a column from a table?

## Question 12. What does the `SHOW CREATE TABLE` command do?

## Question 13. How can you see the structure of a table?

## Question 14. What is the difference between `INT` and `BIGINT` data types?

## Question 15. What is the difference between `FLOAT` and `DOUBLE`?

## Question 16. What is the default storage engine in MySQL 8?

## Question 17. How do you check which storage engine a table uses?

## Question 18. What is the purpose of the `DEFAULT` keyword in a table definition?

## Question 19. What is the purpose of the `ENUM` data type?

## Question 20. What is the difference between `TIMESTAMP` and `DATETIME`?
