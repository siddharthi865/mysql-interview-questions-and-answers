# Set 22

| S.No. | Question                                                                                                                                                         |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between pessimistic and optimistic locking?](#question-1-what-is-the-difference-between-pessimistic-and-optimistic-locking)              |
| 2.    | [How do row-level locks work in InnoDB?](#question-2-how-do-row-level-locks-work-in-innodb)                                                                      |
| 3.    | [How can deadlocks occur in MySQL transactions?](#question-3-how-can-deadlocks-occur-in-mysql-transactions)                                                      |
| 4.    | [How do you detect and prevent deadlocks?](#question-4-how-do-you-detect-and-prevent-deadlocks)                                                                  |
| 5.    | [What is the purpose of the `SHOW ENGINE INNODB STATUS` command?](#question-5-what-is-the-purpose-of-the-show-engine-innodb-status-command)                      |
| 6.    | [How do you simulate a deadlock in MySQL for testing?](#question-6-how-do-you-simulate-a-deadlock-in-mysql-for-testing)                                          |
| 7.    | [How can you tune the transaction isolation level in MySQL?](#question-7-how-can-you-tune-the-transaction-isolation-level-in-mysql)                              |
| 8.    | [What does the `innodb_lock_wait_timeout` variable do?](#question-8-what-does-the-innodb_lock_wait_timeout-variable-do)                                          |
| 9.    | [How does MySQL handle phantom reads?](#question-9-how-does-mysql-handle-phantom-reads)                                                                          |
| 10.   | [What happens if a transaction is left open without a COMMIT or ROLLBACK?](#question-10-what-happens-if-a-transaction-is-left-open-without-a-commit-or-rollback) |
| 11.   | [What is the purpose of the `autocommit` mode?](#question-11-what-is-the-purpose-of-the-autocommit-mode)                                                         |
| 12.   | [How does MySQL handle implicit transactions?](#question-12-how-does-mysql-handle-implicit-transactions)                                                         |
| 13.   | [What are consistent reads in InnoDB?](#question-13-what-are-consistent-reads-in-innodb)                                                                         |
| 14.   | [How does multi-version concurrency control (MVCC) work in MySQL?](#question-14-how-does-multi-version-concurrency-control-mvcc-work-in-mysql)                   |
| 15.   | [How does MySQL manage undo logs?](#question-15-how-does-mysql-manage-undo-logs)                                                                                 |
| 16.   | [What is the difference between shared and exclusive locks?](#question-16-what-is-the-difference-between-shared-and-exclusive-locks)                             |
| 17.   | [What are gap locks in InnoDB?](#question-17-what-are-gap-locks-in-innodb)                                                                                       |
| 18.   | [What is next-key locking in MySQL?](#question-18-what-is-next-key-locking-in-mysql)                                                                             |
| 19.   | [How can you identify transactions causing long locks?](#question-19-how-can-you-identify-transactions-causing-long-locks)                                       |
| 20.   | [What does the `SHOW OPEN TABLES` command display?](#question-20-what-does-the-show-open-tables-command-display)                                                 |

## Question 1. What is the difference between pessimistic and optimistic locking?

## Pessimistic Locking vs Optimistic Locking in MySQL

This is one of the most common MySQL interview questions because it tests your understanding of **concurrency control** and **transactions**.

### Interview Definition

**Pessimistic Locking** assumes that conflicts between transactions are **likely to happen**, so it locks the data immediately to prevent other transactions from modifying it.

**Optimistic Locking** assumes that conflicts are **rare**, so it allows multiple transactions to read and modify data without locking it initially. Before saving changes, it checks whether another transaction has already modified the data.

---

# Comparison Table

| Feature       | Pessimistic Locking                | Optimistic Locking                          |
| ------------- | ---------------------------------- | ------------------------------------------- |
| Assumption    | Conflicts are common               | Conflicts are rare                          |
| Locking       | Locks rows immediately             | No lock during read                         |
| Performance   | Lower under heavy contention       | Better when conflicts are rare              |
| Blocking      | Yes                                | No                                          |
| Deadlocks     | Possible                           | Rare                                        |
| Best For      | Banking, Inventory, Ticket Booking | Web applications, CRUD systems              |
| MySQL Support | Built-in using row locks           | Implemented using version/timestamp columns |

---

# Pessimistic Locking

## How it Works

Imagine two users trying to withdraw money from the same account.

With pessimistic locking:

1. Transaction A locks the row.
2. Transaction B must wait.
3. Transaction A commits.
4. Lock is released.
5. Transaction B continues.

Only one transaction can modify the row at a time.

---

## Example Schema

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

---

## Example using `SELECT ... FOR UPDATE`

### Transaction A

```sql
START TRANSACTION;

SELECT *
FROM employees
WHERE id = 1
FOR UPDATE;

UPDATE employees
SET salary = salary + 5000
WHERE id = 1;

COMMIT;
```

---

### Transaction B

```sql
START TRANSACTION;

SELECT *
FROM employees
WHERE id = 1
FOR UPDATE;
```

Transaction B waits until Transaction A commits or rolls back.

---

## Line-by-Line Explanation

### `START TRANSACTION`

Begins a transaction.

---

### `SELECT ... FOR UPDATE`

```sql
SELECT *
FROM employees
WHERE id = 1
FOR UPDATE;
```

- Reads the row.
- Places an exclusive row lock (InnoDB).
- Prevents other transactions from acquiring conflicting locks on the same row until the transaction ends.

---

### `UPDATE`

```sql
UPDATE employees
SET salary = salary + 5000
WHERE id = 1;
```

Safely updates the locked row.

---

### `COMMIT`

Releases the lock.

---

# Advantages of Pessimistic Locking

- Prevents lost updates.
- Guarantees consistency.
- Ideal for high-conflict scenarios.

---

# Disadvantages

- Users may wait.
- Lower throughput.
- Can cause deadlocks.
- Long transactions reduce concurrency.

---

# Optimistic Locking

Instead of locking rows, maintain a **version** (or timestamp) column.

When updating:

1. Read the row and its version.
2. Modify locally.
3. Update only if the version is unchanged.
4. If no rows are updated, another transaction changed the row, and the application can retry or notify the user.

---

## Add Version Column

```sql
ALTER TABLE employees
ADD COLUMN version INT DEFAULT 1;
```

---

## Read Data

```sql
SELECT id,
       salary,
       version
FROM employees
WHERE id = 1;
```

Suppose the result is:

```text
salary = 75000
version = 3
```

---

## Update with Version Check

```sql
UPDATE employees
SET salary = 80000,
    version = version + 1
WHERE id = 1
  AND version = 3;
```

---

## What Happens?

If another transaction has already updated the row:

```text
version becomes 4
```

Then:

```sql
WHERE version = 3
```

matches no rows.

Affected rows:

```text
0
```

The application knows someone else updated the row first.

---

# Line-by-Line Explanation

```sql
UPDATE employees
SET salary = 80000,
    version = version + 1
```

Updates the salary and increments the version.

---

```sql
WHERE id = 1
AND version = 3;
```

Ensures the row hasn't changed since it was read.

---

# Advantages of Optimistic Locking

- No blocking during reads.
- Better scalability.
- Higher concurrency.
- Excellent for web applications where conflicts are infrequent.

---

# Disadvantages

- Conflicts are detected only when updating.
- Failed updates must be retried or handled by the application.
- Not suitable for systems with frequent concurrent updates.

---

# Real-World Examples

### Pessimistic Locking

- Banking transactions
- Inventory deduction
- Flight seat booking
- Hotel room reservation
- Ticket booking

These scenarios require preventing concurrent modifications immediately.

---

### Optimistic Locking

- Employee profile updates
- Customer information forms
- Blog posts
- Product descriptions
- User preferences

Here, simultaneous edits are relatively uncommon, making retries acceptable.

---

# Performance Considerations

### Pessimistic Locking

- Uses **InnoDB row-level locks**.
- Transactions waiting on locks can increase latency.
- Long-running transactions hold locks longer, reducing concurrency.
- Use `EXPLAIN` to ensure `SELECT ... FOR UPDATE` uses indexes efficiently; otherwise, more rows than expected may be locked.

### Optimistic Locking

- No database locks during reads.
- Higher throughput under low-conflict workloads.
- Requires application logic to detect conflicts and retry updates.
- Works well in distributed or stateless application architectures.

---

# Key MySQL Features

### `FOR UPDATE`

```sql
SELECT ...
FOR UPDATE;
```

Locks selected rows for update within a transaction.

---

### `FOR SHARE`

```sql
SELECT ...
FOR SHARE;
```

Acquires a shared lock, allowing other transactions to read the rows but preventing conflicting updates until the transaction completes.

---

### Transaction Control

```sql
START TRANSACTION;
COMMIT;
ROLLBACK;
```

Essential commands for managing locks and ensuring ACID properties.

---

# Common Pitfalls

### Pessimistic Locking

- Forgetting `COMMIT` or `ROLLBACK` leaves locks held until the transaction ends.
- Missing indexes on `FOR UPDATE` queries can cause broader locking and reduced concurrency.
- Long-running transactions increase the likelihood of lock waits and deadlocks.

### Optimistic Locking

- Omitting the version (or timestamp) check can result in lost updates.
- Ignoring an update that affects **0 rows** means missed concurrency conflicts.
- Not implementing retry or conflict-resolution logic in the application.

---

# Interview Summary

- **Pessimistic locking** acquires locks before updates, preventing concurrent modifications. It is safest for high-contention, business-critical operations such as banking or inventory management.
- **Optimistic locking** avoids locking during reads and detects conflicts at update time using a version or timestamp column. It offers better scalability for applications where concurrent update conflicts are uncommon.
- In MySQL, pessimistic locking is natively supported with statements like `SELECT ... FOR UPDATE` and `SELECT ... FOR SHARE` (InnoDB). Optimistic locking is implemented at the application level using version or timestamp checks.

## Question 2. How do row-level locks work in InnoDB?

## Question 3. How can deadlocks occur in MySQL transactions?

## Question 4. How do you detect and prevent deadlocks?

## Question 5. What is the purpose of the `SHOW ENGINE INNODB STATUS` command?

## Question 6. How do you simulate a deadlock in MySQL for testing?

## Question 7. How can you tune the transaction isolation level in MySQL?

## Question 8. What does the `innodb_lock_wait_timeout` variable do?

## Question 9. How does MySQL handle phantom reads?

## Question 10. What happens if a transaction is left open without a COMMIT or ROLLBACK?

## Question 11. What is the purpose of the `autocommit` mode?

## Question 12. How does MySQL handle implicit transactions?

## Question 13. What are consistent reads in InnoDB?

## Question 14. How does multi-version concurrency control (MVCC) work in MySQL?

## Question 15. How does MySQL manage undo logs?

## Question 16. What is the difference between shared and exclusive locks?

## Question 17. What are gap locks in InnoDB?

## Question 18. What is next-key locking in MySQL?

## Question 19. How can you identify transactions causing long locks?

## Question 20. What does the `SHOW OPEN TABLES` command display?
