## Connection Pooling in JDBC

**Connection Pooling** means creating a group (pool) of database connections in advance and **reusing them** instead of creating a new connection for every request.

### 1. Without Connection Pooling

Suppose 10 users send requests:

```text
Request 1 → Create Connection → Use → Close
Request 2 → Create Connection → Use → Close
Request 3 → Create Connection → Use → Close
...
Request 10 → Create Connection → Use → Close
```

Every time:

```java
Connection con = DriverManager.getConnection(url, user, password);
```

A new database connection is created.

Creating a database connection is relatively expensive because it involves network communication, authentication, database resources, etc.

---

## 2. With Connection Pooling

Instead, we create connections beforehand:

```text
             CONNECTION POOL
        ┌──────────────────────┐
        │ Connection 1         │
        │ Connection 2         │
        │ Connection 3         │
        │ Connection 4         │
        │ Connection 5         │
        └──────────────────────┘
                  ↓
              Application
```

When a request needs a connection:

```text
Request → Take Connection from Pool
              ↓
           Use it
              ↓
       Return to Pool
```

**Important:** `connection.close()` usually does **not actually destroy the physical connection** when using a connection pool. It returns the connection to the pool.

---

# 3. Real-Life Example

Think of a restaurant.

### Without pooling

Every customer:

```text
Customer → Buy a new chair → Sit → Throw chair away
```

Obviously wasteful. 😄

### With pooling

Restaurant has:

```text
10 chairs
```

Customer:

```text
Take chair → Sit → Finish → Return chair
```

Next customer uses the same chair.

That's essentially what connection pooling does.

---

# 4. What if we create only 5 connections?

Suppose:

```text
Pool Size = 5
```

Initially:

```text
C1 → Available
C2 → Available
C3 → Available
C4 → Available
C5 → Available
```

Now 3 requests arrive:

```text
Request 1 → C1
Request 2 → C2
Request 3 → C3
```

Pool:

```text
C1 → Busy
C2 → Busy
C3 → Busy
C4 → Available
C5 → Available
```

Then two more requests:

```text
Request 4 → C4
Request 5 → C5
```

Now:

```text
C1 → Busy
C2 → Busy
C3 → Busy
C4 → Busy
C5 → Busy
```

---

# 5. What happens when the 6th request comes?

This is the important part.

If the pool has:

```text
Maximum connections = 5
```

and all 5 are currently being used:

```text
Request 6
    ↓
No connection available
    ↓
Wait
```

It normally **waits for a connection to become available**.

When Request 1 finishes:

```java
connection.close();
```

the connection is returned to the pool:

```text
C1 → Available
```

Then Request 6 can take it:

```text
Request 6 → C1
```

So the request isn't necessarily "lost."

---

# 6. Minimum vs Maximum Connections

Suppose:

```text
minimumIdle = 5
maximumPoolSize = 10
```

It means:

### Minimum = 5

The pool tries to maintain around **5 idle/ready connections**.

It does **NOT** mean:

> Only 5 connections can ever exist.

### Maximum = 10

The pool can grow up to:

```text
10 connections
```

depending on demand.

Example:

```text
Low traffic

5 connections
↓
2 used
3 idle
```

Then traffic increases:

```text
5 connections
↓
5 used
```

More requests arrive:

```text
6th request
↓
Pool creates another connection
```

Eventually:

```text
C1
C2
C3
C4
C5
C6
C7
C8
C9
C10
```

Maximum reached.

---

# 7. What happens after 10?

Suppose:

```text
maximumPoolSize = 10
```

and:

```text
10 connections = BUSY
```

Request 11 arrives:

```text
Request 11
     ↓
No connection available
     ↓
WAIT
```

If one of the existing requests finishes:

```text
C4 → returned to pool
```

then:

```text
Request 11 → C4
```

If a connection doesn't become available within the configured timeout, the pool can throw a timeout exception.

---

# 8. Simple Architecture

