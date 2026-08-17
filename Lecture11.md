# Hibernate Class Notes 

---

> Imagine you're working in a company and your manager says:

> "Create a student management system."

You already know JDBC, so what will you do?

You'll probably write code like this:

```java
Connection con =
DriverManager.getConnection(
    url,
    username,
    password
);

String sql =
"INSERT INTO student VALUES (?, ?, ?)";

PreparedStatement ps =
con.prepareStatement(sql);

ps.setInt(1, 1);
ps.setString(2, "John");
ps.setInt(3, 22);

ps.executeUpdate();

ps.close();

con.close();
```

Ask students:

**How many things did we do?**

1. Open a connection.
2. Write an SQL query.
3. Create a PreparedStatement.
4. Set values.
5. Execute the query.
6. Close the statement.
7. Close the connection.

Then ask:

**What if we have 100 tables?**

Students will immediately understand the problem.

---

# Problems With JDBC

## Problem 1: Too Much SQL

JDBC requires writing SQL everywhere.

```sql
INSERT INTO student VALUES(1,'John',22);

UPDATE student
SET age=23
WHERE id=1;

DELETE FROM student
WHERE id=1;

SELECT * FROM student;
```

---

## Problem 2: Manual Connection Handling

You must create a connection every time.

```java
Connection con =
DriverManager.getConnection(
    url,
    username,
    password
);
```

---

## Problem 3: Too Much Boilerplate Code

Boilerplate code means code that is repeated again and again.

Example:

```java
PreparedStatement ps =
con.prepareStatement(sql);

ResultSet rs =
ps.executeQuery();

while(rs.next())
{
    Student s =
    new Student();

    s.setId(rs.getInt("id"));

    s.setName(rs.getString("name"));

    s.setAge(rs.getInt("age"));
}
```

This code becomes repetitive.

---

## Problem 4: Converting Database Data Into Objects

Suppose the database returns:

| id | name | age |
| -- | ---- | --- |
| 1  | John | 22  |

Java doesn't automatically understand database rows.

You must manually create an object.

```java
Student s =
new Student();

s.setId(1);

s.setName("John");

s.setAge(22);
```

This process is called **mapping**.

---

# Step 3: So Why Was Hibernate Created?

Hibernate was created to solve all these problems.

Instead of writing SQL,

we work with Java objects.

Hibernate automatically converts objects into database records.

---

# Step 4: What Is Hibernate?

## Definition

Hibernate is an **ORM framework**.

Write this:

```text
Hibernate = ORM Framework
```

---

# Step 5: What Is a Framework?

**What is JDBC?**

Answer:

JDBC is an API.

---

**What is Hibernate?**

Answer:

Hibernate is a framework.

---

## API

An API provides tools.

Example:

```java
Connection

PreparedStatement

ResultSet
```

You decide how to use them.

---

## Framework

A framework controls the application's structure.

It provides many features automatically.

---

# Step 6: What Is ORM?

ORM stands for:

```text
O = Object

R = Relational

M = Mapping
```

---

## What Is an Object?

```java
Student student =
new Student();
```

This is a Java object.

---

## What Is Relational?

A relational database stores data in tables.

```text
Student Table

+----+------+-----+

| id | name | age |

+----+------+-----+

| 1  | John | 22  |

+----+------+-----+
```

---

## What Is Mapping?

Mapping means connecting both worlds.

```text
Java Object
      ↓

Hibernate

      ↓

Database Table
```

---

# Visual Diagram

```text
Java Class

Student
{
    id
    name
    age
}

        ↓

Hibernate (ORM)

        ↓

Database Table

Student

+----+------+-----+

| id | name | age |

+----+------+-----+
```

---

# Step 7: How JDBC Works

```text
Application

↓

Java Code

↓

SQL Query

↓

JDBC

↓

Database
```

---

# Example

```java
Student s =
new Student(
    1,
    "John",
    22
);
```

Convert the object into SQL.

