# JDBC Date and Time Operations

---

# Why Do We Need Date in Database?

Imagine you're building:

* Student Management System
* Hospital Management
* Banking System
* E-commerce Website
* Employee Management

Every application stores dates.

Examples:

Student

| ID  | Name  | Date of Birth |
| --- | ----- | ------------- |
| 101 | John | 2002-06-15    |

Employee

| ID | Name | Joining Date |
| -- | ---- | ------------ |
| 1  | John | 2024-01-15   |

Amazon Order

| Order ID | Order Date |
| -------- | ---------- |
| 5001     | 2025-08-06 |

Without Date datatype, we would have to store

```
"15 January 2024"
```

as a String.

That creates problems.

* Cannot compare dates
* Cannot sort correctly
* Cannot calculate age
* Cannot find today's orders

That's why databases provide dedicated Date data types.

---

# MySQL Date Types

There are four major types.

## 1. DATE

Stores only date.

Example

```
2025-08-06
```

Contains

* Year
* Month
* Day

Does NOT contain

* Hour
* Minute
* Second

---

## 2. TIME

Stores only time.

Example

```
14:30:45
```

Contains

* Hour
* Minute
* Second

Does NOT contain

* Date

---

## 3. DATETIME

Stores both.

Example

```
2025-08-06 14:30:45
```

Contains

* Date
* Time

---

## 4. TIMESTAMP

Looks similar to DATETIME.

Example

```
2025-08-06 14:30:45
```

Difference:

TIMESTAMP automatically stores the current timestamp if configured.

Useful for:

* Created At
* Updated At
* Login Time

---

# Java Classes Used in JDBC

JDBC provides matching classes.

| Database Type | Java Class         |
| ------------- | ------------------ |
| DATE          | java.sql.Date      |
| TIME          | java.sql.Time      |
| TIMESTAMP     | java.sql.Timestamp |

Notice these belong to

```
java.sql
```

NOT

```
java.util
```

---

# Create Database

```sql
CREATE DATABASE mydb;
```

---

# Create Employee Table

```sql
CREATE TABLE employee(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    dob DATE
);
```

---

# Project Structure

```
Main.java
```

---

# Step 1: JDBC Connection

```java
private static final String url =
"jdbc:mysql://localhost:3306/mydb";

private static final String username = "root";
private static final String password = "password";
```

---

# Insert Date into Database

## Full Code

```java
import java.sql.*;

public class Main {

    private static final String url =
            "jdbc:mysql://localhost:3306/mydb";

    private static final String username = "root";
    private static final String password = "password";

    public static void main(String[] args) {

        String query =
                "INSERT INTO employee(name,dob) VALUES(?,?)";

        try {

            Connection connection =
                    DriverManager.getConnection(url, username, password);

            PreparedStatement preparedStatement =
                    connection.prepareStatement(query);

            preparedStatement.setString(1, "John");

            Date date = Date.valueOf("2002-06-15");

            preparedStatement.setDate(2, date);

            int rows = preparedStatement.executeUpdate();

            System.out.println(rows + " row inserted.");

            preparedStatement.close();
            connection.close();

        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```

---

# Understanding Every Important Line

## Creating SQL Date

```java
Date date = Date.valueOf("2002-06-15");
```

Question:

Why not

```java
new Date();
```

Because

```
java.sql.Date
```

expects SQL-compatible format.

The easiest way is

```java
Date.valueOf()
```

Input format must always be

```
yyyy-MM-dd
```

Example

```
2025-01-10
```

---

## Setting Date

```java
preparedStatement.setDate(2,date);
```

Meaning

Put this Date object into

Second

```
?
```

---

Database finally receives

```
INSERT INTO employee(name,dob)

VALUES

('John','2002-06-15')
```

---

# Verify

```sql
SELECT * FROM employee;
```

Output

```
+----+-------+------------+
|id  |name   |dob         |
+----+-------+------------+
|1   |John  |2002-06-15  |
+----+-------+------------+
```

---

# Reading Date from Database

