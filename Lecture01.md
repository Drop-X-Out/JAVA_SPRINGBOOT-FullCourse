# Lecture 01: Introduction to JDBC (Java Database Connectivity)

---

# 1. What is JDBC?

## Definition

**JDBC (Java Database Connectivity)** is a Java API that allows Java programs to communicate with relational databases.

It provides a standard way to:

* Connect to a database
* Execute SQL queries
* Retrieve data
* Insert data
* Update records
* Delete records
* Execute Stored Procedures
* Manage Transactions

Think of JDBC as a translator between Java and Database.

---

## Real Life Analogy

Imagine:

You are in India.

You want to talk to a Japanese person.

You don't know Japanese.

The Japanese person doesn't know Hindi.

So what do you use?

👉 A Translator.

Exactly the same happens in Java.

Java doesn't understand SQL directly.

Database doesn't understand Java Objects.

JDBC acts as the Translator.

```
Java Program
      ↓
     JDBC
      ↓
Database
```

---

# 2. Why JDBC Needed?

Suppose you have built

* Banking System
* Hospital Management
* Student Portal
* Amazon Clone
* Instagram Clone

Where will the data be stored?

Not inside Java variables.

Variables disappear when the program stops.

We need Permanent Storage.

That permanent storage is Database.

Now Java must communicate with Database.

That communication is done using JDBC.

---

Without JDBC

```
Java Program

Cannot talk to

MySQL
Oracle
PostgreSQL
SQL Server
```

With JDBC

```
Java Program
      ↓
    JDBC API
      ↓
Database
```

---

# Why can't Java directly talk to Database?

Because Java understands

```
Objects
Classes
Methods
Interfaces
```

Database understands

```
SQL

SELECT
INSERT
UPDATE
DELETE
```

Different Languages.

Need Translator.

That Translator is JDBC.

---

# 3. Before JDBC (History)

Before JDBC, every database company had its own API.

Example

Oracle

```
Oracle API
```

MySQL

```
MySQL API
```

Microsoft SQL Server

```
SQL Server API
```

IBM DB2

```
DB2 API
```

Problem?

Every database had different syntax.

If company changed database

Oracle

↓

MySQL

Entire Java code needed modification.

Huge Problem.

---

Example

Today

```
Oracle
```

Tomorrow

```
PostgreSQL
```

Developer had to learn another API.

Not efficient.

---

Sun Microsystems solved this problem.

They introduced

JDBC

One Standard API

Works with every database.

Only Driver changes.

Java code mostly remains same.

---

# Before JDBC

```
Java
   ↓
Oracle API
   ↓
Oracle Database
```

For MySQL

```
Java
   ↓
MySQL API
   ↓
MySQL Database
```

Different code.

---

After JDBC

```
Java

↓

JDBC API

↓

Driver

↓

Database
```

Only Driver changes.

---

# 4. JDBC Architecture

```
                 Java Application
                        │
                        │
                  JDBC API
                        │
             DriverManager Class
                        │
                 JDBC Driver
                        │
                  Database Server
                        │
                    Database
```

---

Let's understand each layer.

---

## Java Application

This is your Java Code.

Example

```
Main.java
Employee.java
Student.java
```

It contains

```
Connection

Statement

PreparedStatement

ResultSet
```

---

## JDBC API

A collection of Interfaces and Classes.

Examples

```
Connection

Statement

PreparedStatement

CallableStatement

ResultSet

DriverManager
```

Provided by

```
java.sql package
```

---

## DriverManager

DriverManager manages JDBC Drivers.

When Java asks

```
Connect to MySQL
```

DriverManager finds the correct Driver.

Then creates Connection.

---

## JDBC Driver

Most Important Layer.

It converts Java Requests into Database Specific Commands.

Example

Java says

```
Insert Student
```

Driver converts into

```
MySQL Compatible Protocol
```

Database understands it.

---

## Database

Stores Data.

Example

```
Student Table

Employee Table

Orders

Products
```

---

# 5. What is JDBC Driver?

Definition

A JDBC Driver is a software component that enables Java applications to communicate with a specific database.

It acts as a bridge between Java and Database.

Without Driver

Java cannot understand

MySQL

Oracle

SQL Server

PostgreSQL

---

# Driver Flow

```
Java

↓

JDBC API

↓

MySQL Driver

↓

MySQL Database
```

---

# Different Drivers

MySQL

```
mysql-connector-j
```

Oracle

```
ojdbc
```

SQL Server

```
mssql-jdbc
```

PostgreSQL

```
postgresql
```

---

# 6. JDBC Driver Types

There are four types of JDBC drivers.

## Type 1: JDBC-ODBC Bridge Driver

```
Java
 ↓
JDBC
 ↓
ODBC
 ↓
Database
```

* Oldest driver.
* Needed ODBC installed on the client machine.
* Platform dependent.
* Removed from Java 8 onward.

**Advantages**

* Easy to use initially.
* No vendor-specific driver required.

**Disadvantages**

* Slow because of extra conversion layer.
* Requires ODBC configuration.
* Not suitable for production.

---

