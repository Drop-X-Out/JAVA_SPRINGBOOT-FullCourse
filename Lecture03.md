# Lecture 03: Non-SELECT Operations using `Statement`

---

# What are Non-SELECT Operations?

In SQL, there are mainly two types of operations.

## 1. SELECT

Used to retrieve data.

Example

```sql
SELECT * FROM employee;
```

Returns rows.

---

## 2. Non-SELECT Operations

Used to modify the database.

Examples

```sql
INSERT
UPDATE
DELETE
CREATE
ALTER
DROP
TRUNCATE
```

These operations **do not return rows**.

Instead, they return:

* Number of affected rows
* Or simply indicate that the SQL statement executed successfully (for DDL statements).

---

# Why Can't We Use `executeQuery()`?

Suppose we write

```java
statement.executeQuery(
"INSERT INTO employee(name,age,department) VALUES('Usama',24,'IT')"
);
```

Will it work?

❌ No.

Reason:

`executeQuery()` expects a **ResultSet**.

But INSERT doesn't return any rows.

So Java throws an exception.

---

# Which Method Should We Use?

For every Non-SELECT operation

Use

```java
executeUpdate()
```

---

# Why the Name `executeUpdate()`?

Many beginners think it is only for UPDATE.

Actually it is used for

* INSERT
* UPDATE
* DELETE
* CREATE
* DROP
* ALTER
* TRUNCATE

Almost every SQL statement except SELECT.

---

# Return Value of `executeUpdate()`

Syntax

```java
int rows = statement.executeUpdate(query);
```

The returned integer means

> **How many rows were affected?**

Example

```java
int rows = statement.executeUpdate(query);
```

If

```sql
INSERT
```

adds one row

then

```java
rows = 1
```

---

If

```sql
UPDATE
```

changes three rows

then

```java
rows = 3
```

---

If

```sql
DELETE
```

removes five rows

then

```java
rows = 5
```

---

# Database Table

Assume

```text
employee

-----------------------------------------
id | name   | age | department
-----------------------------------------
1  | Usama  | 24  | IT
2  | Ali    | 22  | HR
3  | Sara   | 26  | Finance
-----------------------------------------
```

---

# Example 1: INSERT using Statement

```java
import java.sql.*;

public class Main {

    private static final String url =
            "jdbc:mysql://localhost:3306/mydb";

    private static final String username = "root";
    private static final String password = "password";

    public static void main(String[] args) {

        try {

            Class.forName("com.mysql.cj.jdbc.Driver");

            Connection connection =
                    DriverManager.getConnection(
                            url,
                            username,
                            password
                    );

            Statement statement =
                    connection.createStatement();

            String query =
                    "INSERT INTO employee(name,age,department) " +
                    "VALUES('Aman',21,'Marketing')";

            int rows =
                    statement.executeUpdate(query);

            System.out.println(
                    rows + " row inserted successfully."
            );

            statement.close();
            connection.close();

        } catch (Exception e) {

            e.printStackTrace();

        }
    }
}
```

---

## Explanation

### SQL Query

```java
String query =
"INSERT INTO employee(name,age,department)
VALUES('Aman',21,'Marketing')";
```

Java stores the SQL command inside a String.

Nothing is executed yet.

---

### Execute

```java
statement.executeUpdate(query);
```

Statement sends SQL to MySQL.

Database inserts a row.

Returns

```text
1
```

because one row was inserted.

---

Output

```text
1 row inserted successfully.
```

---

Database becomes

```text
-----------------------------------------
id | name | age | department
-----------------------------------------
1  | Usama
2  | Ali
3  | Sara
4  | Aman
-----------------------------------------
```

---

# Example 2: UPDATE using Statement

```java
import java.sql.*;

public class Main {

    private static final String url =
            "jdbc:mysql://localhost:3306/mydb";

    private static final String username = "root";
    private static final String password = "password";

    public static void main(String[] args) {

        try {

            Connection connection =
                    DriverManager.getConnection(
                            url,
                            username,
                            password
                    );

            Statement statement =
                    connection.createStatement();

            String query =
                    "UPDATE employee SET department='Development' WHERE id=2";

            int rows =
                    statement.executeUpdate(query);

            System.out.println(
                    rows + " row updated successfully."
            );

            statement.close();
            connection.close();

        } catch (Exception e) {

            e.printStackTrace();

        }
    }
}
```

---

Database Before

```text
2 Ali HR
```

Database After

```text
2 Ali Development
```

Returned value

```text
1
```

One row updated.

---

# Example 3: DELETE using Statement