```java
import java.sql.*;

public class Main {

    private static final String url =
            "jdbc:mysql://localhost:3306/mydb";

    private static final String username = "root";
    private static final String password = "password";

    public static void main(String[] args) {

        String query = "SELECT * FROM employee";

        try {

            Connection connection =
                    DriverManager.getConnection(url, username, password);

            Statement statement =
                    connection.createStatement();

            ResultSet resultSet =
                    statement.executeQuery(query);

            while(resultSet.next()){

                int id = resultSet.getInt("id");

                String name =
                        resultSet.getString("name");

                Date dob =
                        resultSet.getDate("dob");

                System.out.println(id);

                System.out.println(name);

                System.out.println(dob);

                System.out.println("----------------");

            }

            resultSet.close();
            statement.close();
            connection.close();

        } catch (Exception e){
            e.printStackTrace();
        }

    }
}
```

---

# What does getDate() return?

```java
Date dob = resultSet.getDate("dob");
```

Returns

```
java.sql.Date
```

Example output

```
2002-06-15
```

---

# Taking Date from User

```java
Scanner scanner = new Scanner(System.in);

System.out.print("Enter Name : ");
String name = scanner.nextLine();

System.out.print("Enter DOB (yyyy-MM-dd): ");
String input = scanner.nextLine();

Date dob = Date.valueOf(input);
```

Then

```java
preparedStatement.setString(1,name);
preparedStatement.setDate(2,dob);
```

---

# Working with TIME

Now suppose employees have login time.

---

## Create Table

```sql
CREATE TABLE employee_time(

id INT PRIMARY KEY AUTO_INCREMENT,

name VARCHAR(100),

login_time TIME

);
```

---

# Java Class

```
java.sql.Time
```

---

# Insert Time

```java
String query =
"INSERT INTO employee_time(name,login_time) VALUES(?,?)";

Connection connection =
DriverManager.getConnection(url,username,password);

PreparedStatement preparedStatement =
connection.prepareStatement(query);

preparedStatement.setString(1,"John");

Time time = Time.valueOf("09:30:15");

preparedStatement.setTime(2,time);

preparedStatement.executeUpdate();
```

---

## Time Format

Must be

```
HH:mm:ss
```

Example

```
08:45:30

12:10:15

23:59:59
```

---

# Reading Time

```java
ResultSet resultSet =
statement.executeQuery("SELECT * FROM employee_time");

while(resultSet.next()){

Time time =
resultSet.getTime("login_time");

System.out.println(time);

}
```

Output

```
09:30:15
```

---

# Working with TIMESTAMP

Suppose we want to store

```
Order Created Time
```

Need both

Date

*

Time

---

## Table

```sql
CREATE TABLE orders(

id INT PRIMARY KEY AUTO_INCREMENT,

customer_name VARCHAR(100),

created_at TIMESTAMP

);
```

---

# Java Class

```
java.sql.Timestamp
```

---

# Insert Timestamp

```java
String query =
"INSERT INTO orders(customer_name,created_at) VALUES(?,?)";

PreparedStatement preparedStatement =
connection.prepareStatement(query);

preparedStatement.setString(1,"John");

Timestamp timestamp =
Timestamp.valueOf("2025-08-06 10:45:30");

preparedStatement.setTimestamp(2,timestamp);

preparedStatement.executeUpdate();
```

---

# Timestamp Format

```
yyyy-MM-dd HH:mm:ss
```

Example

```
2025-08-06 14:25:30
```

---

# Reading Timestamp

```java
ResultSet resultSet =
statement.executeQuery("SELECT * FROM orders");

while(resultSet.next()){

Timestamp timestamp =
resultSet.getTimestamp("created_at");

System.out.println(timestamp);

}
```

Output

```
2025-08-06 10:45:30.0
```

---

# Automatically Store Current Timestamp

Instead of sending timestamp from Java, let MySQL generate it.

```sql
CREATE TABLE orders(

id INT PRIMARY KEY AUTO_INCREMENT,

customer_name VARCHAR(100),

created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);
```

Now Java becomes

