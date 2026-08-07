# Lecture 06: Batch Operations in JDBC

---

# What is Batch Processing?

## Definition

**Batch Processing** is a JDBC feature that allows us to execute **multiple SQL statements together as a single batch**, instead of sending them to the database one by one.

Instead of

```text
Java → Database
Java → Database
Java → Database
Java → Database
```

We send

```text
Java
   │
   ▼
Batch of SQL Statements
   │
   ▼
Database
```

The database executes all of them together.

---

# Why Do We Need Batch Processing?

Imagine you have to insert **10,000 students** into the database.

Without Batch Processing

```text
Insert Student 1

↓

Database

↓

Insert Student 2

↓

Database

↓

Insert Student 3

↓

Database

↓

...

↓

Insert Student 10000
```

Every insert requires

* Sending SQL
* Waiting for database
* Executing query
* Returning response

Thousands of network trips.

Very slow.

---

# With Batch Processing

Java collects all SQL statements first.

```text
Insert 1

Insert 2

Insert 3

...

Insert 10000
```

Then sends everything together.

```text
Java

↓

Batch

↓

Database
```

Much faster.

---

# Real-Life Analogy

Suppose you need to post **100 letters**.

### Without Batch

Go to the post office

↓

Submit one letter

↓

Come back

↓

Repeat 100 times.

Very slow.

---

### With Batch

Collect all 100 letters.

Go once.

Submit all together.

Done.

Exactly how Batch Processing works.

---

# Advantages of Batch Processing

* Faster execution
* Fewer network calls
* Better performance
* Less communication overhead
* Suitable for bulk data insertion
* Easy to execute thousands of SQL statements

---

# JDBC Methods Used

| Method           | Purpose                               |
| ---------------- | ------------------------------------- |
| `addBatch()`     | Adds SQL statement to batch           |
| `executeBatch()` | Executes all SQL statements together  |
| `clearBatch()`   | Removes all statements from the batch |

---

# Batch using Statement

Suppose we want to insert multiple employees.

## Java Program

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

            statement.addBatch(
                    "INSERT INTO employee(name,age,department) VALUES('Usama',24,'IT')"
            );

            statement.addBatch(
                    "INSERT INTO employee(name,age,department) VALUES('Ali',22,'HR')"
            );

            statement.addBatch(
                    "INSERT INTO employee(name,age,department) VALUES('Sara',26,'Finance')"
            );

            int[] result =
                    statement.executeBatch();

            System.out.println(
                    "Batch Executed Successfully."
            );

            statement.close();

            connection.close();

        }

        catch(Exception e){

            e.printStackTrace();

        }

    }

}
```

---

# Understanding Every Line

## Create Statement

```java
Statement statement =
connection.createStatement();
```

Creates a Statement object.

---

## addBatch()

```java
statement.addBatch(sql);
```

Does **NOT** execute SQL immediately.

It only stores the SQL inside memory.

Think of it as

```text
Pending Queue
```

---

After first statement

```text
Batch

↓

INSERT 1
```

---

After second statement

```text
Batch

↓

INSERT 1

INSERT 2
```

---

After third

```text
Batch

↓

INSERT 1

INSERT 2

INSERT 3
```

Nothing has been sent to the database yet.

---

## executeBatch()

```java
statement.executeBatch();
```

Now Java sends every SQL statement together.

Database executes

```text
Insert 1

Insert 2

Insert 3
```

All in one batch.

---

# Return Value

```java
int[] result =
statement.executeBatch();
```

Returns an integer array.

Example

```text
[1,1,1]
```

Meaning

```text
First Query

↓

1 row affected

Second Query

↓

1 row affected

Third Query

↓