```text
                 Java Application
                        │
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
     Request 1       Request 2       Request 3
        │               │               │
        └───────────────┼───────────────┘
                        ↓
                 CONNECTION POOL
              ┌─────────────────┐
              │ C1  C2  C3  C4  │
              │ C5  C6  C7 ...  │
              └─────────────────┘
                        │
                        ↓
                    MySQL DB
```

---

# 9. Why Connection Pooling?

### Without pooling

```text
Create connection
       ↓
Use connection
       ↓
Destroy connection
       ↓
Create another connection
       ↓
...
```

More overhead.

### With pooling

```text
Get existing connection
       ↓
Use connection
       ↓
Return connection
       ↓
Reuse
```

Benefits:

* ⚡ Faster
* 🔄 Connections are reused
* 💾 Less database overhead
* 👥 Handles concurrent requests better
* 🛡️ Controls maximum database connections
* 📈 Better scalability

---

# 10. Important JDBC Point

JDBC itself provides the concept of:

```java
DataSource
```

Connection-pooling libraries implement this concept.

A popular library is **HikariCP**.

Conceptually:

```java
DataSource
     ↓
Connection Pool
     ↓
Database Connections
```

Then application code becomes:

```java
Connection con = dataSource.getConnection();

try {
    // database operations
} finally {
    con.close();
}
```

With pooling, this:

```java
con.close();
```

means:

> "I'm finished with this connection. Return it to the pool."

rather than necessarily:

> "Destroy the database connection."

---

HikariCP is a JDBC connection-pooling library. Its standard setup uses `HikariConfig` and `HikariDataSource`. ([GitHub][1])

## 1. What we are going to build

We'll make a simple program:

```text
Java Application
       ↓
HikariCP Connection Pool
       ↓
MySQL
```

Configuration:

```text
Minimum idle connections = 2
Maximum pool size        = 5
```

We'll then request connections and observe how the pool reuses them.

---

# 2. Required JAR files — NO MAVEN

You need:

### HikariCP JAR

Download the HikariCP JAR from Maven Central. For example, HikariCP 5.0.0 is published as `com.zaxxer:HikariCP:5.0.0`. ([Maven Central][2])

[HikariCP on Maven Central](https://central.sonatype.com/artifact/com.zaxxer/HikariCP?utm_source=chatgpt.com)

### MySQL Connector/J

You also need the MySQL JDBC driver because HikariCP itself is **not** the MySQL driver.

So your IntelliJ project should have:

```text
project
│
├── lib
│    ├── HikariCP-5.0.0.jar
│    └── mysql-connector-j-....jar
│
└── src
     └── Main.java
```

Then in IntelliJ:

```text
File
 ↓
Project Structure
 ↓
Libraries
 ↓
+
 ↓
Java
 ↓
select the JAR files
 ↓
Apply
 ↓
OK
```

---

# 3. MySQL table

Create a database:

```sql
CREATE DATABASE college;
```

Then:

```sql
USE college;
```

Create a table:

```sql
CREATE TABLE student (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    marks DOUBLE
);
```

Insert some data:

```sql
INSERT INTO student VALUES
(1, 'John', 98.5),
(2, 'Jack', 90.5),
(3, 'Oggy', 80.0);
```

---

# 4. Complete Java Code

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class Main {

    public static void main(String[] args) {

        // 1. Create Hikari configuration
        HikariConfig config = new HikariConfig();

        // 2. Database details
        config.setJdbcUrl("jdbc:mysql://localhost:3306/college");
        config.setUsername("root");
        config.setPassword("root");

        // 3. Connection pool settings
        config.setMinimumIdle(2);
        config.setMaximumPoolSize(5);

        // 4. Create the connection pool
        HikariDataSource dataSource = new HikariDataSource(config);

        // 5. Get a connection from the pool
        try (Connection connection = dataSource.getConnection()) {

            System.out.println("Connection obtained!");

            // 6. SQL query
            String sql = "SELECT * FROM student";

            // 7. Create PreparedStatement
            PreparedStatement ps = connection.prepareStatement(sql);

            // 8. Execute query
            ResultSet rs = ps.executeQuery();

            // 9. Read result
            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                double marks = rs.getDouble("marks");

                System.out.println(id + " " + name + " " + marks);
            }

        } catch (Exception e) {
            e.printStackTrace();
        }

        // 10. Close the pool
        dataSource.close();
    }
}
```

---

# 5. Now understand EVERY line

## Line 1

```java
import com.zaxxer.hikari.HikariConfig;
```

`HikariConfig` belongs to HikariCP.

It is used to **configure our connection pool**.

Think:

```text
HikariConfig
     ↓
