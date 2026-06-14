# Set 24

| S.No. | Question                                                                                                                                                               |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What are the main configuration files used by MySQL?](#question-1-what-are-the-main-configuration-files-used-by-mysql)                                                |
| 2.    | [What is the purpose of the `my.cnf` file?](#question-2-what-is-the-purpose-of-the-mycnf-file)                                                                         |
| 3.    | [How can you change MySQL configuration dynamically?](#question-3-how-can-you-change-mysql-configuration-dynamically)                                                  |
| 4.    | [What is the `performance_schema` used for?](#question-4-what-is-the-performance_schema-used-for)                                                                      |
| 5.    | [How do you enable and use the slow query log?](#question-5-how-do-you-enable-and-use-the-slow-query-log)                                                              |
| 6.    | [How do you monitor memory usage in MySQL?](#question-6-how-do-you-monitor-memory-usage-in-mysql)                                                                      |
| 7.    | [What is the `innodb_buffer_pool_size` parameter, and how do you tune it?](#question-7-what-is-the-innodb_buffer_pool_size-parameter-and-how-do-you-tune-it)           |
| 8.    | [What is the `max_connections` setting used for?](#question-8-what-is-the-max_connections-setting-used-for)                                                            |
| 9.    | [How can you view MySQL server status and uptime?](#question-9-how-can-you-view-mysql-server-status-and-uptime)                                                        |
| 10.   | [How do you find which queries consume the most resources?](#question-10-how-do-you-find-which-queries-consume-the-most-resources)                                     |
| 11.   | [What is the purpose of the `information_schema` database?](#question-11-what-is-the-purpose-of-the-information_schema-database)                                       |
| 12.   | [What is the difference between `information_schema` and `performance_schema`?](#question-12-what-is-the-difference-between-information_schema-and-performance_schema) |
| 13.   | [How can you check which users are currently connected?](#question-13-how-can-you-check-which-users-are-currently-connected)                                           |
| 14.   | [How do you identify long-running queries?](#question-14-how-do-you-identify-long-running-queries)                                                                     |
| 15.   | [How can you terminate a running query?](#question-15-how-can-you-terminate-a-running-query)                                                                           |
| 16.   | [How do you check the MySQL error log?](#question-16-how-do-you-check-the-mysql-error-log)                                                                             |
| 17.   | [What is the purpose of the `tmp_table_size` setting?](#question-17-what-is-the-purpose-of-the-tmp_table_size-setting)                                                 |
| 18.   | [How do you estimate memory usage per connection?](#question-18-how-do-you-estimate-memory-usage-per-connection)                                                       |
| 19.   | [What is the `innodb_flush_log_at_trx_commit` setting?](#question-19-what-is-the-innodb_flush_log_at_trx_commit-setting)                                               |
| 20.   | [How do you tune MySQL for high write workloads?](#question-20-how-do-you-tune-mysql-for-high-write-workloads)                                                         |

## Question 1. What are the main configuration files used by MySQL?

## What are the main configuration files used by MySQL?

### Interview-Friendly Answer

A **MySQL configuration file** (also called an **option file**) is a text file that stores server and client configuration settings. Instead of passing options every time you start MySQL, you define them once in these files.

These files control important settings such as:

- Server port
- Data directory
- Buffer pool size
- Maximum connections
- Binary logging
- Character set and collation
- Replication settings
- InnoDB configuration
- Security-related options

When MySQL starts, it reads configuration files in a predefined order. Settings in later files typically override earlier ones if the same option is specified.

---

# Main MySQL Configuration Files

The exact location depends on the operating system.

| Operating System           | Configuration File                             |
| -------------------------- | ---------------------------------------------- |
| Linux (global)             | `/etc/my.cnf`                                  |
| Linux                      | `/etc/mysql/my.cnf`                            |
| Linux (additional configs) | `/etc/mysql/conf.d/*.cnf`                      |
| Linux (MySQL packages)     | `/etc/mysql/mysql.conf.d/mysqld.cnf`           |
| User-specific              | `~/.my.cnf`                                    |
| Windows                    | `C:\ProgramData\MySQL\MySQL Server 8.x\my.ini` |
| Windows (alternate)        | `C:\Windows\my.ini`                            |

---

# Typical Structure of a Configuration File

Example:

```ini
[mysqld]
port=3306
datadir=/var/lib/mysql

max_connections=300

innodb_buffer_pool_size=2G

log_bin=mysql-bin

server_id=1

character-set-server=utf8mb4

default_authentication_plugin=caching_sha2_password
```

---

# Explanation Line-by-Line

```ini
[mysqld]
```

Configuration for the MySQL Server.

---

```ini
port=3306
```

Server listens on port **3306**.

---

```ini
datadir=/var/lib/mysql
```

Location where database files are stored.

---

```ini
max_connections=300
```

Maximum simultaneous client connections.

---

```ini
innodb_buffer_pool_size=2G
```

Memory allocated for InnoDB caching.

This is one of the most important performance tuning parameters.

---

```ini
log_bin=mysql-bin
```

Enables binary logging.

Required for:

- Replication
- Point-in-time recovery
- Incremental backups

---

```ini
server_id=1
```

Unique server ID.

Required when replication is enabled.

---

```ini
character-set-server=utf8mb4
```

Sets the default character set.

`utf8mb4` is recommended because it fully supports Unicode, including emojis.

---

```ini
default_authentication_plugin=caching_sha2_password
```

Sets the default authentication plugin. In current MySQL releases, authentication defaults have evolved, so this option is often unnecessary unless you need to explicitly control compatibility with older clients. Always verify against the MySQL version you are using.

---

# Common Configuration Sections

Configuration files contain different sections.

```ini
[mysqld]
```

Server settings.

---

```ini
[client]
```

Applies to all MySQL client programs.

Example:

```ini
[client]
port=3306
socket=/var/run/mysqld/mysqld.sock
```

---

```ini
[mysql]
```

Only affects the MySQL command-line client.

Example:

```ini
[mysql]
prompt="mysql> "
```

---

```ini
[mysqldump]
```

Configuration for `mysqldump`.

Example:

```ini
[mysqldump]
quick
max_allowed_packet=256M
```

---

# Example Configuration for a Production Server

```ini
[mysqld]

port=3306

max_connections=500

innodb_buffer_pool_size=8G

innodb_log_file_size=1G

log_bin=mysql-bin

server_id=1

slow_query_log=ON

slow_query_log_file=/var/log/mysql/slow.log

long_query_time=2

character-set-server=utf8mb4

collation-server=utf8mb4_0900_ai_ci
```

---

# Viewing Current Configuration

You can view configuration values using SQL:

```sql
SHOW VARIABLES;
```

Specific variable:

```sql
SHOW VARIABLES LIKE 'max_connections';
```

Example output:

| Variable_name   | Value |
| --------------- | ----- |
| max_connections | 300   |

---

# See Which Option Files MySQL Reads

From the terminal:

```bash
mysqld --help --verbose
```

or

```bash
mysql --help
```

These commands list the option files MySQL searches and reads during startup.

---

# Dynamic vs Static Configuration

Some settings can be changed without restarting the server.

Example:

```sql
SET GLOBAL max_connections = 500;
```

Others require editing the configuration file and restarting MySQL.

Example:

```ini
innodb_log_file_size=1G
```

---

# Commonly Tuned Parameters

| Parameter                 | Purpose                       |
| ------------------------- | ----------------------------- |
| `max_connections`         | Maximum concurrent users      |
| `innodb_buffer_pool_size` | InnoDB cache memory           |
| `innodb_log_file_size`    | Redo log size                 |
| `tmp_table_size`          | Temporary table size          |
| `sort_buffer_size`        | Memory for sorting            |
| `join_buffer_size`        | Join operations               |
| `max_allowed_packet`      | Maximum packet size           |
| `slow_query_log`          | Enable slow query logging     |
| `long_query_time`         | Slow query threshold          |
| `log_bin`                 | Enable binary logs            |
| `server_id`               | Replication server identifier |

---

# Using `EXPLAIN` for Performance

While configuration files tune the server, query performance should be analyzed separately using `EXPLAIN`.

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

INSERT INTO employees (name, department, salary, hire_date, manager_id) VALUES
('Alice', 'Engineering', 75000, '2020-01-15', NULL),
('Bob', 'Engineering', 80000, '2019-03-10', 1),
('Charlie', 'HR', 60000, '2021-07-22', NULL);

EXPLAIN
SELECT *
FROM employees
WHERE department = 'Engineering';
```

### Explanation

1. `CREATE DATABASE` and `USE` create and select the sample database.
2. `CREATE TABLE` defines the `employees` table with columns for employee details.
3. `INSERT` adds sample records.
4. `EXPLAIN` shows the execution plan for the `SELECT` query, helping you determine whether indexes are being used and estimate the cost of the query. If `department` is queried frequently, adding an index on that column can improve performance.

---

# MySQL-Specific Notes and Best Practices

- Store custom configuration changes in included `.cnf` files (such as those under `conf.d` or `mysql.conf.d`) when supported by your installation, instead of modifying package-managed files directly.
- Use `SHOW VARIABLES` to verify the active configuration after changes.
- Validate configuration changes before restarting MySQL using:

  ```bash
  mysqld --validate-config
  ```

  (Available in recent MySQL versions.)

- Restart the MySQL service after modifying static configuration parameters.
- Use `SET PERSIST` or `SET PERSIST_ONLY` (supported in MySQL 8.0+) for many system variables to make changes survive restarts without manually editing configuration files, when appropriate.

---

# Common Interview Pitfalls

- Confusing **server configuration** (`[mysqld]`) with **client configuration** (`[client]`).
- Assuming every configuration change requires a restart; many variables are dynamic.
- Setting `innodb_buffer_pool_size` too high, causing the operating system to swap memory.
- Forgetting to configure a unique `server_id` when enabling replication.
- Editing the wrong configuration file when multiple option files are loaded.

## Question 2. What is the purpose of the `my.cnf` file?

## Question 3. How can you change MySQL configuration dynamically?

## Question 4. What is the `performance_schema` used for?

## Question 5. How do you enable and use the slow query log?

## Question 6. How do you monitor memory usage in MySQL?

## Question 7. What is the `innodb_buffer_pool_size` parameter, and how do you tune it?

## Question 8. What is the `max_connections` setting used for?

## Question 9. How can you view MySQL server status and uptime?

## Question 10. How do you find which queries consume the most resources?

## Question 11. What is the purpose of the `information_schema` database?

## Question 12. What is the difference between `information_schema` and `performance_schema`?

## Question 13. How can you check which users are currently connected?

## Question 14. How do you identify long-running queries?

## Question 15. How can you terminate a running query?

## Question 16. How do you check the MySQL error log?

## Question 17. What is the purpose of the `tmp_table_size` setting?

## Question 18. How do you estimate memory usage per connection?

## Question 19. What is the `innodb_flush_log_at_trx_commit` setting?

## Question 20. How do you tune MySQL for high write workloads?
