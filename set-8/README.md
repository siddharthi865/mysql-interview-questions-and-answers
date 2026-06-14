# Set 8

| S.No. | Question                                                                                                                                                                        |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the use of the `ANY` and `ALL` operators?](#question-1-what-is-the-use-of-the-any-and-all-operators)                                                                   |
| 2.    | [What is the difference between `EXISTS` and `IN`?](#question-2-what-is-the-difference-between-exists-and-in)                                                                   |
| 3.    | [How do you use `DISTINCT` with multiple columns?](#question-3-how-do-you-use-distinct-with-multiple-columns)                                                                   |
| 4.    | [How can you retrieve only even or odd IDs from a table?](#question-4-how-can-you-retrieve-only-even-or-odd-ids-from-a-table)                                                   |
| 5.    | [How do you calculate total salary by department using `GROUP BY`?](#question-5-how-do-you-calculate-total-salary-by-department-using-group-by)                                 |
| 6.    | [How do you get the total count of all employees and the average salary together?](#question-6-how-do-you-get-the-total-count-of-all-employees-and-the-average-salary-together) |
| 7.    | [How do you select top N rows from a table?](#question-7-how-do-you-select-top-n-rows-from-a-table)                                                                             |
| 8.    | [How do you join a table with itself using aliases?](#question-8-how-do-you-join-a-table-with-itself-using-aliases)                                                             |
| 9.    | [How can you list employees who have no managers (i.e., foreign key is `NULL`)?](#question-9-how-can-you-list-employees-who-have-no-managers-ie-foreign-key-is-null)            |
| 10.   | [How can you find all customers who have placed more than one order?](#question-10-how-can-you-find-all-customers-who-have-placed-more-than-one-order)                          |
| 11.   | [How can you find the total number of orders for each customer?](#question-11-how-can-you-find-the-total-number-of-orders-for-each-customer)                                    |
| 12.   | [How can you calculate percentage or ratio values in a query?](#question-12-how-can-you-calculate-percentage-or-ratio-values-in-a-query)                                        |
| 13.   | [How can you find records with duplicate email addresses?](#question-13-how-can-you-find-records-with-duplicate-email-addresses)                                                |
| 14.   | [What is the use of `REGEXP` operator in MySQL?](#question-14-what-is-the-use-of-regexp-operator-in-mysql)                                                                      |
| 15.   | [What is the difference between `REGEXP` and `LIKE`?](#question-15-what-is-the-difference-between-regexp-and-like)                                                              |
| 16.   | [What are user-defined variables in MySQL and how are they used?](#question-16-what-are-user-defined-variables-in-mysql-and-how-are-they-used)                                  |
| 17.   | [What is the difference between `SET` and `ENUM`?](#question-17-what-is-the-difference-between-set-and-enum)                                                                    |
| 18.   | [How do you compare date ranges in a query?](#question-18-how-do-you-compare-date-ranges-in-a-query)                                                                            |
| 19.   | [How do you find data inserted within the last 7 days?](#question-19-how-do-you-find-data-inserted-within-the-last-7-days)                                                      |
| 20.   | [How can you check how many rows a query returned?](#question-20-how-can-you-check-how-many-rows-a-query-returned)                                                              |

## Question 1. What is the use of the `ANY` and `ALL` operators?

## What is the use of the `ANY` and `ALL` operators in MySQL?

### Definition

`ANY` and `ALL` are **comparison operators** in MySQL that are used **with a subquery** to compare a value against **multiple values returned by the subquery**.

- **`ANY`** → The condition is **TRUE if it matches at least one value** returned by the subquery.
- **`ALL`** → The condition is **TRUE only if it matches every value** returned by the subquery.

These operators are useful when comparing a single value with a set of values instead of writing multiple comparison conditions.

---

# Sample Database

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

INSERT INTO employees (name, department, salary, hire_date, manager_id) VALUES
('Alice', 'Engineering', 75000, '2020-01-15', NULL),
('Bob', 'Engineering', 80000, '2019-03-10', 1),
('Charlie', 'HR', 60000, '2021-07-22', NULL),
('David', 'Sales', 55000, '2022-02-11', NULL),
('Eva', 'Sales', 65000, '2020-05-20', 4);
```

---

# 1. Using `ANY`

### Example

Find employees whose salary is **greater than at least one salary in the Sales department**.

```sql
SELECT name, salary
FROM employees
WHERE salary > ANY (
    SELECT salary
    FROM employees
    WHERE department = 'Sales'
);
```

### Subquery Result

```text
55000
65000
```

The condition becomes:

```text
salary > ANY (55000, 65000)
```

Equivalent to:

```text
salary > 55000
OR
salary > 65000
```

Since only **one comparison needs to be true**, the result includes:

| Name    | Salary |
| ------- | ------ |
| Alice   | 75000  |
| Bob     | 80000  |
| Charlie | 60000  |
| Eva     | 65000  |

David (55000) is not included because `55000 > 55000` is false.

---

## Line-by-line Explanation

```sql
SELECT name, salary
```

Returns employee name and salary.

```sql
FROM employees
```

Reads data from the `employees` table.

```sql
WHERE salary > ANY (...)
```

Checks whether the salary is greater than **at least one** value returned by the subquery.

```sql
SELECT salary
FROM employees
WHERE department='Sales';
```

Returns:

```text
55000
65000
```

---

# 2. Using `ALL`

### Example

Find employees whose salary is **greater than every salary in the Sales department**.

```sql
SELECT name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department = 'Sales'
);
```

Subquery:

```text
55000
65000
```

Condition becomes:

```text
salary > 55000
AND
salary > 65000
```

Only salaries greater than **65000** satisfy this condition.

### Result

| Name  | Salary |
| ----- | ------ |
| Alice | 75000  |
| Bob   | 80000  |

---

# Another Example (`< ANY`)

```sql
SELECT name, salary
FROM employees
WHERE salary < ANY (
    SELECT salary
    FROM employees
    WHERE department = 'Engineering'
);
```

Engineering salaries:

```text
75000
80000
```

Condition:

```text
salary < 75000
OR
salary < 80000
```

Result:

| Name    |
| ------- |
| Charlie |
| David   |
| Eva     |
| Alice   |

---

# Another Example (`< ALL`)

Find employees earning less than **every Engineering employee**.

```sql
SELECT name, salary
FROM employees
WHERE salary < ALL (
    SELECT salary
    FROM employees
    WHERE department = 'Engineering'
);
```

Engineering salaries:

```text
75000
80000
```

Condition:

```text
salary < 75000
AND
salary < 80000
```

Result:

| Name    |
| ------- |
| Charlie |
| David   |
| Eva     |

---

# Difference Between `ANY` and `ALL`

| Feature            | `ANY`                           | `ALL`                       |
| ------------------ | ------------------------------- | --------------------------- |
| Meaning            | Matches at least one value      | Matches every value         |
| Logic              | OR                              | AND                         |
| Easier to satisfy? | Yes                             | No                          |
| Typical use        | Compare with one or more values | Compare with the entire set |

---

# How MySQL Evaluates Them

Suppose the subquery returns:

```text
10
20
30
```

### `> ANY`

```sql
value > ANY (10,20,30)
```

Equivalent to:

```text
value > 10
OR value > 20
OR value > 30
```

---

### `> ALL`

```sql
value > ALL (10,20,30)
```

Equivalent to:

```text
value > 10
AND value > 20
AND value > 30
```

---

# Performance Considerations

- `ANY` and `ALL` are typically used with **subqueries**.
- If the subquery is **uncorrelated**, MySQL can evaluate it once and reuse the result.
- Ensure columns used in the subquery's `WHERE` clause are indexed for better performance.
- For simple comparisons against minimum or maximum values, using aggregate functions can be more readable:
  - `x > ALL (subquery)` is often equivalent to `x > (SELECT MAX(...) FROM ...)`.
  - `x > ANY (subquery)` is often equivalent to `x > (SELECT MIN(...) FROM ...)`.

- Use `EXPLAIN` to inspect the execution plan and verify how MySQL executes the query.

Example:

```sql
EXPLAIN
SELECT name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department = 'Sales'
);
```

---

# Common Pitfalls

### 1. Empty Subquery Result

If the subquery returns **no rows**:

- `> ALL (empty set)` evaluates to **TRUE**.
- `> ANY (empty set)` evaluates to **FALSE**.

Example:

```sql
SELECT *
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department = 'Marketing'
);
```

If there are no employees in `Marketing`, the subquery returns an empty set, so the condition is true for all rows.

### 2. `NULL` Values

If the subquery returns `NULL` values, comparisons may evaluate to **UNKNOWN** due to SQL's three-valued logic. Filter out `NULL` values when appropriate:

```sql
SELECT name
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department = 'Sales'
      AND salary IS NOT NULL
);
```

---

# Interview Tip

A common interview question is:

> **What is the difference between `IN`, `ANY`, and `ALL`?**

- `IN` checks whether a value **equals one of the values** returned by the subquery.
- `ANY` compares a value using operators like `>`, `<`, `>=`, `<=`, `=`, or `<>` and succeeds if **at least one** comparison is true.
- `ALL` compares using the same operators but succeeds only if **every** comparison is true.

Example:

```sql
-- Equality check
WHERE department IN ('HR', 'Sales')

-- Comparison with at least one value
WHERE salary > ANY (
    SELECT salary
    FROM employees
    WHERE department = 'Sales'
)

-- Comparison with all values
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department = 'Sales'
)
```

## Question 2. What is the difference between `EXISTS` and `IN`?

## Question 3. How do you use `DISTINCT` with multiple columns?

## Question 4. How can you retrieve only even or odd IDs from a table?

## Question 5. How do you calculate total salary by department using `GROUP BY`?

## Question 6. How do you get the total count of all employees and the average salary together?

## Question 7. How do you select top N rows from a table?

## Question 8. How do you join a table with itself using aliases?

## Question 9. How can you list employees who have no managers (i.e., foreign key is `NULL`)?

## Question 10. How can you find all customers who have placed more than one order?

## Question 11. How can you find the total number of orders for each customer?

## Question 12. How can you calculate percentage or ratio values in a query?

## Question 13. How can you find records with duplicate email addresses?

## Question 14. What is the use of `REGEXP` operator in MySQL?

## Question 15. What is the difference between `REGEXP` and `LIKE`?

## Question 16. What are user-defined variables in MySQL and how are they used?

## Question 17. What is the difference between `SET` and `ENUM`?

## Question 18. How do you compare date ranges in a query?

## Question 19. How do you find data inserted within the last 7 days?

## Question 20. How can you check how many rows a query returned?
