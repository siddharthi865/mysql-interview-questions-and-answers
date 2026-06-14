# Set 1

| S.No. | Question                                                                                                                                 |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is MySQL and how is it different from SQL?](#question-1-what-is-mysql-and-how-is-it-different-from-sql)                            |
| 2.    | [What are the main features of MySQL?](#question-2-what-are-the-main-features-of-mysql)                                                  |
| 3.    | [What are the different data types available in MySQL?](#question-3-what-are-the-different-data-types-available-in-mysql)                |
| 4.    | [What is the difference between `CHAR` and `VARCHAR` in MySQL?](#question-4-what-is-the-difference-between-char-and-varchar-in-mysql)    |
| 5.    | [What is the default port number of MySQL?](#question-5-what-is-the-default-port-number-of-mysql)                                        |
| 6.    | [Explain the difference between `PRIMARY KEY` and `UNIQUE KEY`](#question-6-explain-the-difference-between-primary-key-and-unique-key)   |
| 7.    | [What is the difference between `NULL` and `0` in MySQL?](#question-7-what-is-the-difference-between-null-and-0-in-mysql)                |
| 8.    | [What is a database, table, and record in MySQL?](#question-8-what-is-a-database-table-and-record-in-mysql)                              |
| 9.    | [What is a foreign key? Give an example](#question-9-what-is-a-foreign-key-give-an-example)                                              |
| 10.   | [What is the purpose of the `AUTO_INCREMENT` attribute?](#question-10-what-is-the-purpose-of-the-auto_increment-attribute)               |
| 11.   | [How do you create a database in MySQL?](#question-11-how-do-you-create-a-database-in-mysql)                                             |
| 12.   | [How can you show all databases and tables in MySQL?](#question-12-how-can-you-show-all-databases-and-tables-in-mysql)                   |
| 13.   | [What does the `DESC` command do in MySQL?](#question-13-what-does-the-desc-command-do-in-mysql)                                         |
| 14.   | [What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?](#question-14-what-is-the-difference-between-delete-truncate-and-drop) |
| 15.   | [What is the purpose of the `WHERE` clause?](#question-15-what-is-the-purpose-of-the-where-clause)                                       |
| 16.   | [How do you filter records using `LIKE` operator?](#question-16-how-do-you-filter-records-using-like-operator)                           |
| 17.   | [What is the use of the `ORDER BY` clause?](#question-17-what-is-the-use-of-the-order-by-clause)                                         |
| 18.   | [How do you get distinct records from a table?](#question-18-how-do-you-get-distinct-records-from-a-table)                               |
| 19.   | [What is the `LIMIT` clause used for?](#question-19-what-is-the-limit-clause-used-for)                                                   |
| 20.   | [How can you find the maximum and minimum value in a column?](#question-20-how-can-you-find-the-maximum-and-minimum-value-in-a-column)   |

## Question 1. What is MySQL and how is it different from SQL?

### ✅ What is MySQL?

**MySQL** is an **open-source Relational Database Management System (RDBMS)** used to store, manage, and retrieve structured data using tables.

- It is developed and maintained by **Oracle Corporation**
- It uses **SQL (Structured Query Language)** to interact with data
- It supports features like indexing, transactions, replication, stored procedures, and more

👉 In simple terms:

> **MySQL is the software (database system)** that stores data.

---

### 🧠 What is SQL?

**SQL (Structured Query Language)** is a **standard programming language** used to communicate with relational databases.

It is used to:

- Create databases and tables
- Insert, update, delete data
- Query data (SELECT)
- Manage permissions and transactions

👉 In simple terms:

> **SQL is the language used to talk to databases.**

---

# 🔥 Key Difference Between MySQL and SQL

| Feature    | SQL                                                    | MySQL                              |
| ---------- | ------------------------------------------------------ | ---------------------------------- |
| Type       | Query Language                                         | Database Management System         |
| Purpose    | Used to interact with databases                        | Stores and manages databases       |
| Nature     | Standard language (ISO/ANSI)                           | Software implementation            |
| Dependency | Works with all RDBMS (MySQL, PostgreSQL, Oracle, etc.) | Uses SQL as its query language     |
| Examples   | SELECT \* FROM employees;                              | MySQL Server executing SQL queries |

---

# 🧑‍💻 Real-World Analogy

Think of it like this:

- **SQL = English language**
- **MySQL = A company that understands English and stores books (data)**

You speak SQL (language) to MySQL (system) to get work done.

---

# 💻 Practical MySQL Example (Using SQL inside MySQL)

### Step 1: Create Database

```sql
CREATE DATABASE interview_db;
USE interview_db;
```

👉 Creates and selects a database.

---

### Step 2: Create Table

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE,
    manager_id INT
);
```

**Explanation:**

- `AUTO_INCREMENT` → Automatically generates unique IDs
- `VARCHAR` → Stores text
- `DECIMAL` → Stores precise salary values
- `DATE` → Stores date values

---

### Step 3: Insert Data

```sql
INSERT INTO employees (name, department, salary, hire_date, manager_id)
VALUES ('Alice', 'Engineering', 75000, '2020-01-15', NULL);
```

---

### Step 4: Query Data

```sql
SELECT * FROM employees;
```

---

# ⚙️ Important MySQL-Specific Insights (Interview Point)

- MySQL uses **SQL as its query language**, but adds its own extensions like:
  - `LIMIT`
  - `AUTO_INCREMENT`
  - `ENGINE=InnoDB`
  - JSON functions (modern MySQL 8+ feature)

- MySQL supports multiple storage engines:
  - InnoDB (default, supports transactions)
  - MyISAM (older, faster reads but no transactions)

- You can analyze performance using:

```sql
EXPLAIN SELECT * FROM employees;
```

---

# ⚠️ Common Pitfalls (Interview Gold)

- Confusing SQL with MySQL (very common mistake)
- Assuming SQL is database software
- Not knowing MySQL is just one implementation of SQL-based systems

---

# 🚀 Real-World Use Cases of MySQL

- Banking systems (transactional data)
- E-commerce platforms (orders, inventory)
- Web applications (user data storage)
- Logging and analytics systems

## Question 2. What are the main features of MySQL?

## Question 3. What are the different data types available in MySQL?

## Question 4. What is the difference between `CHAR` and `VARCHAR` in MySQL?

## Question 5. What is the default port number of MySQL?

## Question 6. Explain the difference between `PRIMARY KEY` and `UNIQUE KEY`

## Question 7. What is the difference between `NULL` and `0` in MySQL?

## Question 8. What is a database, table, and record in MySQL?

## Question 9. What is a foreign key? Give an example

## Question 10. What is the purpose of the `AUTO_INCREMENT` attribute?

## Question 11. How do you create a database in MySQL?

## Question 12. How can you show all databases and tables in MySQL?

## Question 13. What does the `DESC` command do in MySQL?

## Question 14. What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?

## Question 15. What is the purpose of the `WHERE` clause?

## Question 16. How do you filter records using `LIKE` operator?

## Question 17. What is the use of the `ORDER BY` clause?

## Question 18. How do you get distinct records from a table?

## Question 19. What is the `LIMIT` clause used for?

## Question 20. How can you find the maximum and minimum value in a column?