"How should my pool behave?"
```

---

## Line 2

```java
import com.zaxxer.hikari.HikariDataSource;
```

`HikariDataSource` represents our **connection pool/data source**.

This is the object through which our application asks for database connections.

---

## JDBC imports

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
```

These are normal JDBC classes.

HikariCP doesn't replace JDBC.

It sits **on top of the JDBC connection acquisition process**.

```text
Your Java Code
      ↓
HikariCP
      ↓
JDBC Driver
      ↓
MySQL
```

---

# 6. Create configuration

```java
HikariConfig config = new HikariConfig();
```

We're creating a configuration object.

Currently:

```text
config
  ↓
empty configuration
```

Now we start telling HikariCP what to do.

---

# 7. Database URL

```java
config.setJdbcUrl(
    "jdbc:mysql://localhost:3306/college"
);
```

This tells HikariCP:

> "Use this JDBC URL to create database connections."

It is the same URL you already know from normal JDBC:

```java
DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/college",
    "root",
    "root"
);
```

---

# 8. Username

```java
config.setUsername("root");
```

Database username:

```text
root
```

---

# 9. Password

```java
config.setPassword("root");
```

Database password.

---

# 10. Minimum idle

```java
config.setMinimumIdle(2);
```

This tells HikariCP:

> Try to keep around **2 idle connections** available.

Conceptually:

```text
Pool

C1 → IDLE
C2 → IDLE
```

These are already connected to MySQL.

So if a request comes:

```text
Request
   ↓
getConnection()
   ↓
C1
```

It can use an existing connection.

---

# 11. Maximum pool size

```java
config.setMaximumPoolSize(5);
```

This is much more important.

It means:

> The pool cannot have more than **5 physical connections in use/managed by the pool at once**.

Conceptually:

```text
C1
C2
C3
C4
C5
```

Maximum:

```text
5
```

If all 5 are being used and another request comes:

```text
Request 6
    ↓
No connection available
    ↓
Wait
```

It doesn't automatically create:

```text
C6
```

because we said:

```text
maximumPoolSize = 5
```

---

# 12. Create the pool

```java
HikariDataSource dataSource =
        new HikariDataSource(config);
```

This is the **important line**.

We're saying:

> "HikariCP, create a DataSource using this configuration."

Behind the scenes, HikariCP now manages the pool.

Conceptually:

```text
dataSource
     ↓
┌─────────────────┐
│ Connection Pool │
│                 │
│ C1              │
│ C2              │
│ ...             │
│ maximum = 5     │
└─────────────────┘
```

The HikariCP documentation also shows creating a `HikariConfig` and then passing it to `HikariDataSource`. ([GitHub][1])

---

# 13. Get a connection

```java
Connection connection = dataSource.getConnection();
```

This looks very similar to:

```java
DriverManager.getConnection(...);
```

But there is a **huge difference**.

### Normal JDBC

```text
DriverManager
      ↓
Create physical connection
      ↓
MySQL
```

### HikariCP

```text
dataSource
      ↓
Check pool
      ↓
Is an idle connection available?
      ↓
YES
      ↓
Give it to application
```

So:

```java
dataSource.getConnection();
```

means:

> "Give me one connection from the pool."

---