## Type 2: Native-API Driver

```
Java
 ↓
JDBC
 ↓
Native Database Library
 ↓
Database
```

* Uses database vendor's native libraries (DLLs or shared libraries).
* Faster than Type 1.
* Platform dependent.

**Advantages**

* Better performance than Type 1.

**Disadvantages**

* Native libraries must be installed.
* Not portable.

---

## Type 3: Network Protocol Driver

```
Java
 ↓
JDBC
 ↓
Middleware Server
 ↓
Database
```

* Sends requests to a middleware server, which communicates with the database.
* Can work with multiple databases.

**Advantages**

* Database-independent client.
* Centralized management.

**Disadvantages**

* Requires a middleware server.
* Additional network overhead.

---

## Type 4: Thin Driver (Pure Java Driver)

```
Java
 ↓
JDBC
 ↓
Database
```

* Written entirely in Java.
* Communicates directly with the database using its native protocol.
* No native libraries required.

**Advantages**

* Fastest and most widely used.
* Platform independent.
* Easy to deploy.

**Disadvantages**

* Separate driver required for each database.

> **MySQL Connector/J is a Type 4 JDBC Driver.**

---

# 7. Prerequisites

Before learning JDBC, students should know:

## Java

Must Know

Basic Java

Loops

Methods

Classes

Objects

Exception Handling

Packages

Interfaces 

Polymorphism

Collections (Helpful)

---

## SQL

Must Know

```
CREATE DATABASE

CREATE TABLE

INSERT

UPDATE

DELETE

SELECT

WHERE

ORDER BY

Basic Queries 
```

Without SQL

JDBC is impossible.

---

# 8. Required Software

## Java JDK

Required to compile Java code.

---

## IntelliJ IDEA

Why IntelliJ?

Professional IDE

Auto Completion

Error Highlighting

Database Support

Maven Support

Gradle Support

Git Integration

Easy Library Management

---

## MySQL

Stores Data.

We will create

Database

↓

Tables

↓

Rows

↓

Columns

---

## MySQL Workbench (Optional but Recommended)

GUI tool to:

* Create databases
* Write SQL queries
* Design tables
* View data

---

# 9. What is MySQL Connector/J?

It is the official JDBC driver for MySQL.

It enables Java applications to connect with MySQL databases.

Connector/J converts Java JDBC calls into the MySQL protocol.

---

Without Connector/J

```
Java

Cannot

Connect

to

MySQL
```

---

With Connector/J

```
Java

↓

Connector/J

↓

MySQL
```

---

The driver class used in modern applications is:

```java
com.mysql.cj.jdbc.Driver
```

Although modern JDBC automatically loads the driver, understanding this class is important for learning and compatibility with older code.

---

# 10. JDBC Version History

| JDBC Version | Java Version | Major Features                                                                |
| ------------ | ------------ | ----------------------------------------------------------------------------- |
| JDBC 1.0     | JDK 1.1      | Initial JDBC API                                                              |
| JDBC 2.0     | JDK 1.2      | Scrollable ResultSet, Batch Updates, Transactions                             |
| JDBC 3.0     | JDK 1.4      | Savepoints, Generated Keys, Connection Pooling improvements                   |
| JDBC 4.0     | Java 6       | Automatic Driver Loading, SQLXML, Enhanced Exceptions                         |
| JDBC 4.1     | Java 7       | Try-with-Resources support, `getObject(Class<T>)`                             |
| JDBC 4.2     | Java 8       | Java Time API (`LocalDate`, `LocalTime`, `LocalDateTime`), REF_CURSOR support |
| JDBC 4.3     | Java 9       | Small API enhancements, module system compatibility                           |

---

# Difference Between JDBC Versions

### JDBC 1.0

* First JDBC release.
* Basic database connectivity.

---

### JDBC 2.0

Added:

* Batch Processing
* Scrollable ResultSet
* Updatable ResultSet
* Transactions

---

### JDBC 3.0

Added:

* Savepoints
* Auto-generated Keys
* Better Connection Pooling support

---

### JDBC 4.0

One of the biggest improvements.

Before Java 6:

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

was required to load the driver.

From JDBC 4.0 onward, the driver is discovered automatically if the JDBC driver JAR is on the classpath, so explicit loading is usually unnecessary.

Also added:

* SQLXML support
* Better exception hierarchy

---

### JDBC 4.1

Added support for:

* Try-with-Resources
* Generic `getObject()` methods

Example:

```java
LocalDate date = resultSet.getObject("dob", LocalDate.class);
```

---

### JDBC 4.2

Added support for Java 8 Date and Time API.

Example:

```java
preparedStatement.setObject(1, LocalDate.now());
```

---

### JDBC 4.3

Supports the Java Module System (introduced in Java 9) and includes small API refinements and performance-related improvements.

---

# Complete JDBC Flow

```
Java Program
      │
      ▼
JDBC API
      │
      ▼
DriverManager
      │
      ▼
MySQL Connector/J (Driver)
      │
      ▼
MySQL Server
      │
      ▼
Database
      │
      ▼
Table
```
