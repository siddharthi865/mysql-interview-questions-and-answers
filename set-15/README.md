# Set 15

| S.No. | Question                                                                                                                                                                  |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the purpose of relay logs in replication?](#question-1-what-is-the-purpose-of-relay-logs-in-replication)                                                         |
| 2.    | [What is semi-synchronous replication in MySQL?](#question-2-what-is-semi-synchronous-replication-in-mysql)                                                               |
| 3.    | [What are replication filters?](#question-3-what-are-replication-filters)                                                                                                 |
| 4.    | [How can you skip a replication error?](#question-4-how-can-you-skip-a-replication-error)                                                                                 |
| 5.    | [What is a replication lag and how can it be minimized?](#question-5-what-is-a-replication-lag-and-how-can-it-be-minimized)                                               |
| 6.    | [What is read-write splitting in MySQL replication?](#question-6-what-is-read-write-splitting-in-mysql-replication)                                                       |
| 7.    | [What are failover and switchover in replication?](#question-7-what-are-failover-and-switchover-in-replication)                                                           |
| 8.    | [What is multi-source replication?](#question-8-what-is-multi-source-replication)                                                                                         |
| 9.    | [How do you implement circular replication?](#question-9-how-do-you-implement-circular-replication)                                                                       |
| 10.   | [What is MySQL fabric or MySQL router?](#question-10-what-is-mysql-fabric-or-mysql-router)                                                                                |
| 11.   | [What is sharding in MySQL?](#question-11-what-is-sharding-in-mysql)                                                                                                      |
| 12.   | [How does horizontal partitioning differ from sharding?](#question-12-how-does-horizontal-partitioning-differ-from-sharding)                                              |
| 13.   | [What are common sharding strategies?](#question-13-what-are-common-sharding-strategies)                                                                                  |
| 14.   | [What are the limitations of MySQL partitioning?](#question-14-what-are-the-limitations-of-mysql-partitioning)                                                            |
| 15.   | [What are functional indexes in MySQL 8?](#question-15-what-are-functional-indexes-in-mysql-8)                                                                            |
| 16.   | [How do you create an index on an expression or function?](#question-16-how-do-you-create-an-index-on-an-expression-or-function)                                          |
| 17.   | [What are invisible indexes and when would you use them?](#question-17-what-are-invisible-indexes-and-when-would-you-use-them)                                            |
| 18.   | [How does MySQL handle JSON indexing?](#question-18-how-does-mysql-handle-json-indexing)                                                                                  |
| 19.   | [What is the difference between generated columns and virtual columns?](#question-19-what-is-the-difference-between-generated-columns-and-virtual-columns)                |
| 20.   | [What are common performance tuning parameters in `my.cnf` for large databases?](#question-20-what-are-common-performance-tuning-parameters-in-mycnf-for-large-databases) |

## Question 1. What is the purpose of relay logs in replication?

## What is the purpose of relay logs in MySQL replication?

### Interview Answer

**Relay logs** are **temporary log files maintained on a MySQL replica (slave)** that store the binary log events received from the source (master) server. Their primary purpose is to **decouple the process of receiving changes from applying them**, making replication more reliable and efficient.

In simple terms:

- **Source server** writes changes to the **binary log (binlog)**.
- **Replica's I/O thread** reads those binary log events from the source and writes them into the **relay log**.
- **Replica's SQL thread (or worker threads in parallel replication)** reads the relay log and executes the events locally.

This separation allows the replica to continue receiving new changes even if applying previous changes is temporarily slow.

---

# Replication Flow

```text
                Source (Master)
             +-------------------+
             | Application Writes|
             +---------+---------+
                       |
                Binary Log (binlog)
                       |
                Replication I/O Thread
                       |
                       v
                Replica (Slave)
             +-------------------+
             |   Relay Log       |
             +---------+---------+
                       |
          SQL Thread / Worker Threads
                       |
                       v
             Local Database Updated
```

---

# Step-by-Step Working

### Step 1: Transaction occurs on Source

```sql
INSERT INTO employees(name, department, salary)
VALUES ('David', 'Sales', 55000);
```

The source records this transaction in its **binary log**.

---

### Step 2: Replica I/O Thread

The replica's **I/O thread** connects to the source and copies new binary log events.

It writes them into:

```text
relay-log.000001
relay-log.000002
...
```

No SQL execution happens yet.

---

### Step 3: SQL Thread

The SQL thread reads:

```text
relay-log.000001
```

and executes:

```sql
INSERT INTO employees(name, department, salary)
VALUES ('David', 'Sales', 55000);
```

Now the replica data matches the source.

---

# Why Relay Logs Are Needed

Without relay logs:

```text
Master Binlog
      ↓
Replica SQL Thread
```

The SQL thread would need to continuously read from the source while executing queries.

With relay logs:

```text
Master Binlog
      ↓
Replica IO Thread
      ↓
 Relay Log
      ↓
Replica SQL Thread
```

Receiving and executing become independent processes.

Benefits include:

- Faster replication
- Better fault tolerance
- Reduced dependency on continuous network connectivity
- Supports parallel replication

---

# Example Replication Threads

You can view replication status using:

```sql
SHOW REPLICA STATUS\G
```

Example output:

```text
Replica_IO_Running: Yes
Replica_SQL_Running: Yes

Relay_Log_File: relay-bin.000015
Relay_Log_Pos: 2583

Source_Log_File: mysql-bin.000108
Read_Source_Log_Pos: 84352
```

### Explanation

- `Replica_IO_Running` → I/O thread is fetching binary logs.
- `Replica_SQL_Running` → SQL thread is executing relay logs.
- `Relay_Log_File` → current relay log file.
- `Relay_Log_Pos` → current execution position.
- `Source_Log_File` → source binary log currently being read.
- `Read_Source_Log_Pos` → last position fetched from the source.

---

# Relay Log Files

Typical files on the replica:

```text
relay-bin.000001
relay-bin.000002
relay-bin.index
```

- `relay-bin.*` → contains copied binary log events.
- `relay-bin.index` → list of relay log files.

---

# Parallel Replication

In modern MySQL, multiple worker threads can execute relay log events concurrently.

```text
Relay Log
     |
-------------------------
|       |       |       |
Worker1 Worker2 Worker3
```

This significantly improves replication throughput on multi-core systems.

---

# Relay Log Recovery

If the replica crashes:

- Relay logs help resume replication from the last consistent position.
- MySQL can rebuild or recover relay logs depending on the configuration (for example, using relay log recovery during startup).

---

# Difference Between Binary Log and Relay Log

| Feature    | Binary Log                                           | Relay Log                                                                  |
| ---------- | ---------------------------------------------------- | -------------------------------------------------------------------------- |
| Created on | Source server                                        | Replica server                                                             |
| Purpose    | Record all data changes                              | Store copied binary log events before execution                            |
| Read by    | Replicas                                             | Replica SQL/worker threads                                                 |
| Used for   | Replication, backup, point-in-time recovery          | Applying replicated changes locally                                        |
| Permanent  | Usually retained based on binlog expiration settings | Typically temporary and purged after use (subject to replication settings) |

---

# Advantages of Relay Logs

- Separate data transfer from SQL execution.
- Allow the replica to continue downloading changes even if execution is delayed.
- Improve replication reliability.
- Enable parallel replication.
- Help with crash recovery.
- Reduce dependency on network latency during SQL execution.

---

# Common Interview Pitfalls

### 1. Relay logs are **not** the same as binary logs

- **Binary log** → generated by the source.
- **Relay log** → generated by the replica from the source's binary log.

---

### 2. Relay logs exist only on replicas

The source server does **not** use relay logs.

---

### 3. Relay logs are internal replication files

Applications should never read or modify relay log files directly.

---

# Performance Considerations

- Keep relay logs on fast storage (SSD/NVMe) for better replication performance.
- Monitor replication lag using:

  ```sql
  SHOW REPLICA STATUS\G
  ```

- Use parallel replication where appropriate to improve apply throughput.
- Ensure relay logs are purged appropriately after successful execution to avoid unnecessary disk usage.

---

# Interview Summary

> **Relay logs are temporary log files stored on a MySQL replica. The replica's I/O thread copies events from the source's binary log into the relay log, and the SQL thread (or parallel worker threads) reads and executes those events. This separation improves replication performance, reliability, supports parallel execution, and helps with recovery after interruptions.**

## Question 2. What is semi-synchronous replication in MySQL?

## Question 3. What are replication filters?

## Question 4. How can you skip a replication error?

## Question 5. What is a replication lag and how can it be minimized?

## Question 6. What is read-write splitting in MySQL replication?

## Question 7. What are failover and switchover in replication?

## Question 8. What is multi-source replication?

## Question 9. How do you implement circular replication?

## Question 10. What is MySQL fabric or MySQL router?

## Question 11. What is sharding in MySQL?

## Question 12. How does horizontal partitioning differ from sharding?

## Question 13. What are common sharding strategies?

## Question 14. What are the limitations of MySQL partitioning?

## Question 15. What are functional indexes in MySQL 8?

## Question 16. How do you create an index on an expression or function?

## Question 17. What are invisible indexes and when would you use them?

## Question 18. How does MySQL handle JSON indexing?

## Question 19. What is the difference between generated columns and virtual columns?

## Question 20. What are common performance tuning parameters in `my.cnf` for large databases?
