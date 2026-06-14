# Set 11

| S.No. | Question                                                                                                                                          |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What command is used to log into MySQL from the terminal?](#question-1-what-command-is-used-to-log-into-mysql-from-the-terminal)                 |
| 2.    | [How do you create a new MySQL user?](#question-2-how-do-you-create-a-new-mysql-user)                                                             |
| 3.    | [How do you grant all privileges to a user on a specific database?](#question-3-how-do-you-grant-all-privileges-to-a-user-on-a-specific-database) |
| 4.    | [How do you revoke privileges from a user?](#question-4-how-do-you-revoke-privileges-from-a-user)                                                 |
| 5.    | [What does the `FLUSH PRIVILEGES` command do?](#question-5-what-does-the-flush-privileges-command-do)                                             |
| 6.    | [How do you change the MySQL root password?](#question-6-how-do-you-change-the-mysql-root-password)                                               |
| 7.    | [How can you list all MySQL users?](#question-7-how-can-you-list-all-mysql-users)                                                                 |
| 8.    | [What is the purpose of the `mysql.user` table?](#question-8-what-is-the-purpose-of-the-mysqluser-table)                                          |
| 9.    | [How do you delete a MySQL user?](#question-9-how-do-you-delete-a-mysql-user)                                                                     |
| 10.   | [What are the advantages of using MySQL over other databases?](#question-10-what-are-the-advantages-of-using-mysql-over-other-databases)          |
| 11.   | [How do you check if MySQL service is running?](#question-11-how-do-you-check-if-mysql-service-is-running)                                        |
| 12.   | [How do you stop and start MySQL service manually?](#question-12-how-do-you-stop-and-start-mysql-service-manually)                                |
| 13.   | [What's the difference between MySQL client and MySQL server?](#question-13-whats-the-difference-between-mysql-client-and-mysql-server)           |
| 14.   | [How do you check the size of a database?](#question-14-how-do-you-check-the-size-of-a-database)                                                  |
| 15.   | [How can you see the storage engine used by each table?](#question-15-how-can-you-see-the-storage-engine-used-by-each-table)                      |
| 16.   | [What is a schema in MySQL?](#question-16-what-is-a-schema-in-mysql)                                                                              |
| 17.   | [How do you duplicate a database in MySQL?](#question-17-how-do-you-duplicate-a-database-in-mysql)                                                |
| 18.   | [How do you drop a database safely?](#question-18-how-do-you-drop-a-database-safely)                                                              |
| 19.   | [How do you rename a database in MySQL?](#question-19-how-do-you-rename-a-database-in-mysql)                                                      |
| 20.   | [What happens if you try to drop a database that doesn't exist?](#question-20-what-happens-if-you-try-to-drop-a-database-that-doesnt-exist)       |

## Question 1. What command is used to log into MySQL from the terminal?

## **What command is used to log into MySQL from the terminal?**

### **Answer (Interview-Friendly)**

To log into a MySQL server from the terminal (Command Prompt, PowerShell, or Linux/macOS Terminal), you use the **`mysql` client program**.

### Basic Syntax

```bash
mysql -u username -p
```

**Explanation:**

- `mysql` → Starts the MySQL command-line client.
- `-u` → Specifies the MySQL username.
- `username` → The MySQL user account (e.g., `root`).
- `-p` → Prompts for the user's password securely.

**Example:**

```bash
mysql -u root -p
```

After pressing **Enter**, you'll see:

```text
Enter password:
```

Type your password (it won't be displayed on the screen), then press **Enter**.

If authentication succeeds:

```text
Welcome to the MySQL monitor.
mysql>
```

You can now execute SQL statements.

---

## Common Login Examples

### 1. Login as the root user

```bash
mysql -u root -p
```

---

### 2. Login to a remote MySQL server

```bash
mysql -h 192.168.1.100 -u root -p
```

**Explanation:**

- `-h` → Hostname or IP address of the MySQL server
- `-u` → Username
- `-p` → Password prompt

---

### 3. Specify a port

```bash
mysql -h localhost -P 3307 -u root -p
```

- `-P` (uppercase) specifies the port number.

---

### 4. Open a specific database immediately after login

```bash
mysql -u root -p interview_db
```

or

```bash
mysql -u root -p -D interview_db
```

After login, `interview_db` becomes the default database.

---

## Verify the Connection

Once logged in:

```sql
SHOW DATABASES;
```

Example output:

```text
+--------------------+
| Database           |
+--------------------+
| information_schema |
| interview_db       |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

You can then select a database:

```sql
USE interview_db;
```

---

## Using the Sample Database

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
('Alice', 'Engineering', 75000, '2020-01-15', NULL),
('Bob', 'Engineering', 80000, '2019-03-10', 1),
('Charlie', 'HR', 60000, '2021-07-22', NULL);

SELECT * FROM employees;
```

### Line-by-line Explanation

- `CREATE DATABASE interview_db;` → Creates a new database.
- `USE interview_db;` → Makes it the active database.
- `CREATE TABLE employees (...)` → Creates the `employees` table.
- `INSERT INTO ... VALUES ...` → Inserts three sample employee records.
- `SELECT * FROM employees;` → Retrieves all rows from the table.

---

## Other Useful MySQL Terminal Commands

```bash
mysql --version
```

Displays the installed MySQL client version.

```bash
mysql -u root
```

Logs in **without** prompting for a password (only works if the account has no password or uses socket/authentication mechanisms).

```bash
mysql -u root -p -e "SHOW DATABASES;"
```

Executes a SQL statement directly from the terminal without entering the interactive MySQL shell.

---

## Security Best Practice

Avoid specifying the password directly on the command line, such as:

```bash
mysql -u root -pmypassword
```

Although supported, this is **not recommended** because the password may be visible in the shell history or process list. Using `-p` without the password prompts securely for it.

---

## Common Pitfalls

- **`mysql: command not found`** → The MySQL client is not installed or its `bin` directory is not in your system's `PATH`.
- **`Access denied for user`** → The username or password is incorrect, or the user lacks permission.
- **Can't connect to MySQL server** → Verify that the MySQL server is running and that the host and port are correct.

---

## Interview Tips

- The standard command to log in is:

  ```bash
  mysql -u root -p
  ```

- Remember common options:
  - `-u` → Username
  - `-p` → Password prompt
  - `-h` → Host
  - `-P` → Port
  - `-D` → Default database

These options are part of the MySQL command-line client and are documented in the latest MySQL Reference Manual.

## Question 2. How do you create a new MySQL user?

## Question 3. How do you grant all privileges to a user on a specific database?

## Question 4. How do you revoke privileges from a user?

## Question 5. What does the `FLUSH PRIVILEGES` command do?

## Question 6. How do you change the MySQL root password?

## Question 7. How can you list all MySQL users?

## Question 8. What is the purpose of the `mysql.user` table?

## Question 9. How do you delete a MySQL user?

## Question 10. What are the advantages of using MySQL over other databases?

## Question 11. How do you check if MySQL service is running?

## Question 12. How do you stop and start MySQL service manually?

## Question 13. What’s the difference between MySQL client and MySQL server?

## Question 14. How do you check the size of a database?

## Question 15. How can you see the storage engine used by each table?

## Question 16. What is a schema in MySQL?

## Question 17. How do you duplicate a database in MySQL?

## Question 18. How do you drop a database safely?

## Question 19. How do you rename a database in MySQL?

## Question 20. What happens if you try to drop a database that doesn’t exist?