```java
String query =
"INSERT INTO orders(customer_name) VALUES(?)";

PreparedStatement preparedStatement =
connection.prepareStatement(query);

preparedStatement.setString(1,"John");

preparedStatement.executeUpdate();
```

Database automatically stores

```
Current Date

+

Current Time
```

---

# Java 8 Date-Time API (Recommended)

From Java 8 onward, use the `java.time` package in your application code.

| Modern Java Class | Represents    | JDBC Support                  |
| ----------------- | ------------- | ----------------------------- |
| `LocalDate`       | Date only     | `setObject()` / `getObject()` |
| `LocalTime`       | Time only     | `setObject()` / `getObject()` |
| `LocalDateTime`   | Date and Time | `setObject()` / `getObject()` |

Example:

```java
LocalDate dob = LocalDate.of(2002, 6, 15);

preparedStatement.setObject(2, dob);
```

Reading:

```java
LocalDate dob = resultSet.getObject("dob", LocalDate.class);

System.out.println(dob);
```

Similarly:

```java
LocalTime loginTime = LocalTime.of(9, 30, 15);
preparedStatement.setObject(2, loginTime);
```

```java
LocalDateTime createdAt = LocalDateTime.now();
preparedStatement.setObject(2, createdAt);
```

Using the `java.time` API is generally preferred in new Java applications because it is immutable, easier to use, and less error-prone than the older `java.sql.Date`, `Time`, and `Timestamp` classes.

---

# Common Mistakes

### 1. Wrong Date Format

❌

```java
Date.valueOf("15-06-2002");
```

✔

```java
Date.valueOf("2002-06-15");
```

---

### 2. Using `java.util.Date`

❌

```java
java.util.Date date = new java.util.Date();
preparedStatement.setDate(2, date); // Compilation error
```

✔

```java
java.sql.Date date = java.sql.Date.valueOf("2002-06-15");
preparedStatement.setDate(2, date);
```

---

### 3. Wrong Time Format

❌

```java
Time.valueOf("9:30");
```

✔

```java
Time.valueOf("09:30:00");
```

---

### 4. Mixing Date and Timestamp

* Use `setDate()` only for `DATE` columns.
* Use `setTime()` only for `TIME` columns.
* Use `setTimestamp()` or `setObject(LocalDateTime)` for `TIMESTAMP`/`DATETIME` columns.

---

# JDBC Lecture: Working with LOBs (Large Objects)

## What are LOBs?

**LOB** stands for **Large Object**.

A LOB is a special database data type used to store **very large amounts of data** that cannot fit into normal columns like `VARCHAR` or `INT`.

For example:

* A student's profile picture
* A PDF resume
* A movie
* A song
* A large text document
* A research paper

Instead of storing small values, LOBs are designed to store **MBs or even GBs of data**.

---

# Why Do We Need LOBs?

Imagine you're building a Student Management System.

Each student has:

* Name
* Email
* Photo
* Resume
* Certificate

The table might look like:

| ID  | Name  | Photo | Resume |
| --- | ----- | ----- | ------ |
| 101 | John | Image | PDF    |

Can we store an image inside a `VARCHAR(255)`?

**No.**

Images and files are much larger than 255 characters.

That's why databases provide LOB data types.

---

# Types of LOBs

There are two main types.

## 1. BLOB (Binary Large Object)

Stores **binary data**.

Examples:

* Images
* Videos
* Audio
* PDF
* ZIP files
* Word documents

Example:

```
student_photo.jpg
```

```
resume.pdf
```

```
song.mp3
```

---

## 2. CLOB (Character Large Object)

Stores **large text data**.

Examples:

* Books
* Articles
* Research Papers
* HTML files
* XML
* JSON
* Logs

Example

```
500-page book
```

```
Large blog article
```

---

# Difference Between BLOB and CLOB

| BLOB               | CLOB             |
| ------------------ | ---------------- |
| Stores binary data | Stores text data |
| Images             | Books            |
| Videos             | Articles         |
| Audio              | XML              |
| PDF                | JSON             |
| Byte based         | Character based  |

---