```sql
INSERT INTO student
VALUES(1,'John',22);
```

Who writes this SQL?

**The programmer.**

---

# Step 8: How Hibernate Works

```text
Application

↓

Java Object

↓

Hibernate

↓

SQL Generation

↓

Database
```

---

# Example

Create an object.

```java
Student s =
new Student(
    1,
    "John",
    22
);
```

Save it.

```java
session.save(s);
```

Hibernate automatically generates:

```sql
INSERT INTO student
VALUES(1,'John',22);
```

Who writes the SQL?

**Hibernate.**

---

# Step 9: JDBC vs Hibernate

| Feature           | JDBC   | Hibernate  |
| ----------------- | ------ | ---------- |
| SQL               | Manual | Automatic  |
| Code              | More   | Less       |
| Mapping           | Manual | Automatic  |
| Development Speed | Slow   | Fast       |
| Error Probability | High   | Lower      |
| CRUD              | Manual | Simplified |

---

# Step 10: Real-World Example

Imagine a food delivery app.

---

## JDBC

You cook the food yourself.

You buy vegetables.

You cut them.

You prepare them.

You cook them.

Everything is manual.

---

## Hibernate

You order food from an app.

You only choose the food.

Everything else happens automatically.

---

# Step 11: Important Hibernate Components

Tell students not to worry about these terms.

We'll study them one by one.

---

## 1. Configuration File

Contains database information.

```text
hibernate.cfg.xml
```

---

## 2. SessionFactory

Creates sessions.

```text
SessionFactory
```

---

## 3. Session

Communicates with the database.

```text
Session
```

---

## 4. Transaction

Ensures data consistency.

```text
Transaction
```

---

# Step 12: CRUD Comparison

## Create

### JDBC

```java
String sql =
"INSERT INTO student VALUES(?,?,?)";
```

### Hibernate

```java
session.save(student);
```

---

## Read

### JDBC

```java
SELECT * FROM student;
```

### Hibernate

```java
session.get(
    Student.class,
    1
);
```

---

## Update

### JDBC

```sql
UPDATE student
SET age=23
WHERE id=1;
```

### Hibernate

```java
session.update(student);
```

---

## Delete

### JDBC

```sql
DELETE FROM student
WHERE id=1;
```

### Hibernate

```java
session.delete(student);
```

# Create a Maven Project in IntelliJ

```text
File

↓

New

↓

Project

↓

Maven

↓

maven-archetype-quickstart
```

Project structure:

```text
MyProject

├── pom.xml

├── src

│   ├── main

│   │   └── java

│   │
│   └── test

│       └── java

└── target
```

---

# Add Dependencies

## MySQL Connector

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.4.0</version>
</dependency>
```

Purpose:

```text
Java ↔ MySQL
```

---

## Hibernate

```xml
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>7.1.0.Final</version>
</dependency>
```

Purpose:

```text
Java Objects ↔ SQL
```

---

## JPA

```xml
<dependency>
    <groupId>jakarta.persistence</groupId>
    <artifactId>jakarta.persistence-api</artifactId>
    <version>3.2.0</version>
</dependency>
```

Purpose:

```text
Provides annotations.
```

---

# What Is an Annotation?

An annotation is **metadata**.

Think of it as a sticky note.

Example:

```text
Approved

Urgent

Important
```

Annotations do the same thing.

They provide extra information.

Every annotation starts with:

```java
@
```

Examples:

```java
@Override

@Entity

@Table

@Id

@Column
```

---

# Create the Resources Folder

Right-click:

```text
src/main

↓

New

↓

Directory

↓

resources
```

---

# Create hibernate.cfg.xml

Inside the resources folder:

```text
resources

↓

hibernate.cfg.xml
```

Project structure:

```text
src

└── main

    ├── java

    └── resources

        └── hibernate.cfg.xml
