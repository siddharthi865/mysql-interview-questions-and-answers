# Set 18

| S.No. | Question                                                                                                                                                                   |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What happens when you use a wildcard (`%`) at the start of a `LIKE` pattern?](#question-1-what-happens-when-you-use-a-wildcard--at-the-start-of-a-like-pattern)           |
| 2.    | [How can you make `LIKE` queries faster?](#question-2-how-can-you-make-like-queries-faster)                                                                                |
| 3.    | [What is a covering index and how can it speed up queries?](#question-3-what-is-a-covering-index-and-how-can-it-speed-up-queries)                                          |
| 4.    | [What is the difference between an index and a key in MySQL terminology?](#question-4-what-is-the-difference-between-an-index-and-a-key-in-mysql-terminology)              |
| 5.    | [How do you create an index on multiple columns?](#question-5-how-do-you-create-an-index-on-multiple-columns)                                                              |
| 6.    | [How can you find duplicate indexes in a table?](#question-6-how-can-you-find-duplicate-indexes-in-a-table)                                                                |
| 7.    | [What is an invisible index and how can you activate or deactivate it?](#question-7-what-is-an-invisible-index-and-how-can-you-activate-or-deactivate-it)                  |
| 8.    | [How can you check how many indexes exist on a table?](#question-8-how-can-you-check-how-many-indexes-exist-on-a-table)                                                    |
| 9.    | [How can you remove an unused index?](#question-9-how-can-you-remove-an-unused-index)                                                                                      |
| 10.   | [How can you identify slow subqueries in MySQL?](#question-10-how-can-you-identify-slow-subqueries-in-mysql)                                                               |
| 11.   | [What is a query cache and why was it deprecated in MySQL 8.0?](#question-11-what-is-a-query-cache-and-why-was-it-deprecated-in-mysql-80)                                  |
| 12.   | [What is the difference between caching at the SQL level vs application level?](#question-12-what-is-the-difference-between-caching-at-the-sql-level-vs-application-level) |
| 13.   | [What is query optimization and why is it important?](#question-13-what-is-query-optimization-and-why-is-it-important)                                                     |
| 14.   | [What are some ways to optimize `JOIN` operations?](#question-14-what-are-some-ways-to-optimize-join-operations)                                                           |
| 15.   | [What are the most common causes of slow queries?](#question-15-what-are-the-most-common-causes-of-slow-queries)                                                           |
| 16.   | [What is a temporary table and how is it different from a derived table?](#question-16-what-is-a-temporary-table-and-how-is-it-different-from-a-derived-table)             |
| 17.   | [What happens when two users create temporary tables with the same name?](#question-17-what-happens-when-two-users-create-temporary-tables-with-the-same-name)             |
| 18.   | [How can you ensure a temporary table is dropped automatically?](#question-18-how-can-you-ensure-a-temporary-table-is-dropped-automatically)                               |
| 19.   | [What is the difference between a temporary table and a CTE?](#question-19-what-is-the-difference-between-a-temporary-table-and-a-cte)                                     |
| 20.   | [Can you use a CTE inside a view?](#question-20-can-you-use-a-cte-inside-a-view)                                                                                           |

## Question 1. What happens when you use a wildcard (`%`) at the start of a `LIKE` pattern?

## Interview Answer

When you use a wildcard (`%`) **at the beginning** of a `LIKE` pattern (for example, `LIKE '%abc'` or `LIKE '%abc%'`), **MySQL cannot use a normal B-tree index efficiently to locate the matching rows**. Instead, it typically performs a **full table scan** (or a full index scan), making the query much slower on large tables.

### Why does this happen?

A B-tree index stores values in sorted order.

For example, suppose the `name` column contains:

```
Alice
Bob
Charlie
David
Emma
```

If you search like this:

```sql
WHERE name LIKE 'Al%'
```

MySQL knows the string **starts with "Al"**, so it can jump directly to the relevant section of the index.

```
Index

Alice   ← Start here
...
```

However, if you search like this:

```sql
WHERE name LIKE '%ice'
```

The beginning of the string is unknown.

Possible matches:

```
Alice
Nice
Office
Device
```

Since the pattern could match **any value ending with "ice"**, MySQL cannot determine where to start in the index. It has to examine every row.

---

## Example

Using the sample schema:

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
('Charlie', 'HR', 60000, '2021-07-22', NULL);

CREATE INDEX idx_name ON employees(name);
```

---

### Case 1: Wildcard at the end (Index Friendly)

```sql
SELECT *
FROM employees
WHERE name LIKE 'Al%';
```

### Explanation

- `LIKE 'Al%'`
- Matches:
  - Alice
  - Albert
  - Alfred

- MySQL can use `idx_name`.
- This is called a **prefix search**.

Execution plan often shows:

```text
type: range
key: idx_name
```

---

### Case 2: Wildcard at the beginning (Not Index Friendly)

```sql
SELECT *
FROM employees
WHERE name LIKE '%ice';
```

### Explanation

- Finds names ending in `ice`
- MySQL usually cannot use the index for a direct lookup.
- Every row must be checked.

Execution plan often shows:

```text
type: ALL
key: NULL
```

or, depending on the optimizer and query, a full index scan rather than an efficient range scan.

---

### Case 3: Wildcards on both sides

```sql
SELECT *
FROM employees
WHERE name LIKE '%lic%';
```

Matches:

```
Alice
Alicia
Malice
```

Again, MySQL generally scans the table because the starting characters are unknown.

---

## Verify Using `EXPLAIN`

```sql
EXPLAIN
SELECT *
FROM employees
WHERE name LIKE '%ice';
```

Possible output:

| id  | select_type | table     | type | key  |
| --- | ----------- | --------- | ---- | ---- |
| 1   | SIMPLE      | employees | ALL  | NULL |

### Meaning

- `ALL` → Full table scan
- `NULL` → No useful index selected

Compare with:

```sql
EXPLAIN
SELECT *
FROM employees
WHERE name LIKE 'Al%';
```

Possible output:

| id  | table     | type  | key      |
| --- | --------- | ----- | -------- |
| 1   | employees | range | idx_name |

Here:

- `range` indicates an efficient index range scan.
- `idx_name` is used.

---

## Performance Impact

On a table with:

- 1,000 rows → Difference is usually small.
- 100,000 rows → Noticeable slowdown.
- Millions of rows → Can become very expensive.

A leading `%` forces MySQL to inspect many or all rows, increasing CPU usage and I/O.

---

## Optimizations

### 1. Prefer Prefix Searches

Instead of:

```sql
WHERE name LIKE '%Ali%'
```

Use (when the requirement allows):

```sql
WHERE name LIKE 'Ali%'
```

This lets MySQL use the index.

---

### 2. Use Full-Text Search for Text Searching

For searching words or phrases anywhere within large text columns, use MySQL's `FULLTEXT` indexes and `MATCH ... AGAINST` instead of `LIKE '%...%'` where appropriate.

Example:

```sql
ALTER TABLE employees
ADD FULLTEXT(name);

SELECT *
FROM employees
WHERE MATCH(name) AGAINST('Alice');
```

This is generally much more efficient for text search than a leading-wildcard `LIKE`.

---

### 3. Consider Schema or Search Design

Depending on the use case, you might:

- Store searchable prefixes or normalized values.
- Use generated columns and indexes.
- Use a dedicated search engine (such as Elasticsearch or OpenSearch) for complex substring or fuzzy searches.

---

## Common Interview Pitfalls

**❌ Misconception:** "`LIKE` always uses indexes."

**✔ Reality:** Only patterns with a known leading prefix (for example, `'abc%'`) are generally index-friendly with standard B-tree indexes.

---

**❌ Misconception:** "`'%abc'` is just as fast as `'abc%'`."

**✔ Reality:** A leading `%` usually prevents an efficient index range scan, often resulting in a full table scan or full index scan.

---

## Interview Summary

- `LIKE 'abc%'` → **Index-friendly** (prefix search).
- `LIKE '%abc'` → **Usually cannot use a B-tree index efficiently**.
- `LIKE '%abc%'` → **Also generally not index-friendly**.
- Use `EXPLAIN` to verify whether MySQL uses an index.
- For large-scale text searches, consider `FULLTEXT` indexes or a dedicated search solution instead of leading-wildcard `LIKE`.

> This behavior is consistent with the latest MySQL optimizer documentation for B-tree indexes and `LIKE` pattern matching, where only left-anchored patterns can typically benefit from index range scans.

## Question 2. How can you make `LIKE` queries faster?

## Question 3. What is a covering index and how can it speed up queries?

## Question 4. What is the difference between an index and a key in MySQL terminology?

## Question 5. How do you create an index on multiple columns?

## Question 6. How can you find duplicate indexes in a table?

## Question 7. What is an invisible index and how can you activate or deactivate it?

## Question 8. How can you check how many indexes exist on a table?

## Question 9. How can you remove an unused index?

## Question 10. How can you identify slow subqueries in MySQL?

## Question 11. What is a query cache and why was it deprecated in MySQL 8.0?

## Question 12. What is the difference between caching at the SQL level vs application level?

## Question 13. What is query optimization and why is it important?

## Question 14. What are some ways to optimize `JOIN` operations?

## Question 15. What are the most common causes of slow queries?

## Question 16. What is a temporary table and how is it different from a derived table?

## Question 17. What happens when two users create temporary tables with the same name?

## Question 18. How can you ensure a temporary table is dropped automatically?

## Question 19. What is the difference between a temporary table and a CTE?

## Question 20. Can you use a CTE inside a view?