# MySQL LOB Types

## BLOB Family

| Type       | Maximum Size |
| ---------- | ------------ |
| TINYBLOB   | 255 Bytes    |
| BLOB       | 65 KB        |
| MEDIUMBLOB | 16 MB        |
| LONGBLOB   | 4 GB         |

---

## TEXT Family (CLOB Equivalent)

MySQL doesn't have a datatype literally named **CLOB**. Instead, it uses the **TEXT** family.

| Type       | Maximum Size   |
| ---------- | -------------- |
| TINYTEXT   | 255 Characters |
| TEXT       | 65 KB          |
| MEDIUMTEXT | 16 MB          |
| LONGTEXT   | 4 GB           |

---

# Java Classes Used

For BLOB

```java
InputStream
```

or

```java
FileInputStream
```

For CLOB

```java
Reader
```

or

```java
FileReader
```

---

# Example Project

Suppose we have Employee records.

Each employee has

* ID
* Name
* Photo
* Resume

---

# Create Table

```sql
CREATE TABLE employee(

id INT PRIMARY KEY AUTO_INCREMENT,

name VARCHAR(100),

photo LONGBLOB,

resume LONGTEXT

);
```

---

# Project Structure

```
Project

│

├── Main.java

├── photo.jpg

└── resume.txt
```

---

# Part 1 : Insert Image (BLOB)

## Step 1

Create File object

```java
File file = new File("photo.jpg");
```

---

## Step 2

Open FileInputStream

```java
FileInputStream inputStream =
new FileInputStream(file);
```

Think of it like opening a water pipe.

The bytes start flowing from

```
photo.jpg
```

into Java.

---

## Complete Program

```java
import java.io.File;
import java.io.FileInputStream;
import java.sql.*;

public class Main {

    private static final String url =
            "jdbc:mysql://localhost:3306/mydb";

    private static final String username = "root";

    private static final String password = "password";

    public static void main(String[] args) {

        String query =
                "INSERT INTO employee(name,photo) VALUES(?,?)";

        try {

            Connection connection =
                    DriverManager.getConnection(url, username, password);

            PreparedStatement preparedStatement =
                    connection.prepareStatement(query);

            preparedStatement.setString(1, "John");

            File file =
                    new File("photo.jpg");

            FileInputStream inputStream =
                    new FileInputStream(file);

            preparedStatement.setBinaryStream(2,
                    inputStream,
                    (int) file.length());

            int rows =
                    preparedStatement.executeUpdate();

            System.out.println(rows + " row inserted.");

            inputStream.close();
            preparedStatement.close();
            connection.close();

        } catch (Exception e) {

            e.printStackTrace();

        }

    }

}
```

---

# Understanding setBinaryStream()

```java
preparedStatement.setBinaryStream(
2,
inputStream,
(int) file.length());
```

### Parameter 1

```java
2
```

Means

Second `?`

---

### Parameter 2

```java
inputStream
```

JDBC reads bytes from here.

---

### Parameter 3

```java
file.length()
```

Number of bytes to send.

Example

If image size is

```
450 KB
```

JDBC sends

```
460800 bytes
```

---

# What Happens Internally?

```
photo.jpg

↓

FileInputStream

↓

PreparedStatement

↓

JDBC Driver

↓

MySQL

↓

LONGBLOB column
```

---

# Reading Image from Database

Suppose we want to retrieve the image and save it back to disk.

---

## Complete Program

```java
import java.io.FileOutputStream;
import java.io.InputStream;
import java.sql.*;

public class Main {

    private static final String url =
            "jdbc:mysql://localhost:3306/mydb";

    private static final String username = "root";

    private static final String password = "password";

    public static void main(String[] args) {

        String query =
                "SELECT photo FROM employee WHERE id=?";

        try {

            Connection connection =
                    DriverManager.getConnection(url, username, password);

            PreparedStatement preparedStatement =
                    connection.prepareStatement(query);

            preparedStatement.setInt(1, 1);

            ResultSet resultSet =
                    preparedStatement.executeQuery();

            if(resultSet.next()){

                InputStream inputStream =
                        resultSet.getBinaryStream("photo");

                FileOutputStream outputStream =
                        new FileOutputStream("downloaded_photo.jpg");

                byte[] buffer = new byte[4096];

                int bytesRead;

                while((bytesRead =
                        inputStream.read(buffer)) != -1){

                    outputStream.write(buffer,0,bytesRead);

                }

                inputStream.close();

                outputStream.close();

            }

            resultSet.close();

            preparedStatement.close();

            connection.close();

        } catch(Exception e){

            e.printStackTrace();

        }

    }

}
```