```

---

# Configure Hibernate

```xml
<hibernate-configuration>

    <session-factory>

        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/mydb
        </property>

        <property name="hibernate.connection.username">
            root
        </property>

        <property name="hibernate.connection.password">
            root
        </property>

        <property name="hibernate.dialect">
            org.hibernate.dialect.MySQLDialect
        </property>

        <property name="hibernate.show_sql">
            true
        </property>

        <property name="hibernate.hbm2ddl.auto">
            update
        </property>

        <mapping class="model.Student"/>

    </session-factory>

</hibernate-configuration>
```

---

# Create the Model Package

```text
src/main/java

↓

New

↓

Package

↓

model
```

---

# What Is an Entity?

An entity is a Java class that represents a database table.

```text
Database Table → Java Class

students       → Student.java
```

---

# Create Student.java

```java
package model;

import jakarta.persistence.*;

@Entity

@Table(name = "students")

public class Student {

    @Id

    @GeneratedValue(strategy = GenerationType.IDENTITY)

    private int id;

    @Column(name = "name")

    private String name;

    @Column(name = "age")

    private int age;

    public Student() {

    }

    public Student(String name, int age) {

        this.name = name;

        this.age = age;
    }

    public int getId() {

        return id;
    }

    public void setId(int id) {

        this.id = id;
    }

    public String getName() {

        return name;
    }

    public void setName(String name) {

        this.name = name;
    }

    public int getAge() {

        return age;
    }

    public void setAge(int age) {

        this.age = age;
    }
}
```

---

# Understanding the Annotations

## @Entity

```java
@Entity
```

Meaning:

```text
This class represents a database table.
```

---

## @Table

```java
@Table(name = "students")
```

Meaning:

```text
Student.java → students table
```

---

## @Id

```java
@Id
```

Meaning:

```text
Primary Key
```

---

## @GeneratedValue

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

Meaning:

```text
MySQL automatically generates IDs.
```

---

## @Column

```java
@Column(name = "name")
```

Meaning:

```text
Java Field ↔ Database Column
```

---

# Why Do We Need Constructors?

The constructor creates objects.

```java
Student student = new Student();
```

Hibernate also creates objects automatically.

Therefore, we need:

```java
public Student() {

}
```

If the constructor is private:

```java
private Student() {

}
```

Hibernate cannot access it.

---

# Why Do We Need Getters and Setters?

Setter:

```java
student.setName("John");
```

Getter:

```java
student.getName();
```

They allow us to access private variables safely.

---

# Create App.java

```java
package com.hibernate;

import model.Student;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

public class App {

    public static void main(String[] args) {

        Configuration configuration =
                new Configuration().configure();

        SessionFactory sessionFactory =
                configuration.buildSessionFactory();

        Session session =
                sessionFactory.openSession();

        Transaction transaction =
                session.beginTransaction();

        Student student = new Student();

        student.setName("John");

        student.setAge(20);

        session.save(student);

        transaction.commit();

        session.close();

        sessionFactory.close();

        System.out.println(
                "Student saved successfully."
        );
    }
}
```

---

# What Happens Behind the Scenes?

```text
Student Object

↓

Hibernate

↓

SQL Query

↓

MySQL
```

---

# Hibernate Converts This:

```java
student.setName("John");

student.setAge(20);
```

Into This:

```sql
INSERT INTO students(age, name)

VALUES(20, 'John');
```

---

# Real Output

```text
Hibernate: insert into students (age, name)

values (?, ?)

Student saved successfully.
```

---

# Verify the Result

Run:

```sql
SELECT * FROM students;
```

Expected output:

```text
+----+-------+-----+

| id | name  | age |

+----+-------+-----+

| 1  | John   | 20  |

+----+-------+-----+
```

---

# Complete Hibernate Flow

```text
Core Java

↓

JDBC

↓

Maven

↓

Dependencies

↓

Resources Folder

↓

hibernate.cfg.xml

↓

Model Package

↓

Entity Class

↓

SessionFactory

↓

Session

↓

Transaction

↓

Create an Object

↓

Save the Object

↓

Commit

↓

Close the Session
```