1 row affected
```

---

# Batch using PreparedStatement

This is the most common approach because it is safer and avoids SQL injection.

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

            String query =
                    "INSERT INTO employee(name,age,department) VALUES(?,?,?)";

            PreparedStatement ps =
                    connection.prepareStatement(query);

            ps.setString(1,"Usama");
            ps.setInt(2,24);
            ps.setString(3,"IT");

            ps.addBatch();

            ps.setString(1,"Ali");
            ps.setInt(2,22);
            ps.setString(3,"HR");

            ps.addBatch();

            ps.setString(1,"Sara");
            ps.setInt(2,26);
            ps.setString(3,"Finance");

            ps.addBatch();

            int[] result =
                    ps.executeBatch();

            System.out.println(
                    "Batch Inserted Successfully."
            );

            ps.close();

            connection.close();

        }

        catch(Exception e){

            e.printStackTrace();

        }

    }

}
```

---

# Why Doesn't `addBatch()` Take SQL Here?

Because

```java
PreparedStatement ps =
connection.prepareStatement(query);
```

already knows the SQL.

Only parameter values change.

Each time we write

```java
ps.addBatch();
```

JDBC stores the current parameter values as one execution in the batch.

---

# Internal Working

After first

```text
VALUES

Usama

24

IT
```

Stored.

---

Second

```text
Ali

22

HR
```

Stored.

---

Third

```text
Sara

26

Finance
```

Stored.

---

Then

```java
executeBatch()
```

executes all three inserts together.

---

# clearBatch()

Suppose

```java
statement.addBatch(sql1);

statement.addBatch(sql2);

statement.addBatch(sql3);
```

Now you decide not to execute them.

Use

```java
statement.clearBatch();
```

Everything stored in the batch is removed.

The database is **not** affected because nothing has been executed yet.

---

# Performance Comparison

Suppose

10,000 inserts.

Without Batch

```text
Java

↓

Database

↓

Java

↓

Database

↓

Java

↓

Database

(10,000 times)
```

---

With Batch

```text
Java

↓

10,000 SQL Statements

↓

Database

(One or a few large requests)
```

Much faster.

---

# Batch Execution Flow

```text
Java Program

↓

Connection

↓

Statement / PreparedStatement

↓

addBatch()

↓

addBatch()

↓

addBatch()

↓

executeBatch()

↓

Database

↓

Return int[]

↓

Close Resources
```

---

# Real-World Uses

Batch Processing is commonly used for

* Importing Excel data
* Importing CSV files
* Student admission records
* Employee payroll upload
* E-commerce order processing
* Banking transactions
* Attendance systems
* Bulk invoice generation

---

# Batch with Transactions (Recommended)

For important operations, disable auto-commit so the entire batch succeeds or fails together.

```java
connection.setAutoCommit(false);

try {

    int[] result = ps.executeBatch();

    connection.commit();

} catch (SQLException e) {

    connection.rollback();

}
```

This ensures data consistency if an error occurs.

---

# Statement vs PreparedStatement Batch

| Feature            | Statement Batch | PreparedStatement Batch               |
| ------------------ | --------------- | ------------------------------------- |
| SQL Injection Safe | ❌ No            | ✅ Yes                                 |
| Reusable SQL       | ❌ No            | ✅ Yes                                 |
| Parameter Support  | ❌ No            | ✅ Yes                                 |
| Recommended        | ❌ Rarely        | ✅ Yes                                 |
| Performance        | Good            | Better for repeated parameterized SQL |

---

# Interview Questions

### What is Batch Processing?

Executing multiple SQL statements together instead of one by one.

---

### Which methods are used?

```java
addBatch();

executeBatch();

clearBatch();
```

---

### What does `executeBatch()` return?

```java
int[]
```

Each element represents the update count for the corresponding SQL statement. In some cases, JDBC drivers may return special values such as `Statement.SUCCESS_NO_INFO` or `Statement.EXECUTE_FAILED`.

---

### Can Batch Processing be used with PreparedStatement?

Yes.

It is the preferred approach because it is secure and efficient.

---

### Why is Batch Processing faster?

Because it reduces the number of round trips between the Java application and the database.

---