# 14. Why `try-with-resources`?

```java
try (Connection connection = dataSource.getConnection()) {
```

This is extremely important when teaching connection pooling.

When we're finished:

```java
connection.close();
```

is effectively:

```text
Return connection to pool
```

rather than necessarily destroying the underlying physical database connection.

So:

```text
Application
     ↓
getConnection()
     ↓
C1
     ↓
use C1
     ↓
close()
     ↓
C1 returned to pool
```

Then another request can use C1.

---

# 15. SQL

```java
String sql = "SELECT * FROM student";
```

Nothing special here.

This is normal JDBC.

---

# 16. PreparedStatement

```java
PreparedStatement ps =
        connection.prepareStatement(sql);
```

Again, normal JDBC.

We're using the connection obtained from HikariCP.

---

# 17. Execute

```java
ResultSet rs = ps.executeQuery();
```

Normal JDBC.

---

# 18. Read results

```java
while (rs.next()) {
```

Move through each database row.

Then:

```java
int id = rs.getInt("id");
```

Gets the `id`.

```java
String name = rs.getString("name");
```

Gets the name.

```java
double marks = rs.getDouble("marks");
```

Gets marks.

---

# 19. Close the connection

Because we used:

```java
try (Connection connection = ...)
```

Java automatically calls:

```java
connection.close();
```

when the `try` block finishes.

With HikariCP:

```text
connection.close()
       ↓
HikariCP intercepts it
       ↓
Connection returned to pool
```

---

# 20. Finally close the pool

```java
dataSource.close();
```

This is different from:

```java
connection.close();
```

### `connection.close()`

Means:

> "I'm done using this connection."

It goes back to the pool.

### `dataSource.close()`

Means:

> "I'm shutting down the entire connection pool."

Then HikariCP can close its physical database connections.

```text
dataSource.close()
       ↓
C1 → close
C2 → close
C3 → close
C4 → close
C5 → close
       ↓
Pool shutdown
```

---

# 21. The complete lifecycle

This is what I'd draw on the board:

```text
                    APPLICATION
                         │
                         │
                         ▼
                HikariDataSource
                         │
                         ▼
                 CONNECTION POOL
              ┌──────────────────┐
              │ C1 → IDLE        │
              │ C2 → IDLE        │
              │ C3 → IDLE        │
              │ C4 → IDLE        │
              │ C5 → IDLE        │
              └──────────────────┘
                         │
                         ▼
                       MySQL
```

Request comes:

```text
Request 1
    │
    ▼
getConnection()
    │
    ▼
C1
    │
    ▼
Execute SQL
    │
    ▼
close()
    │
    ▼
C1 returns to pool
```

Next request:

```text
Request 2
    │
    ▼
getConnection()
    │
    ▼
C1 again
```

**That's the whole point of connection pooling.**

---

# 22. Now demonstrate the pool with 7 requests

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

import java.sql.Connection;

public class Main {

    public static void main(String[] args) {

        HikariConfig config = new HikariConfig();

        config.setJdbcUrl("jdbc:mysql://localhost:3306/college");
        config.setUsername("root");
        config.setPassword("root");

        config.setMinimumIdle(2);
        config.setMaximumPoolSize(5);

        HikariDataSource dataSource =
                new HikariDataSource(config);

        try {

            for (int i = 1; i <= 7; i++) {

                Connection connection =
                        dataSource.getConnection();

                System.out.println(
                        "Request " + i +
                        " got connection: " +
                        connection
                );

                connection.close();

                System.out.println(
                        "Request " + i +
                        " returned connection"
                );
            }

        } catch (Exception e) {
            e.printStackTrace();
        }

        dataSource.close();
    }
}
```
```text
Request 1 → C1
Request 2 → C1
Request 3 → C1
Request 4 → C1
...
```

Why?

Because you're doing:

```java
connection.close();
```

after every request.

The connection is immediately returned to the pool and can be reused.

---

## 23. To actually demonstrate 5 connections

Instead, **don't immediately close them**:

```java
Connection c1 = dataSource.getConnection();
Connection c2 = dataSource.getConnection();
Connection c3 = dataSource.getConnection();
Connection c4 = dataSource.getConnection();
Connection c5 = dataSource.getConnection();