```java
import java.sql.*;

public class Main {

    private static final String url =
            "jdbc:mysql://localhost:3306/mydb";

    private static final String username = "root";
    private static final String password = "password";

    public static void main(String[] args) {

        try {

            Connection connection =
                    DriverManager.getConnection(
                            url,
                            username,
                            password
                    );

            Statement statement =
                    connection.createStatement();

            String query =
                    "DELETE FROM employee WHERE id=3";

            int rows =
                    statement.executeUpdate(query);

            System.out.println(
                    rows + " row deleted successfully."
            );

            statement.close();
            connection.close();

        } catch (Exception e) {

            e.printStackTrace();

        }
    }
}
```

---

Database Before

```text
1 Usama
2 Ali
3 Sara
4 Aman
```

After Delete

```text
1 Usama
2 Ali
4 Aman
```

---

# Example 4: CREATE TABLE using Statement

```java
import java.sql.*;

public class Main {

    private static final String url =
            "jdbc:mysql://localhost:3306/mydb";

    private static final String username = "root";
    private static final String password = "password";

    public static void main(String[] args) {

        try {

            Connection connection =
                    DriverManager.getConnection(
                            url,
                            username,
                            password
                    );

            Statement statement =
                    connection.createStatement();

            String query =
                    "CREATE TABLE student(" +
                    "id INT PRIMARY KEY AUTO_INCREMENT," +
                    "name VARCHAR(100)," +
                    "age INT)";

            statement.executeUpdate(query);

            System.out.println(
                    "Table Created Successfully."
            );

            statement.close();
            connection.close();

        } catch (Exception e) {

            e.printStackTrace();

        }
    }
}
```

---

Why didn't we store the return value?

Because DDL statements like `CREATE TABLE` don't affect existing rows.

The important thing is whether the command succeeds.

---

# Example 5: DROP TABLE using Statement

```java
import java.sql.*;

public class Main {

    private static final String url =
            "jdbc:mysql://localhost:3306/mydb";

    private static final String username = "root";
    private static final String password = "password";

    public static void main(String[] args) {

        try {

            Connection connection =
                    DriverManager.getConnection(
                            url,
                            username,
                            password
                    );

            Statement statement =
                    connection.createStatement();

            String query =
                    "DROP TABLE student";

            statement.executeUpdate(query);

            System.out.println(
                    "Table Deleted Successfully."
            );

            statement.close();
            connection.close();

        } catch (Exception e) {

            e.printStackTrace();

        }
    }
}
```

---

# Difference Between `executeQuery()` and `executeUpdate()`

| Feature       | `executeQuery()` | `executeUpdate()`                                     |
| ------------- | ---------------- | ----------------------------------------------------- |
| Used For      | SELECT           | INSERT, UPDATE, DELETE, CREATE, DROP, ALTER, TRUNCATE |
| Returns       | ResultSet        | int                                                   |
| Data Returned | Yes              | No                                                    |
| Affected Rows | No               | Yes (for DML statements)                              |

---

# Internal Flow

```text
Java Program
      │
      ▼
Create Connection
      │
      ▼
Create Statement
      │
      ▼
Write SQL Query
      │
      ▼
executeUpdate()
      │
      ▼
MySQL Server
      │
      ▼
Database Updated
      │
      ▼
Return Number of Affected Rows
      │
      ▼
Print Result
      │
      ▼
Close Statement
      │
      ▼
Close Connection
```

---

# Common Beginner Mistakes

### 1. Using `executeQuery()` for INSERT

```java
statement.executeQuery("INSERT ...");
```

❌ Wrong

Use

```java
statement.executeUpdate("INSERT ...");
```

---

### 2. Forgetting the `WHERE` clause

```sql
UPDATE employee
SET department='IT';
```

This updates **every row** in the table.

---

### 3. Deleting all records accidentally

```sql
DELETE FROM employee;
```

This deletes every record because no `WHERE` condition is provided.

---

### 4. Not closing resources

Always close:

```java
statement.close();
connection.close();
```

to release database resources.

---

# Interview Questions

### Why does `executeUpdate()` return an `int`?

Because it indicates the number of rows affected by DML operations such as INSERT, UPDATE, and DELETE.

---

### Can `executeUpdate()` execute a SELECT query?

No. SELECT should be executed using `executeQuery()`.

---

### Can `Statement` execute any SQL statement?

Yes. `Statement` can execute SQL statements, but building SQL by concatenating user input is unsafe and can lead to SQL injection. That's why `PreparedStatement` is preferred when user input is involved.

---
* Why `PreparedStatement` was introduced
* Advantages of `PreparedStatement` over `Statement`
