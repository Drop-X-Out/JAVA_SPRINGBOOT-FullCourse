# Lecture 02: Your First JDBC Program (SELECT Operation)

---

# Step 1: Create Database

Open MySQL Command Line or MySQL Workbench.

```sql
CREATE DATABASE mydb;
```

### Explanation

* `CREATE` → SQL keyword used to create a new object.
* `DATABASE` → Specifies that we are creating a database.
* `mydb` → Name of our database.

Now MySQL creates an empty database named **mydb**.

---

# Step 2: Use the Database

```sql
USE mydb;
```

### Explanation

Suppose your computer contains 100 databases.

How does MySQL know which database you want to work with?

Using:

```sql
USE mydb;
```

Now every SQL query will execute inside **mydb**.

---

# Step 3: Create Employee Table

```sql
CREATE TABLE employee(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    age INT,
    department VARCHAR(100)
);
```

---

## Explanation

### id

```sql
id INT
```

Stores employee ID.

Example

```
1
2
3
```

---

### PRIMARY KEY

Every row should be uniquely identified.

```
1  Usama

2  Ali

3  Ahmad
```

Two employees cannot have the same primary key.

---

### AUTO_INCREMENT

Automatically generates

```
1

2

3

4

5
```

No need to insert IDs manually.

---

### name

```sql
VARCHAR(100)
```

Stores employee name.

Maximum 100 characters.

---

### age

Stores employee age.

---

### department

Stores department name.

---

# Step 4: Insert Sample Data

```sql
INSERT INTO employee(name, age, department)
VALUES
('Usama',24,'IT'),
('Ali',22,'HR'),
('Sara',26,'Finance');
```

---

### Explanation

We don't insert **id** because it is **AUTO_INCREMENT**.

MySQL automatically creates

```
1

2

3
```

---

# Verify

```sql
SELECT * FROM employee;
```

Output

```
+----+--------+-----+------------+
| id | name   | age | department |
+----+--------+-----+------------+
| 1  | Usama  | 24  | IT         |
| 2  | Ali    | 22  | HR         |
| 3  | Sara   | 26  | Finance    |
+----+--------+-----+------------+
```

---

# Step 5: Create Java Project

Open IntelliJ

```
File

↓

New Project

↓

Java

↓

JDK 21

↓

Finish
```

Project Name

```
JDBCProject
```

---

# Step 6: Add MySQL Connector/J

Download the MySQL Connector/J JAR and add it to your project.

In IntelliJ:

```
Project Structure

↓

Libraries

↓

+

↓

Java

↓

Select mysql-connector-j.jar

↓

Apply
```

Now Java can communicate with MySQL.

---

# Step 7: Project Structure

```
JDBCProject

│

└── src

      │

      └── Main.java
```

---

# Step 8: First JDBC Program

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

            String query = "SELECT * FROM employee";

            Statement statement =
                    connection.createStatement();

            ResultSet resultSet =
                    statement.executeQuery(query);

            while(resultSet.next()){

                int id =
                        resultSet.getInt("id");

                String name =
                        resultSet.getString("name");

                int age =
                        resultSet.getInt("age");

                String department =
                        resultSet.getString("department");

                System.out.println(
                        id + " "
                        + name + " "
                        + age + " "
                        + department
                );
            }

            resultSet.close();
            statement.close();
            connection.close();

        } catch(Exception e){

            e.printStackTrace();

        }
    }
}
```

---

# Line-by-Line Explanation

---

## Import Statement

```java
import java.sql.*;
```

### Why?

All JDBC interfaces and classes are inside the **java.sql** package.

Without importing this package,

Java won't recognize

* Connection
* Statement
* ResultSet
* DriverManager

---

## Main Class

```java
public class Main
```

Creates a Java class.

Every Java program starts from a class.

---

## Database URL

```java
private static final String url =
"jdbc:mysql://localhost:3306/mydb";
```

Let's divide it.

```
jdbc
```

Means

Use JDBC protocol.

---

```
mysql
```

Database type.

---

```
localhost
```

Database is running on your own computer.

---

```
3306
```

Default MySQL Port Number.

Think of it like a door number through which applications communicate with the MySQL server.

---

```
mydb
```

Database Name.

---

Complete Meaning

```
Use JDBC

↓

Connect to MySQL

↓

Running on Local Machine