System.out.println("5 connections acquired.");

Connection c6 = dataSource.getConnection();

System.out.println("6th connection acquired.");
```

```text
C1 → BUSY
C2 → BUSY
C3 → BUSY
C4 → BUSY
C5 → BUSY
```

Then:

```java
Connection c6 = dataSource.getConnection();
```

The 6th request **waits**, because:

```text
maximumPoolSize = 5
```

If you then do:

```java
c1.close();
```

C1 returns to the pool:

```text
C1 → AVAILABLE
```

and the waiting request can receive it.

---

### One important teaching point

> "`minimumIdle = 2` means HikariCP always creates exactly 2 connections immediately."

That's too simplistic.

Better:

> **`minimumIdle` is the target number of idle connections HikariCP tries to maintain, while `maximumPoolSize` limits the pool's total size.**


[1]: https://github.com/openbouquet/HikariCP/blob/master/README.md?utm_source=chatgpt.com "HikariCP/README.md at master · openbouquet/HikariCP · GitHub"
[2]: https://central.sonatype.com/artifact/com.zaxxer/HikariCP/5.0.0?utm_source=chatgpt.com "Maven Central: com.zaxxer:HikariCP:5.0.0"
Exactly. The **real-world flow** is much easier to understand if we stop thinking about a single Java `main()` and imagine a web application with hundreds of users.

## Real-world example

Suppose you have an online shopping application:

```text
                    USERS
        ┌──────────┬──────────┬──────────┐
        ↓          ↓          ↓          ↓
      User 1     User 2     User 3    User 1000
        │          │          │          │
        └──────────┴──────────┴──────────┘
                       ↓
                Java Web Application
                       ↓
                HikariCP Pool
                       ↓
                    MySQL
```

The important thing is:

> **Users do NOT each get their own permanent database connection.**

Instead, connections are shared.

---

# 1. Application starts

Suppose the developer configures:

```java
config.setMinimumIdle(5);
config.setMaximumPoolSize(20);
```

When the application starts, HikariCP manages a pool that aims to keep idle connections available.

Conceptually:

```text
HikariCP
┌─────────────────────────┐
│ C1  IDLE                │
│ C2  IDLE                │
│ C3  IDLE                │
│ C4  IDLE                │
│ C5  IDLE                │
└─────────────────────────┘
```

These are **physical connections to MySQL**.

---

# 2. User 1 logs in

User sends:

```text
POST /login
```

Your Java application needs the database:

```java
Connection con = dataSource.getConnection();
```

HikariCP says:

```text
"Do I have an idle connection?"
        ↓
       YES
        ↓
Give C1 to application
```

Now:

```text
C1 → BUSY
C2 → IDLE
C3 → IDLE
C4 → IDLE
C5 → IDLE
```

Your code does:

```sql
SELECT * FROM users WHERE email = ?
```

---

# 3. Login finishes

Your code does:

```java
con.close();
```

It does **not normally destroy the physical MySQL connection**.

Instead:

```text
C1 → returned to HikariCP
```

Pool:

```text
C1 → IDLE
C2 → IDLE
C3 → IDLE
C4 → IDLE
C5 → IDLE
```

Another user can use C1.

---

# 4. Now 10 users arrive together

Imagine:

```text
User 1  → Request
User 2  → Request
User 3  → Request
...
User 10 → Request
```

HikariCP gives out connections:

```text
User 1 → C1
User 2 → C2
User 3 → C3
User 4 → C4
User 5 → C5
```

The pool needs more:

```text
User 6 → C6
User 7 → C7
User 8 → C8
User 9 → C9
User 10 → C10
```

Now:

```text
C1  BUSY
C2  BUSY
C3  BUSY
C4  BUSY
C5  BUSY
C6  BUSY
C7  BUSY
C8  BUSY
C9  BUSY
C10 BUSY
```

The pool can continue growing until:

```text
20 connections
```

because:

```java
maximumPoolSize = 20;
```

---

# 5. What if 21 users request the database?

This is the important part.

Suppose:

```text
C1 → BUSY
C2 → BUSY
...
C20 → BUSY
```

Now:

```text
User 21
    ↓