---

# Flow

```
Database

↓

InputStream

↓

Java

↓

FileOutputStream

↓

downloaded_photo.jpg
```

---

# Part 2 : Insert Large Text (CLOB)

Suppose

```
resume.txt
```

contains

```
Skills

Java

Spring

SQL

React

Docker

AWS
```

---

## Complete Program

```java
import java.io.File;
import java.io.FileReader;
import java.sql.*;

public class Main {

    private static final String url =
            "jdbc:mysql://localhost:3306/mydb";

    private static final String username = "root";

    private static final String password = "password";

    public static void main(String[] args) {

        String query =
                "INSERT INTO employee(name,resume) VALUES(?,?)";

        try {

            Connection connection =
                    DriverManager.getConnection(url, username, password);

            PreparedStatement preparedStatement =
                    connection.prepareStatement(query);

            preparedStatement.setString(1,"John");

            File file =
                    new File("resume.txt");

            FileReader reader =
                    new FileReader(file);

            preparedStatement.setCharacterStream(
                    2,
                    reader,
                    (int) file.length());

            preparedStatement.executeUpdate();

            reader.close();

            preparedStatement.close();

            connection.close();

        } catch(Exception e){

            e.printStackTrace();

        }

    }

}
```

---

# Reading CLOB

```java
PreparedStatement preparedStatement =
connection.prepareStatement(
"SELECT resume FROM employee WHERE id=?");

preparedStatement.setInt(1,1);

ResultSet resultSet =
preparedStatement.executeQuery();

if(resultSet.next()){

Reader reader =
resultSet.getCharacterStream("resume");

char[] buffer =
new char[1024];

int charsRead;

while((charsRead=
reader.read(buffer))!=-1){

System.out.print(
new String(buffer,0,charsRead));

}

reader.close();

}
```

---

# Internal Flow

```
resume.txt

↓

FileReader

↓

PreparedStatement

↓

Database

↓

LONGTEXT
```

---

# Real-World Examples

| Application | BLOB            | CLOB                |
| ----------- | --------------- | ------------------- |
| Facebook    | Profile Picture | Posts               |
| WhatsApp    | Images          | Messages            |
| YouTube     | Videos          | Descriptions        |
| Hospital    | X-Ray Image     | Doctor Notes        |
| College ERP | Student Photo   | Project Report      |
| Amazon      | Product Images  | Product Description |

---

# Important JDBC Methods

| Method                 | Purpose          |
| ---------------------- | ---------------- |
| `setBinaryStream()`    | Insert BLOB      |
| `getBinaryStream()`    | Read BLOB        |
| `setCharacterStream()` | Insert CLOB/TEXT |
| `getCharacterStream()` | Read CLOB/TEXT   |

---

# Common Mistakes

### 1. Using `setString()` for Images

❌

```java
preparedStatement.setString(2, "photo.jpg");
```

This stores only the file name, **not** the image.

✔

```java
preparedStatement.setBinaryStream(2, inputStream, (int) file.length());
```

---

### 2. Forgetting to Close Streams

Always close:

```java
inputStream.close();

outputStream.close();

reader.close();
```

---

### 3. Wrong File Path

❌

```java
File file = new File("abc.jpg");
```

If the file doesn't exist:

```
FileNotFoundException
```

---

### 4. Confusing BLOB and CLOB

* **BLOB** → Binary files (images, videos, PDFs)
* **CLOB/TEXT** → Large text (articles, resumes, JSON, XML)

---