↓

Port 3306

↓

Database mydb
```

---

## Username

```java
private static final String username="root";
```

Database username.

---

## Password

```java
private static final String password="password";
```

Database password.

Replace it with your own MySQL password.

---

# Main Method

```java
public static void main(String[] args)
```

Program execution starts here.

---

# Try Block

```java
try{
```

Database operations can fail.

Examples

* Wrong Password
* MySQL Server Not Running
* Internet Issue (Remote Database)
* SQL Error

So we use exception handling.

---

# Loading Driver

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

### What happens?

Java loads the MySQL JDBC Driver into memory.

The driver registers itself with `DriverManager`, making it available to create connections.

**Note:** In modern JDBC (4.0+), this line is usually optional because the driver is loaded automatically if the JAR is on the classpath. We include it here to understand the internal process.

---

# Create Connection

```java
Connection connection =
DriverManager.getConnection(
url,
username,
password
);
```

This is the most important line.

DriverManager asks the registered MySQL driver to connect using:

* URL
* Username
* Password

If everything is correct,

Connection object is returned.

Think of it as

```
Phone Connected

Now you can talk.
```

Without Connection

Nothing is possible.

---

# SQL Query

```java
String query =
"SELECT * FROM employee";
```

Stores SQL query inside a Java String.

Java does not execute it yet.

---

# Create Statement

```java
Statement statement =
connection.createStatement();
```

Connection creates a Statement object.

Statement sends SQL queries to the database.

Think of Statement as

```
Courier

Carries SQL Query

to Database
```

---

# Execute Query

```java
ResultSet resultSet =
statement.executeQuery(query);
```

`executeQuery()` is used only for **SELECT** statements.

It returns a `ResultSet`.

ResultSet contains rows returned by the database.

---

# What is ResultSet?

Think of it as a table in memory.

```
ID

Name

Age

Department
```

It points to rows one by one.

Initially

```
Before First Row
```

---

# Move Cursor

```java
while(resultSet.next())
```

Initially

```
↓

Before First Row
```

First `next()`

```
↓

Row 1
```

Second

```
↓

Row 2
```

Third

```
↓

Row 3
```

Fourth

No row.

Returns **false**.

Loop ends.

---

# Read Integer

```java
int id =
resultSet.getInt("id");
```

Reads

```
id
```

column.

Returns Integer.

---

# Read String

```java
String name =
resultSet.getString("name");
```

Reads name column.

---

Similarly

```java
getInt()

getString()

getDouble()

getBoolean()

getDate()

getTimestamp()
```

Each method matches the column's data type.

---

# Print Data

```java
System.out.println(
id+" "
+name+" "
+age+" "
+department
);
```

Output

```
1 Usama 24 IT

2 Ali 22 HR

3 Sara 26 Finance
```

---

# Close ResultSet

```java
resultSet.close();
```

Frees memory used by the retrieved rows.

---

# Close Statement

```java
statement.close();
```

Stops sending SQL commands.

---

# Close Connection

```java
connection.close();
```

Disconnects from the database.

Always close resources when finished to avoid resource leaks.

---

# Catch Block

```java
catch(Exception e)
```

If anything goes wrong,

control comes here.

---

```java
e.printStackTrace();
```

Prints the complete error details for debugging.

---

# Complete JDBC Flow

```
Java Program

↓

Load Driver

↓

DriverManager

↓

Connection

↓

Statement

↓

SQL Query

↓

Database

↓

ResultSet

↓

Java Program

↓

Print Data

↓

Close Resources
```

---

# Important Interview Questions

### 1. Why do we use `Class.forName()`?

To explicitly load and register the JDBC driver. In JDBC 4.0 and later, it is usually optional because drivers are auto-loaded.

---

### 2. Why do we use `DriverManager`?

It manages registered JDBC drivers and creates database connections.

---

### 3. What is `Connection`?

It represents an active connection (session) between the Java application and the database.

---

### 4. What is `Statement`?

It sends SQL statements to the database.

---

### 5. What is `ResultSet`?

An object that stores the rows returned by a `SELECT` query and lets you iterate through them.

---

### 6. Why is `executeQuery()` used instead of `executeUpdate()`?

Because `SELECT` returns rows of data. `executeUpdate()` is used for `INSERT`, `UPDATE`, `DELETE`, and DDL statements.

---