getConnection()
    ↓
No idle connection
    ↓
Pool already at maximum
    ↓
WAIT
```

HikariCP does **not** create:

```text
C21 ❌
```

because:

```text
maximumPoolSize = 20
```

Instead, the request waits for a connection to be returned, subject to the configured connection timeout.

---

# 6. User 3 finishes

Suppose User 3 finishes:

```java
connection.close();
```

Then:

```text
C3 → IDLE
```

HikariCP can immediately give C3 to the waiting request:

```text
User 21
    ↓
C3
```

So:

```text
User 21 doesn't get a NEW connection.
User 21 gets a REUSED connection.
```

That's the whole power of pooling.

---

# 7. The REALLY important real-world point

You might ask:

> "If 1000 users are online, do we need 1000 connections?"

**No.**

For example:

```text
1000 users
       ↓
Java application
       ↓
20 database connections
       ↓
MySQL
```

Users don't continuously hold connections.

A typical request might be:

```text
HTTP request
     ↓
Get connection
     ↓
Run SQL
     ↓
Close/return connection
     ↓
Send HTTP response
```

The connection is held only while database work is happening.

---

# 8. Imagine an e-commerce website

Suppose 500 people are browsing Amazon-like pages.

Most users are doing things like:

```text
Read product page
↓
No DB connection needed after query finishes

Read product page
↓
No DB connection needed

Add to cart
↓
Need DB connection
↓
SQL
↓
Return connection

Browse again
↓
No DB connection
```

Therefore:

```text
500 users
      ↓
maybe only 10–30 DB connections
      ↓
MySQL
```

The exact pool size depends on workload, database capacity, application architecture, query latency, number of application instances, etc. **You don't simply choose the pool size based on number of users.**

---

# 9. What happens with multiple application servers?

This is where real production systems get interesting.

Suppose your application has 3 servers:

```text
                    Load Balancer
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Server 1       Server 2       Server 3
       HikariCP       HikariCP       HikariCP
       max=20         max=20         max=20
          │              │              │
          └──────────────┼──────────────┘
                         ↓
                       MySQL
```

Now your maximum is potentially:

```text
20 + 20 + 20 = 60
```

**not 20.**

This is a very important production concept.

If you have 10 application servers and each has:

```text
maximumPoolSize = 20
```

then potentially:

```text
10 × 20 = 200
```

database connections.

So the pool size must also consider the **database's connection capacity**.

---

# 10. What happens when the application shuts down?

Suppose you deploy a new version.

The application shuts down:

```text
Application
    ↓
dataSource.close()
    ↓
HikariCP shutdown
    ↓
Physical DB connections closed
```

Then the new application instance starts and creates/manages its pool again.

---

# 11. So what is HikariCP actually doing?

Think of HikariCP as a **traffic manager for database connections**.

Your application says:

```text
"I need a connection."
```

HikariCP:

```text
"Here, take C7."
```

Application:

```text
"Finished."
```

HikariCP:

```text
"Okay, I'll keep C7 and give it
to somebody else later."
```

Another request:

```text
"I need a connection."
```

HikariCP:

```text
"Take C7 again."
```

So:

```text
             REQUESTS
     1  2  3  4  5  6  7  8 ...
     ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓
       HikariCP
     ┌───────────────────┐
     │ C1 C2 C3 ... C20  │
     └───────────────────┘
              ↓
            MySQL
