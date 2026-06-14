# Set 3

| S.No. | Question                                                                                                                                                               |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What are foreign key constraints and what happens on `CASCADE` delete/update?](#question-1-what-are-foreign-key-constraints-and-what-happens-on-cascade-deleteupdate) |
| 2.    | [What is the difference between `NOW()` and `CURRENT_TIMESTAMP()`?](#question-2-what-is-the-difference-between-now-and-current_timestamp)                              |
| 3.    | [How can you find duplicate records in a table?](#question-3-how-can-you-find-duplicate-records-in-a-table)                                                            |
| 4.    | [How can you delete duplicate rows while keeping one copy?](#question-4-how-can-you-delete-duplicate-rows-while-keeping-one-copy)                                      |
| 5.    | [What is the difference between `BETWEEN` and `IN` operators?](#question-5-what-is-the-difference-between-between-and-in-operators)                                    |
| 6.    | [How can you join more than two tables?](#question-6-how-can-you-join-more-than-two-tables)                                                                            |
| 7.    | [How can you select rows from one table that are not in another?](#question-7-how-can-you-select-rows-from-one-table-that-are-not-in-another)                          |
| 8.    | [What is a subquery?](#question-8-what-is-a-subquery)                                                                                                                  |
| 9.    | [What is the difference between correlated and non-correlated subqueries?](#question-9-what-is-the-difference-between-correlated-and-non-correlated-subqueries)        |
| 10.   | [Can we use a subquery inside a `JOIN`?](#question-10-can-we-use-a-subquery-inside-a-join)                                                                             |
| 11.   | [What is a stored procedure?](#question-11-what-is-a-stored-procedure)                                                                                                 |
| 12.   | [How do you create and call a stored procedure?](#question-12-how-do-you-create-and-call-a-stored-procedure)                                                           |
| 13.   | [What are stored functions? How are they different from procedures?](#question-13-what-are-stored-functions-how-are-they-different-from-procedures)                    |
| 14.   | [What are triggers in MySQL?](#question-14-what-are-triggers-in-mysql)                                                                                                 |
| 15.   | [When should you use a trigger?](#question-15-when-should-you-use-a-trigger)                                                                                           |
| 16.   | [How can you update multiple tables in a single query?](#question-16-how-can-you-update-multiple-tables-in-a-single-query)                                             |
| 17.   | [How can you copy data from one table to another?](#question-17-how-can-you-copy-data-from-one-table-to-another)                                                       |
| 18.   | [How can you find the second highest salary from an employee table?](#question-18-how-can-you-find-the-second-highest-salary-from-an-employee-table)                   |
| 19.   | [How can you find the nth highest salary?](#question-19-how-can-you-find-the-nth-highest-salary)                                                                       |
| 20.   | [What is the use of the `CASE` statement?](#question-20-what-is-the-use-of-the-case-statement)                                                                         |

## Question 1. What are foreign key constraints and what happens on `CASCADE` delete/update?

# Foreign Key Constraints in MySQL

A **Foreign Key (FK)** is a constraint used to enforce **referential integrity** between two tables.

- The **parent table** contains the primary key (or unique key).
- The **child table** contains the foreign key column that references the parent table.
- MySQL ensures that a child row cannot reference a non-existent parent row.

### Real-World Example

Suppose you have:

- `departments` table → stores department information.
- `employees` table → stores employees and their department.

Each employee must belong to a valid department.

---

## Creating a Foreign Key

```sql
CREATE DATABASE interview_db;
USE interview_db;

CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY AUTO_INCREMENT,
    employee_name VARCHAR(100),
    department_id INT,

    CONSTRAINT fk_department
    FOREIGN KEY (department_id)
    REFERENCES departments(department_id)
);
```

### Explanation

```sql
FOREIGN KEY (department_id)
```

- Indicates that `department_id` in `employees` is a foreign key.

```sql
REFERENCES departments(department_id)
```

- References the primary key of the `departments` table.

---

## Sample Data

```sql
INSERT INTO departments VALUES
(1, 'Engineering'),
(2, 'HR');

INSERT INTO employees (employee_name, department_id)
VALUES
('Alice', 1),
('Bob', 1),
('Charlie', 2);
```

Current Data:

### departments

| department_id | department_name |
| ------------- | --------------- |
| 1             | Engineering     |
| 2             | HR              |

### employees

| employee_id | employee_name | department_id |
| ----------- | ------------- | ------------- |
| 1           | Alice         | 1             |
| 2           | Bob           | 1             |
| 3           | Charlie       | 2             |

---

# What Happens Without CASCADE?

Suppose you try:

```sql
DELETE FROM departments
WHERE department_id = 1;
```

MySQL returns an error:

```text
Cannot delete or update a parent row:
a foreign key constraint fails
```

Why?

Because employees Alice and Bob still reference department 1.

This prevents orphan records.

---

# CASCADE DELETE

`ON DELETE CASCADE` tells MySQL:

> "When a parent row is deleted, automatically delete all related child rows."

## Example

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY AUTO_INCREMENT,
    employee_name VARCHAR(100),
    department_id INT,

    FOREIGN KEY (department_id)
    REFERENCES departments(department_id)
    ON DELETE CASCADE
);
```

---

### Before Delete

#### departments

| department_id | department_name |
| ------------- | --------------- |
| 1             | Engineering     |
| 2             | HR              |

#### employees

| employee_id | employee_name | department_id |
| ----------- | ------------- | ------------- |
| 1           | Alice         | 1             |
| 2           | Bob           | 1             |
| 3           | Charlie       | 2             |

---

### Delete Parent Row

```sql
DELETE FROM departments
WHERE department_id = 1;
```

---

### After Delete

#### departments

| department_id | department_name |
| ------------- | --------------- |
| 2             | HR              |

#### employees

| employee_id | employee_name | department_id |
| ----------- | ------------- | ------------- |
| 3           | Charlie       | 2             |

Notice:

- Engineering department deleted.
- Alice and Bob automatically deleted.

This is called **cascading delete**.

---

# CASCADE UPDATE

`ON UPDATE CASCADE` tells MySQL:

> "If the parent key changes, automatically update matching foreign keys in child tables."

## Example

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY AUTO_INCREMENT,
    employee_name VARCHAR(100),
    department_id INT,

    FOREIGN KEY (department_id)
    REFERENCES departments(department_id)
    ON UPDATE CASCADE
);
```

---

### Parent Table

```sql
SELECT * FROM departments;
```

| department_id | department_name |
| ------------- | --------------- |
| 1             | Engineering     |

---

### Child Table

| employee_id | employee_name | department_id |
| ----------- | ------------- | ------------- |
| 1           | Alice         | 1             |
| 2           | Bob           | 1             |

---

### Update Parent Key

```sql
UPDATE departments
SET department_id = 10
WHERE department_id = 1;
```

---

### Result

#### departments

| department_id | department_name |
| ------------- | --------------- |
| 10            | Engineering     |

#### employees

| employee_id | employee_name | department_id |
| ----------- | ------------- | ------------- |
| 1           | Alice         | 10            |
| 2           | Bob           | 10            |

The child rows are automatically updated.

---

# Complete Example with Both CASCADE Options

```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY AUTO_INCREMENT,
    employee_name VARCHAR(100),
    department_id INT,

    CONSTRAINT fk_department
    FOREIGN KEY (department_id)
    REFERENCES departments(department_id)
    ON DELETE CASCADE
    ON UPDATE CASCADE
);
```

---

# Other Foreign Key Actions in MySQL

| Action        | Behavior                                         |
| ------------- | ------------------------------------------------ |
| `CASCADE`     | Automatically delete/update child rows           |
| `RESTRICT`    | Prevent parent delete/update if child rows exist |
| `NO ACTION`   | Same as RESTRICT in MySQL                        |
| `SET NULL`    | Set child foreign key to NULL                    |
| `SET DEFAULT` | Not supported by MySQL                           |

Example:

```sql
FOREIGN KEY (department_id)
REFERENCES departments(department_id)
ON DELETE SET NULL
```

When a department is deleted:

```text
Alice -> department_id = NULL
Bob   -> department_id = NULL
```

instead of deleting employee records.

---

# Performance Considerations

### 1. Index Foreign Key Columns

MySQL automatically requires an index on foreign key columns (InnoDB creates one if needed).

```sql
SHOW INDEX FROM employees;
```

---

### 2. Be Careful with Large Cascades

```sql
DELETE FROM departments
WHERE department_id = 1;
```

might delete:

- Thousands of employees
- Millions of orders
- Related audit records

A single delete can trigger a large chain of cascading operations.

---

### 3. Use EXPLAIN Before Large Deletes

```sql
EXPLAIN DELETE
FROM departments
WHERE department_id = 1;
```

This helps understand the execution plan before performing large operations.

---

# Interview Answer (Short Version)

**Foreign Key Constraints** maintain referential integrity between parent and child tables by ensuring that child records reference valid parent records.

- **ON DELETE CASCADE**: When a parent row is deleted, all matching child rows are automatically deleted.
- **ON UPDATE CASCADE**: When a parent key value changes, all corresponding foreign key values in child tables are automatically updated.

They are commonly used in relationships such as **Departments → Employees**, **Customers → Orders**, and **Orders → Order_Items** to keep data consistent.

## Question 2. What is the difference between `NOW()` and `CURRENT_TIMESTAMP()`?

## Question 3. How can you find duplicate records in a table?

## Question 4. How can you delete duplicate rows while keeping one copy?

## Question 5. What is the difference between `BETWEEN` and `IN` operators?

## Question 6. How can you join more than two tables?

## Question 7. How can you select rows from one table that are not in another?

## Question 8. What is a subquery?

## Question 9. What is the difference between correlated and non-correlated subqueries?

## Question 10. Can we use a subquery inside a `JOIN`?

## Question 11. What is a stored procedure?

## Question 12. How do you create and call a stored procedure?

## Question 13. What are stored functions? How are they different from procedures?

## Question 14. What are triggers in MySQL?

## Question 15. When should you use a trigger?

## Question 16. How can you update multiple tables in a single query?

## Question 17. How can you copy data from one table to another?

## Question 18. How can you find the second highest salary from an employee table?

## Question 19. How can you find the nth highest salary?

## Question 20. What is the use of the `CASE` statement?
