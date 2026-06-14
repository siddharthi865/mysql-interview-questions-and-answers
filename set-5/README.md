# Set 5

| S.No. | Question                                                                                                                                                                    |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What are MySQL locks? Explain shared and exclusive locks](#question-1-what-are-mysql-locks-explain-shared-and-exclusive-locks)                                             |
| 2.    | [What are isolation levels in MySQL transactions?](#question-2-what-are-isolation-levels-in-mysql-transactions)                                                             |
| 3.    | [Explain `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, and `SERIALIZABLE`](#question-3-explain-read-uncommitted-read-committed-repeatable-read-and-serializable) |
| 4.    | [How can you identify slow queries using logs?](#question-4-how-can-you-identify-slow-queries-using-logs)                                                                   |
| 5.    | [What is the purpose of the `INFORMATION_SCHEMA` database?](#question-5-what-is-the-purpose-of-the-information_schema-database)                                             |
| 6.    | [What are stored routines and their advantages?](#question-6-what-are-stored-routines-and-their-advantages)                                                                 |
| 7.    | [How can you schedule tasks in MySQL?](#question-7-how-can-you-schedule-tasks-in-mysql)                                                                                     |
| 8.    | [How can you handle large datasets efficiently in MySQL?](#question-8-how-can-you-handle-large-datasets-efficiently-in-mysql)                                               |
| 9.    | [How can you implement pagination in MySQL efficiently?](#question-9-how-can-you-implement-pagination-in-mysql-efficiently)                                                 |
| 10.   | [What is the difference between logical replication and physical replication?](#question-10-what-is-the-difference-between-logical-replication-and-physical-replication)    |
| 11.   | [How can you replicate a MySQL database to another server?](#question-11-how-can-you-replicate-a-mysql-database-to-another-server)                                          |
| 12.   | [What is binlog in MySQL and what is its use?](#question-12-what-is-binlog-in-mysql-and-what-is-its-use)                                                                    |
| 13.   | [What are temporary tables vs derived tables?](#question-13-what-are-temporary-tables-vs-derived-tables)                                                                    |
| 14.   | [What is a materialized view and is it supported in MySQL?](#question-14-what-is-a-materialized-view-and-is-it-supported-in-mysql)                                          |
| 15.   | [How do you enforce referential integrity in MySQL?](#question-15-how-do-you-enforce-referential-integrity-in-mysql)                                                        |
| 16.   | [What is JSON data type in MySQL 8.0?](#question-16-what-is-json-data-type-in-mysql-80)                                                                                     |
| 17.   | [How do you query JSON data in MySQL?](#question-17-how-do-you-query-json-data-in-mysql)                                                                                    |
| 18.   | [What is the difference between MySQL 5.7 and MySQL 8.0?](#question-18-what-is-the-difference-between-mysql-57-and-mysql-80)                                                |
| 19.   | [How does MySQL handle concurrency control?](#question-19-how-does-mysql-handle-concurrency-control)                                                                        |
| 20.   | [How do you secure a MySQL database (user privileges, encryption, etc.)?](#question-20-how-do-you-secure-a-mysql-database-user-privileges-encryption-etc)                   |

## Question 1. What are MySQL locks? Explain shared and exclusive locks

# What Are MySQL Locks? Explain Shared and Exclusive Locks

## Definition

**MySQL locks** are mechanisms used to control concurrent access to data and ensure **data consistency, integrity, and isolation** when multiple transactions access the same rows or tables simultaneously.

Locks prevent problems such as:

- Dirty reads
- Lost updates
- Inconsistent data
- Race conditions

In **InnoDB** (the default storage engine in modern MySQL), locking is tightly integrated with transactions and follows the ACID principles.

---

# Why Are Locks Needed?

Imagine two employees updating the same salary record at the same time:

| Transaction A           | Transaction B           |
| ----------------------- | ----------------------- |
| Reads salary = 50000    | Reads salary = 50000    |
| Updates salary to 55000 | Updates salary to 60000 |
| Commits                 | Commits                 |

Without locking, one update may overwrite the other, causing a **lost update problem**.

Locks ensure that only one transaction modifies the row at a time.

---

# Types of Locks in MySQL

The most important row-level locks are:

1. **Shared Lock (S Lock)**
2. **Exclusive Lock (X Lock)**

These are commonly used by InnoDB during transactions.

---

# 1. Shared Lock (S Lock)

## Definition

A **Shared Lock** allows a transaction to **read** a row but **not modify** it.

Multiple transactions can hold shared locks on the same row simultaneously.

### Characteristics

✅ Multiple readers allowed

✅ Prevents updates/deletes

❌ Does not allow exclusive lock until released

---

## Syntax

```sql
SELECT *
FROM employees
WHERE id = 1
LOCK IN SHARE MODE;
```

**MySQL 8.0+ Preferred Syntax**

```sql
SELECT *
FROM employees
WHERE id = 1
FOR SHARE;
```

`FOR SHARE` is the modern syntax recommended in recent MySQL versions.

---

## Example

### Transaction A

```sql
START TRANSACTION;

SELECT *
FROM employees
WHERE id = 1
FOR SHARE;
```

### What Happens?

MySQL places a **shared lock** on employee ID 1.

Other transactions:

```sql
SELECT * FROM employees WHERE id = 1;
```

✅ Allowed

```sql
SELECT * FROM employees
WHERE id = 1
FOR SHARE;
```

✅ Allowed

```sql
UPDATE employees
SET salary = 90000
WHERE id = 1;
```

❌ Blocked until Transaction A commits or rolls back.

---

## Real-World Example

A banking system verifies account balance before processing a transfer:

```sql
START TRANSACTION;

SELECT balance
FROM accounts
WHERE account_id = 100
FOR SHARE;
```

This ensures the account data isn't modified while validation is taking place.

---

# 2. Exclusive Lock (X Lock)

## Definition

An **Exclusive Lock** allows a transaction to:

- Read the row
- Update the row
- Delete the row

While preventing all other transactions from reading with lock or modifying that row.

---

## Characteristics

✅ One writer only

❌ No other shared locks allowed

❌ No other exclusive locks allowed

---

## Syntax

```sql
SELECT *
FROM employees
WHERE id = 1
FOR UPDATE;
```

`FOR UPDATE` acquires an exclusive lock.

---

## Example

### Transaction A

```sql
START TRANSACTION;

SELECT *
FROM employees
WHERE id = 1
FOR UPDATE;
```

### What Happens?

Row ID 1 receives an exclusive lock.

### Transaction B

```sql
UPDATE employees
SET salary = 95000
WHERE id = 1;
```

❌ Blocked

---

```sql
SELECT *
FROM employees
WHERE id = 1
FOR UPDATE;
```

❌ Blocked

---

```sql
SELECT *
FROM employees
WHERE id = 1
FOR SHARE;
```

❌ Blocked

---

After Transaction A:

```sql
COMMIT;
```

Transaction B can proceed.

---

# Lock Compatibility Matrix

| Existing Lock  | Shared Lock Request | Exclusive Lock Request |
| -------------- | ------------------- | ---------------------- |
| Shared Lock    | ✅ Allowed          | ❌ Blocked             |
| Exclusive Lock | ❌ Blocked          | ❌ Blocked             |

### Interpretation

- Many transactions can hold Shared Locks together.
- Only one transaction can hold an Exclusive Lock.
- Exclusive Lock blocks everyone else.

---

# Sample Example Using Employees Table

## Setup

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

INSERT INTO employees
(name, department, salary, hire_date, manager_id)
VALUES
('Alice', 'Engineering', 75000, '2020-01-15', NULL),
('Bob', 'Engineering', 80000, '2019-03-10', 1),
('Charlie', 'HR', 60000, '2021-07-22', NULL);
```

---

## Shared Lock Example

### Session 1

```sql
START TRANSACTION;

SELECT *
FROM employees
WHERE id = 1
FOR SHARE;
```

### Session 2

```sql
UPDATE employees
SET salary = 85000
WHERE id = 1;
```

Result:

```text
Waiting for lock...
```

---

## Exclusive Lock Example

### Session 1

```sql
START TRANSACTION;

SELECT *
FROM employees
WHERE id = 1
FOR UPDATE;
```

### Session 2

```sql
SELECT *
FROM employees
WHERE id = 1
FOR SHARE;
```

Result:

```text
Waiting for lock...
```

---

# How InnoDB Uses Locks Automatically

You don't always need to explicitly request locks.

When you execute:

```sql
UPDATE employees
SET salary = 85000
WHERE id = 1;
```

InnoDB automatically acquires:

```text
Exclusive Lock (X Lock)
```

on the affected row.

Similarly:

```sql
DELETE FROM employees
WHERE id = 1;
```

also acquires an Exclusive Lock.

---

# Viewing Lock Information

MySQL provides lock information through Performance Schema.

```sql
SELECT *
FROM performance_schema.data_locks;
```

Useful columns include:

- ENGINE
- OBJECT_NAME
- INDEX_NAME
- LOCK_TYPE
- LOCK_MODE
- LOCK_STATUS

This helps diagnose lock waits and blocking transactions.

---

# Performance Considerations

## 1. Keep Transactions Short

Bad:

```sql
START TRANSACTION;

SELECT * FROM employees FOR UPDATE;

/* user goes for lunch */

COMMIT;
```

Locks remain held for a long time.

---

## 2. Use Proper Indexes

Without indexes:

```sql
SELECT *
FROM employees
WHERE department = 'Engineering'
FOR UPDATE;
```

MySQL may lock many rows while scanning.

Create indexes:

```sql
CREATE INDEX idx_department
ON employees(department);
```

This reduces unnecessary locking.

---

## 3. Commit Quickly

```sql
COMMIT;
```

or

```sql
ROLLBACK;
```

releases locks immediately.

---

# Interview Tip

A common interview answer is:

> A Shared Lock (S Lock) allows multiple transactions to read the same row concurrently but prevents modifications. An Exclusive Lock (X Lock) allows a transaction to read and modify a row while blocking all other shared and exclusive locks. In InnoDB, `FOR SHARE` acquires a shared lock and `FOR UPDATE` acquires an exclusive lock.

## Question 2. What are isolation levels in MySQL transactions?

## Question 3. Explain `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, and `SERIALIZABLE`

## Question 4. How can you identify slow queries using logs?

## Question 5. What is the purpose of the `INFORMATION_SCHEMA` database?

## Question 6. What are stored routines and their advantages?

## Question 7. How can you schedule tasks in MySQL?

## Question 8. How can you handle large datasets efficiently in MySQL?

## Question 9. How can you implement pagination in MySQL efficiently?

## Question 10. What is the difference between logical replication and physical replication?

## Question 11. How can you replicate a MySQL database to another server?

## Question 12. What is binlog in MySQL and what is its use?

## Question 13. What are temporary tables vs derived tables?

## Question 14. What is a materialized view and is it supported in MySQL?

## Question 15. How do you enforce referential integrity in MySQL?

## Question 16. What is JSON data type in MySQL 8.0?

## Question 17. How do you query JSON data in MySQL?

## Question 18. What is the difference between MySQL 5.7 and MySQL 8.0?

## Question 19. How does MySQL handle concurrency control?

## Question 20. How do you secure a MySQL database (user privileges, encryption, etc.)?