```


Suppose your application has 1,000 registered users. They are stored in your **`users` table**.

### Login flow

User enters:

```text
Email: john@gmail.com
Password: 1234
```

Your Java application receives these values:

```java
String email = "john@gmail.com";
String password = "1234";
```

Then your application asks HikariCP for a **database connection**:

```java
Connection con = dataSource.getConnection();
```

Then you use that connection to check the application's user:

```java
String sql =
    "SELECT * FROM users WHERE email = ? AND password = ?";

PreparedStatement ps = con.prepareStatement(sql);

ps.setString(1, email);
ps.setString(2, password);

ResultSet rs = ps.executeQuery();
```

So there are **two completely different "users"**:

| User             | Meaning                          |
| ---------------- | -------------------------------- |
| `root`           | MySQL database account           |
| `john@gmail.com` | Your application's customer/user |

---

### Behind the scenes

```text
John enters login
        ↓
email + password
        ↓
Java application
        ↓
dataSource.getConnection()
        ↓
HikariCP
        ↓
Connection C3
        ↓
MySQL
        ↓
SELECT user WHERE email = ?
        ↓
John's record
        ↓
Login successful
        ↓
connection.close()
        ↓
C3 returns to pool
```

The important part is:

**John does NOT get C3 permanently.**

C3 belongs to the **application's connection pool**.

Five seconds later, another user might log in:

```text
Alice
 ↓
getConnection()
 ↓
C3
```

The **same physical database connection** can be reused for Alice.

That's why connection pooling is so useful.

## What is a Properties File?

A `.properties` file is a simple text file used to store **configuration values as key-value pairs**.

Example: `config.properties`

```properties
db.url=jdbc:mysql://localhost:3306/mydb
db.username=root
db.password=password
app.name=Student Management System
```

Instead of writing these values directly in Java:

```java
String url = "jdbc:mysql://localhost:3306/mydb";
String username = "root";
String password = "password";
```

we can keep them in a properties file.

---

# 2. Creating a Properties File

In IntelliJ, create:

```text
src
 └── config.properties
```

Put:

```properties
name=Usama
age=25
course=Java
city=Lucknow
```

---

# 3. Reading Properties File

Java provides the `Properties` class.

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

public class Main {

    public static void main(String[] args) {

        Properties properties = new Properties();

        try {
            FileInputStream fis =
                    new FileInputStream("src/config.properties");

            properties.load(fis);

            String name = properties.getProperty("name");
            String age = properties.getProperty("age");
            String course = properties.getProperty("course");

            System.out.println(name);
            System.out.println(age);
            System.out.println(course);

            fis.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### Important methods

| Method          | Purpose                      |
| --------------- | ---------------------------- |
| `load()`        | Loads properties from a file |
| `getProperty()` | Gets value using key         |
| `setProperty()` | Adds/updates a property      |
| `remove()`      | Removes a property           |
| `store()`       | Saves properties to a file   |

---

# 4. Better Way — Try-with-Resources

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

public class Main {

    public static void main(String[] args) {

        Properties properties = new Properties();

        try (FileInputStream fis =
                     new FileInputStream("src/config.properties")) {

            properties.load(fis);

            System.out.println(properties.getProperty("name"));
            System.out.println(properties.getProperty("course"));

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

The file is automatically closed.

---

# 5. What if the Property Doesn't Exist?

```java
String country = properties.getProperty("country");

System.out.println(country);
```

Output:

```text
null
```

We can provide a default value:

```java
String country =
        properties.getProperty("country", "India");

System.out.println(country);
```

Output:

```text
India
```

---

# 6. Writing to a Properties File

Suppose we want to create properties programmatically.

```java
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Properties;

public class Main {

    public static void main(String[] args) {

        Properties properties = new Properties();

        properties.setProperty("name", "John");
        properties.setProperty("age", "22");
        properties.setProperty("course", "Java");

        try (FileOutputStream fos =
                     new FileOutputStream("src/config.properties")) {

            properties.store(fos, "Student Information");

            System.out.println("Properties saved successfully");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

The file becomes something like:

```properties
#Student Information
#Mon Aug 10 12:47:00 IST 2026
name=John
age=22
course=Java
```

---

# 7. Updating a Property

```java
properties.setProperty("age", "23");
```

If `age` already exists, it gets updated.

```java
properties.setProperty("course", "Advanced Java");
```

---

# 8. Removing a Property

```java
properties.remove("age");
```

---

# 9. Practical Example — Database Configuration

### `db.properties`

```properties
db.url=jdbc:mysql://localhost:3306/mydb
db.username=root
db.password=password
```

Java:

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

public class Main {

    public static void main(String[] args) {

        Properties properties = new Properties();

        try (FileInputStream fis =
                     new FileInputStream("src/db.properties")) {

            properties.load(fis);

            String url = properties.getProperty("db.url");
            String username = properties.getProperty("db.username");
            String password = properties.getProperty("db.password");

            System.out.println(url);
            System.out.println(username);
            System.out.println(password);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

# Part 2 — Basic File Handling

After Properties, move into normal file handling.

Java's modern basic API is `java.nio.file`.

## 10. Creating a File

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {

    public static void main(String[] args) {

        Path path = Path.of("student.txt");

        try {
            Files.createFile(path);

            System.out.println("File created");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

# 11. Writing to a File

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {

    public static void main(String[] args) {

        Path path = Path.of("student.txt");

        try {
            Files.writeString(path, "John\nJack\nBob");

            System.out.println("Data written");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

`student.txt`

```text
John
Jack
Bob
```

---

# 12. Reading a File

```java
String data = Files.readString(path);

System.out.println(data);
```

Complete:

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {

    public static void main(String[] args) {

        Path path = Path.of("student.txt");

        try {

            String data = Files.readString(path);

            System.out.println(data);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

# 13. Append Data

Suppose the file contains:

```text
John
Jack
Bob
```

We want to add:

```text
Oggy
```

Use:

```java
Files.writeString(
        path,
        "\nOggy",
        StandardOpenOption.APPEND
);
```

Need:

```java
import java.nio.file.StandardOpenOption;
```

---

# 14. Checking Whether File Exists

```java
if (Files.exists(path)) {
    System.out.println("File exists");
} else {
    System.out.println("File does not exist");
}
```

---

# 15. Delete a File

```java
Files.delete(path);
```

Safer:

```java
Files.deleteIfExists(path);
```

---

# 16. Basic File Information

```java
System.out.println(Files.exists(path));
System.out.println(Files.isRegularFile(path));
System.out.println(Files.size(path));
```

---

# 17. Working with Directories

Create directory:

```java
Path path = Path.of("students");

Files.createDirectory(path);
```

Check:

```java
if (Files.exists(path)) {
    System.out.println("Directory exists");
}
```

---

# 18. List Files in a Directory

```java
Path directory = Path.of("students");

try (var files = Files.list(directory)) {

    files.forEach(System.out::println);

} catch (IOException e) {
    e.printStackTrace();
}
```

---

# Recommended Teaching Flow

For **first-time learners**, I'd structure the lecture like this:

### Properties Files

1. Why configuration should be separated from Java code
2. What is `.properties`
3. Key-value structure
4. `Properties`
5. `load()`
6. `getProperty()`
7. Default values
8. `setProperty()`
9. `store()`
10. `remove()`
11. Database configuration example

### File Handling

12. What is a file?
13. `Path`
14. `Files`
15. Create file
16. Write file
17. Read file
18. Append file
19. Check existence
20. Delete file
21. Create directory
22. List directory contents

### One final mini-project

Have students build:

**Student Configuration + File Logger**

```text
student-app/
│
├── config.properties
├── students.txt
└── Main.java
```

`config.properties`

```properties
college=PW IOI
course=BTech CSE
```

`students.txt`

```text
John
Jack
Bob
Oggy
```
