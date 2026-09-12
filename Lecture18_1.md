# Object-Relational Mapping — `@OneToOne`

### Java + Maven + Hibernate + MySQL

We will build a **complete working project from scratch**.

### What we are going to create

```text
Student
   |
   | @OneToOne
   ↓
StudentProfile
```

One student will have **one profile**, and one profile will belong to **one student**.

Example:

```text
Student
--------------------------------
id = 1
name = "Rahul"
email = "rahul@gmail.com"
          |
          ↓
StudentProfile
--------------------------------
id = 101
phone = "9876543210"
city = "Delhi"
```

> **Important:** We will NOT use `CascadeType` for now, because the goal is to understand the basic `@OneToOne` mapping first.

---

# 1. Prerequisites

You need:

* Java JDK
* IntelliJ IDEA / Eclipse / VS Code
* MySQL
* MySQL Workbench
* Maven

Check Java:

```bash
java -version
```

Check Maven:

```bash
mvn -version
```

---

# 2. Create Database

First open **MySQL Workbench**.

Run:

```sql
CREATE DATABASE onetoone_db;
```

Now select the database:

```sql
USE onetoone_db;
```

At this point, we have only created the **database**.

We will let Hibernate create the tables for us.

So currently:

```text
MySQL
  |
  └── onetoone_db
```

There are no tables yet.

---

# 3. Create Maven Project

Create a new Maven project.

Project structure:

```text
OneToOneMapping
│
├── pom.xml
│
└── src
    └── main
        ├── java
        └── resources
```

Our final project will look like:

```text
OneToOneMapping
│
├── pom.xml
│
└── src
    └── main
        ├── java
        │   └── com.example
        │       ├── Main.java
        │       ├── model
        │       │   ├── Student.java
        │       │   └── StudentProfile.java
        │       └── util
        │           └── HibernateUtil.java
        │
        └── resources
            └── hibernate.cfg.xml
```

---

# 4. `pom.xml`

The `pom.xml` file contains information about our Maven project and the libraries/dependencies required by our project.

Copy this entire file:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>OneToOneMapping</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>

        <!-- Hibernate ORM -->
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>6.6.1.Final</version>
        </dependency>

        <!-- MySQL JDBC Driver -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>9.0.0</version>
        </dependency>

        <!-- JPA API -->
        <dependency>
            <groupId>jakarta.persistence</groupId>
            <artifactId>jakarta.persistence-api</artifactId>
            <version>3.2.0</version>
        </dependency>

    </dependencies>

</project>
```

## What are these dependencies?

### Hibernate

```xml
<artifactId>hibernate-core</artifactId>
```

Hibernate is responsible for converting our Java objects into database records.

For example:

```java
Student student = new Student();
```

Hibernate can convert this Java object into a database row.

---

### MySQL Connector

```xml
<artifactId>mysql-connector-j</artifactId>
```

This allows Java/Hibernate to communicate with MySQL.

Think:

```text
Java
  ↓
Hibernate
  ↓
MySQL Connector
  ↓
MySQL
```

---

### Jakarta Persistence

```xml
<artifactId>jakarta.persistence-api</artifactId>
```

This provides annotations such as:

```java
@Entity
@Id
@OneToOne
@JoinColumn
```

These annotations tell Hibernate how our Java classes are related to database tables.

---

# 5. Create `Student` Model

Create:

```text
src/main/java/com/example/model/Student.java
```

Paste:

```java
package com.example.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.OneToOne;

@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;

    private String email;

    @OneToOne
    private StudentProfile studentProfile;

    public Student() {
    }

    public Student(String name, String email, StudentProfile studentProfile) {
        this.name = name;
        this.email = email;
        this.studentProfile = studentProfile;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public StudentProfile getStudentProfile() {
        return studentProfile;
    }

    public void setStudentProfile(StudentProfile studentProfile) {
        this.studentProfile = studentProfile;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}
```

---

# 6. Understand `Student.java`

Let's understand this from the beginning.

## Package

```java
package com.example.model;
```

This tells Java that our class belongs to the `com.example.model` package.

---

## Imports

```java
import jakarta.persistence.Entity;
```

We need `Entity` because we want Hibernate to treat `Student` as a database entity.

---

## `@Entity`

```java
@Entity
public class Student {
```

This is extremely important.

`@Entity` means:

> "Hibernate, treat this Java class as a database entity."

So:

```java
@Entity
public class Student
```

will result in a table similar to:

```text
student
```

The class represents the table.

The object represents a row.

Think:

```text
Java Class
    ↓
Database Table

Student
    ↓
student
```

---

# 7. Primary Key

```java
@Id
private int id;
```

`@Id` tells Hibernate:

> This field is the primary key.

So:

```text
id
```

becomes the primary key of the `student` table.

---

# 8. Automatically Generate ID

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

This tells Hibernate/MySQL to generate the ID automatically.

For example, if we create:

```java
Student student1 = new Student(...);
Student student2 = new Student(...);
```

MySQL can generate:

```text
student1 → id = 1
student2 → id = 2
```

We don't need to manually write:

```java
student.setId(1);
```

---

# 9. Normal Fields

```java
private String name;
private String email;
```

These become columns.

So our table will approximately look like:

```text
student
--------------------------------
id
name
email
studentProfile_id
```

The exact generated SQL/table naming can depend on Hibernate configuration/version.

---

# 10. The Important Part — `@OneToOne`

```java
@OneToOne
private StudentProfile studentProfile;
```

This is the actual relationship.

It means:

> One `Student` is associated with one `StudentProfile`.

For example:

```text
Student
Rahul
   |
   | one-to-one
   |
   ↓
StudentProfile
9876543210
```

---

# 11. Create `StudentProfile`

Create:

```text
src/main/java/com/example/model/StudentProfile.java
```

Paste:

```java
package com.example.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class StudentProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String phone;

    private String city;

    public StudentProfile() {
    }

    public StudentProfile(String phone, String city) {
        this.phone = phone;
        this.city = city;
    }

    public int getId() {
        return id;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    @Override
    public String toString() {
        return "StudentProfile{" +
                "id=" + id +
                ", phone='" + phone + '\'' +
                ", city='" + city + '\'' +
                '}';
    }
}
```

---

# 12. Understand `StudentProfile`

Again:

```java
@Entity
public class StudentProfile {
```

means Hibernate should create a table for this class.

The fields:

```java
private int id;
private String phone;
private String city;
```

will become columns.

Conceptually:

```text
student_profile
--------------------------
id
phone
city
```

---

# 13. Why Do We Have Two Classes?

Because we are modelling two different things.

### Student

```text
Student
----------------
id
name
email
```

### Student Profile

```text
StudentProfile
----------------
id
phone
city
```

And they are related:

```text
Student  --------  StudentProfile
   1                     1
```

That's why we use:

```java
@OneToOne
```

---

# 14. Create Hibernate Configuration

Create:

```text
src/main/resources/hibernate.cfg.xml
```

Paste:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "https://hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <!-- MySQL Database Connection -->
        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/onetoone_db
        </property>

        <property name="hibernate.connection.username">
            root
        </property>

        <property name="hibernate.connection.password">
            YOUR_MYSQL_PASSWORD
        </property>

        <!-- Hibernate Dialect -->
        <property name="hibernate.dialect">
            org.hibernate.dialect.MySQLDialect
        </property>

        <!-- Automatically create/update tables -->
        <property name="hibernate.hbm2ddl.auto">
            update
        </property>

        <!-- Show SQL queries in console -->
        <property name="hibernate.show_sql">
            true
        </property>

        <!-- Format SQL queries -->
        <property name="hibernate.format_sql">
            true
        </property>

        <!-- Register Entity Classes -->
        <mapping class="com.example.model.Student"/>
        <mapping class="com.example.model.StudentProfile"/>

    </session-factory>

</hibernate-configuration>
```

### IMPORTANT

Change:

```xml
YOUR_MYSQL_PASSWORD
```

to your actual MySQL password.

For example:

```xml
<property name="hibernate.connection.password">
    root123
</property>
```

Don't put the password on GitHub in a real project.

For this beginner practice project, you can use it locally.

---

# 15. Understand `hibernate.cfg.xml`

This file tells Hibernate:

> "How do I connect to my database and which Java classes should I manage?"

---

## MySQL Driver

```xml
<property name="hibernate.connection.driver_class">
    com.mysql.cj.jdbc.Driver
</property>
```

This tells Hibernate which JDBC driver to use.

---

## Database URL

```xml
<property name="hibernate.connection.url">
    jdbc:mysql://localhost:3306/onetoone_db
</property>
```

Break it down:

```text
jdbc:mysql
```

We are using JDBC with MySQL.

```text
localhost
```

MySQL is running on our own computer.

```text
3306
```

Default MySQL port.

```text
onetoone_db
```

Our database name.

So:

```text
jdbc:mysql://localhost:3306/onetoone_db
```

means:

> Connect Java to the MySQL database named `onetoone_db` running on localhost.

---

# 16. Username

```xml
<property name="hibernate.connection.username">
    root
</property>
```

This is your MySQL username.

Usually:

```text
root
```

for a local installation.

---

# 17. Password

```xml
<property name="hibernate.connection.password">
    YOUR_MYSQL_PASSWORD
</property>
```

This must match your MySQL password.

---

# 18. Hibernate Dialect

```xml
<property name="hibernate.dialect">
    org.hibernate.dialect.MySQLDialect
</property>
```

Hibernate supports different databases.

For example:

```text
MySQL
PostgreSQL
Oracle
SQL Server
```

Dialect tells Hibernate:

> "Generate SQL appropriate for MySQL."

---

# 19. `hbm2ddl.auto`

```xml
<property name="hibernate.hbm2ddl.auto">
    update
</property>
```

For our learning project, we use:

```text
update
```

Hibernate checks the entity classes and updates the database schema as necessary.

For example:

```java
@Entity
public class Student
```

can result in Hibernate creating the corresponding table.

---

# 20. Show SQL

```xml
<property name="hibernate.show_sql">
    true
</property>
```

This is useful for learning.

Hibernate will show SQL queries in the console.

For example:

```sql
insert into student ...
```

This lets us see what Hibernate is doing behind the scenes.

---

# 21. Register Our Entities

```xml
<mapping class="com.example.model.Student"/>
```

and:

```xml
<mapping class="com.example.model.StudentProfile"/>
```

This tells Hibernate:

> These two classes are entities that Hibernate should manage.

---

# 22. Create `HibernateUtil`

Now create:

```text
src/main/java/com/example/util/HibernateUtil.java
```

Paste:

```java
package com.example.util;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {

    private static final SessionFactory sessionFactory;

    static {
        try {

            sessionFactory = new Configuration()
                    .configure()
                    .buildSessionFactory();

        } catch (Throwable ex) {

            System.out.println("SessionFactory creation failed.");
            ex.printStackTrace();

            throw new ExceptionInInitializerError(ex);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
}
```

---

# 23. Why Do We Need `HibernateUtil`?

Our main program needs a Hibernate `SessionFactory`.

The `SessionFactory` is responsible for creating Hibernate sessions.

Think:

```text
Hibernate Configuration
        ↓
SessionFactory
        ↓
Session
        ↓
Database
```

---

# 24. Important Line

```java
new Configuration()
```

creates a Hibernate configuration object.

Then:

```java
.configure()
```

loads:

```text
hibernate.cfg.xml
```

Then:

```java
.buildSessionFactory()
```

creates the:

```text
SessionFactory
```

So this:

```java
sessionFactory = new Configuration()
        .configure()
        .buildSessionFactory();
```

basically means:

> Read my Hibernate configuration and create a SessionFactory.

---

# 25. Create Main Class

Create:

```text
src/main/java/com/example/Main.java
```

Paste:

```java
package com.example;

import com.example.model.Student;
import com.example.model.StudentProfile;
import com.example.util.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class Main {

    public static void main(String[] args) {

        SessionFactory sessionFactory =
                HibernateUtil.getSessionFactory();

        Session session = sessionFactory.openSession();

        Transaction transaction = null;

        try {

            transaction = session.beginTransaction();

            StudentProfile profile =
                    new StudentProfile(
                            "9876543210",
                            "Delhi"
                    );

            Student student =
                    new Student(
                            "Rahul",
                            "rahul@gmail.com",
                            profile
                    );

            session.persist(profile);

            session.persist(student);

            transaction.commit();

            System.out.println("Data inserted successfully.");

        } catch (Exception e) {

            if (transaction != null) {
                transaction.rollback();
            }

            e.printStackTrace();

        } finally {

            session.close();
            sessionFactory.close();
        }
    }
}
```

---

# 26. Understand `Main.java`

This is the most important part because this is where we actually save data.

---

## Step 1 — Get SessionFactory

```java
SessionFactory sessionFactory =
        HibernateUtil.getSessionFactory();
```

We get the `SessionFactory` that we created in:

```text
HibernateUtil.java
```

---

# 27. Open Session

```java
Session session = sessionFactory.openSession();
```

A Hibernate `Session` is used to communicate with the database.

Think of it as a working connection/context through which we perform operations such as:

```text
INSERT
UPDATE
DELETE
SELECT
```

So:

```text
SessionFactory
      ↓
   Session
      ↓
 Database
```

---

# 28. Create Transaction

```java
Transaction transaction = null;
```

We create a variable for the transaction.

Then:

```java
transaction = session.beginTransaction();
```

starts the transaction.

Why?

Because database operations should be performed inside a transaction.

---

# 29. Create StudentProfile Object

```java
StudentProfile profile =
        new StudentProfile(
                "9876543210",
                "Delhi"
        );
```

This creates a Java object.

At this moment:

**Nothing has been inserted into MySQL yet.**

We only have an object in Java memory:

```text
profile
   |
   ↓
phone = 9876543210
city = Delhi
```

---

# 30. Create Student Object

```java
Student student =
        new Student(
                "Rahul",
                "rahul@gmail.com",
                profile
        );
```

We create a Student object.

Notice this:

```java
profile
```

is passed into the Student constructor.

Therefore:

```text
Student
----------------------
name = Rahul
email = rahul@gmail.com
studentProfile
      |
      ↓
StudentProfile
----------------------
phone = 9876543210
city = Delhi
```

This is our Java-side relationship.

---

# 31. Save Profile

```java
session.persist(profile);
```

This tells Hibernate:

> Make this `StudentProfile` object persistent.

Hibernate will eventually execute an SQL `INSERT`.

Something conceptually similar to:

```sql
INSERT INTO student_profile
(phone, city)
VALUES
('9876543210', 'Delhi');
```

---

# 32. Save Student

```java
session.persist(student);
```

Now we tell Hibernate to save the Student.

Hibernate will generate the required SQL.

Because Student contains:

```java
@OneToOne
private StudentProfile studentProfile;
```

Hibernate also needs to store the relationship.

Conceptually, the Student table may contain something like:

```text
id
name
email
student_profile_id
```

The exact column/table naming can vary.

---

# 33. Why Are We Saving Profile First?

We deliberately do:

```java
session.persist(profile);
session.persist(student);
```

instead of using cascade.

This is important for our current learning.

Because we are **not using `CascadeType`**, we explicitly save both objects.

Remember:

```java
session.persist(profile);
```

and:

```java
session.persist(student);
```

are two separate persistence operations.

Later, when you learn cascading, you'll see how this can be simplified.

---

# 34. Commit

```java
transaction.commit();
```

This is extremely important.

It means:

> The transaction is successful. Apply the database changes.

So the flow is:

```text
beginTransaction()
       ↓
persist(profile)
       ↓
persist(student)
       ↓
commit()
       ↓
Database
```

---

# 35. What If Something Goes Wrong?

```java
catch (Exception e) {
```

If an error occurs:

```java
if (transaction != null) {
    transaction.rollback();
}
```

`rollback()` tells the database:

> Cancel the changes made in this transaction.

For example:

```text
begin
 ↓
insert profile
 ↓
insert student
 ↓
ERROR
 ↓
rollback
```

The transaction's changes are rolled back.

---

# 36. Close Session

```java
session.close();
```

We don't want to leave database resources open.

---

# 37. Close SessionFactory

```java
sessionFactory.close();
```

After the program is finished, we close the `SessionFactory`.

---

# 38. Complete Project

Your project should now look like:

```text
OneToOneMapping
│
├── pom.xml
│
└── src
    └── main
        │
        ├── java
        │   │
        │   └── com
        │       └── example
        │           │
        │           ├── Main.java
        │           │
        │           ├── model
        │           │   ├── Student.java
        │           │   └── StudentProfile.java
        │           │
        │           └── util
        │               └── HibernateUtil.java
        │
        └── resources
            └── hibernate.cfg.xml
```

---

# 39. Run the Application

Before running, make sure:

### MySQL is running

Then make sure your database exists:

```sql
CREATE DATABASE onetoone_db;
```

Then make sure your password in:

```text
hibernate.cfg.xml
```

is correct.

Now run:

```text
Main.java
```

---

# 40. What Will Hibernate Do?

When you run the application, Hibernate first reads:

```text
hibernate.cfg.xml
```

Then it connects to:

```text
onetoone_db
```

Then it sees:

```java
@Entity
public class Student
```

and:

```java
@Entity
public class StudentProfile
```

Hibernate creates/updates the required tables.

Then:

```java
session.persist(profile);
```

saves the profile.

Then:

```java
session.persist(student);
```

saves the student and its relationship.

Finally:

```java
transaction.commit();
```

commits everything.

---

# 41. Check MySQL

Open MySQL Workbench.

Run:

```sql
USE onetoone_db;
```

See tables:

```sql
SHOW TABLES;
```

You should see tables corresponding to:

```text
student
student_profile
```

Now:

```sql
SELECT * FROM student;
```

And:

```sql
SELECT * FROM student_profile;
```

You should see your inserted data.

---

# 42. Understanding the Database Relationship

Conceptually, your database will look like:

```text
student
------------------------------------------------
id | name  | email             | studentProfile_id
------------------------------------------------
1  | Rahul | rahul@gmail.com   | 1
```

And:

```text
student_profile
------------------------------------
id | phone      | city
------------------------------------
1  | 9876543210 | Delhi
```

So:

```text
Student ID = 1
       |
       | studentProfile_id = 1
       ↓
StudentProfile ID = 1
```

That's the database representation of the relationship.

---

# 43. The Complete Concept

Now look at the entire flow:

```text
                    JAVA
                     |
                     ↓
          ┌─────────────────────┐
          │      Student        │
          │---------------------│
          │ id                  │
          │ name                │
          │ email               │
          │ studentProfile      │
          └──────────┬──────────┘
                     │
                 @OneToOne
                     │
                     ↓
          ┌─────────────────────┐
          │  StudentProfile     │
          │---------------------│
          │ id                  │
          │ phone               │
          │ city                │
          └─────────────────────┘
                     |
                     ↓
                  Hibernate
                     |
                     ↓
                  JDBC
                     |
                     ↓
                   MySQL
```

This is the basic idea of **Object-Relational Mapping**.

---

# 44. What Does ORM Actually Mean?

ORM stands for:

> **Object Relational Mapping**

There are two worlds.

### Java world

We work with:

```java
Student student = new Student();
```

These are **objects**.

### Database world

We work with:

```text
student table
```

These are **relational tables**.

ORM connects these two worlds:

```text
Java Object
     ↕
   ORM
     ↕
Database Table
```

Hibernate is an ORM framework.

---

# 45. Without Hibernate

Without ORM, you would have to write JDBC code like:

```java
Connection connection = ...;

PreparedStatement statement =
        connection.prepareStatement(
                "INSERT INTO student(name, email) VALUES (?, ?)"
        );

statement.setString(1, "Rahul");
statement.setString(2, "rahul@gmail.com");

statement.executeUpdate();
```

With Hibernate, we can simply write:

```java
Student student =
        new Student(
                "Rahul",
                "rahul@gmail.com",
                profile
        );

session.persist(student);
```

Hibernate handles much of the SQL generation for us.

---

# 46. Most Important Annotations

For this first project, remember these:

### `@Entity`

```java
@Entity
```

Means:

> This class should be mapped to a database table.

---

### `@Id`

```java
@Id
```

Means:

> This field is the primary key.

---

### `@GeneratedValue`

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

Means:

> Generate the primary-key value automatically.

---

### `@OneToOne`

```java
@OneToOne
```

Means:

> One object is associated with one object of another entity.

---

# 47. One-to-One in Simple English

Imagine a person and passport.

```text
One Person → One Passport
```

Or:

```text
One Student → One StudentProfile
```

Or:

```text
One Employee → One EmployeeProfile
```

Or:

```text
One User → One UserAccount
```

All of these can represent a **One-to-One relationship**.

---

# 48. Why We Didn't Use `CascadeType`

You specifically wanted to learn the basic version first.

So we use:

```java
@OneToOne
private StudentProfile studentProfile;
```

and **not**:

```java
@OneToOne(cascade = CascadeType.ALL)
```

Therefore we explicitly save:

```java
session.persist(profile);
session.persist(student);
```

This makes it easier to understand what Hibernate is doing.

Later, when learning cascading, you'll understand why:

```java
cascade = CascadeType.ALL
```

can change the persistence behavior.

---

# 49. Final Copy-Paste Files

For quick GitHub setup, these are the files you need:

```text
OneToOneMapping/
│
├── pom.xml
│
└── src/main/
    │
    ├── java/com/example/
    │   ├── Main.java
    │   ├── model/
    │   │   ├── Student.java
    │   │   └── StudentProfile.java
    │   └── util/
    │       └── HibernateUtil.java
    │
    └── resources/
        └── hibernate.cfg.xml
```

The execution sequence is:

```text
1. Create MySQL database
          ↓
2. Create Maven project
          ↓
3. Add Hibernate + MySQL dependencies
          ↓
4. Create Student entity
          ↓
5. Create StudentProfile entity
          ↓
6. Configure Hibernate
          ↓
7. Create SessionFactory
          ↓
8. Open Session
          ↓
9. Begin Transaction
          ↓
10. Create Profile object
          ↓
11. Create Student object
          ↓
12. persist(profile)
          ↓
13. persist(student)
          ↓
14. commit()
          ↓
15. Check MySQL
```

**This is the basic `@OneToOne` implementation.** The next natural step after understanding this is **bidirectional `@OneToOne` using `mappedBy`**, where we make both `Student → StudentProfile` and `StudentProfile → Student` navigable.

# `@OneToOne` — Part 2: Bidirectional Mapping

Now we move one step ahead.

In the previous example, we had:

```text
Student → StudentProfile
```

The `Student` knew about its `StudentProfile`.

But the `StudentProfile` did **not** know which `Student` it belonged to.

Now we will make it **bidirectional**:

```text
Student ↔ StudentProfile
```

That means:

```text
Student
   ↓
StudentProfile

StudentProfile
   ↓
Student
```

---

## 1. Change `StudentProfile.java`

Replace your previous `StudentProfile.java` completely with this:

```java
package com.example.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.OneToOne;

@Entity
public class StudentProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String phone;

    private String city;

    @OneToOne(mappedBy = "studentProfile")
    private Student student;

    public StudentProfile() {
    }

    public StudentProfile(String phone, String city) {
        this.phone = phone;
        this.city = city;
    }

    public int getId() {
        return id;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public Student getStudent() {
        return student;
    }

    public void setStudent(Student student) {
        this.student = student;
    }

    @Override
    public String toString() {
        return "StudentProfile{" +
                "id=" + id +
                ", phone='" + phone + '\'' +
                ", city='" + city + '\'' +
                '}';
    }
}
```

---

# 2. The Important Change

Previously we had:

```java
@Entity
public class StudentProfile {

    private int id;
    private String phone;
    private String city;
}
```

Now we have:

```java
@OneToOne(mappedBy = "studentProfile")
private Student student;
```

This creates the reverse side of the relationship.

---

# 3. What Does `mappedBy` Mean?

This is one of the **most important concepts** in bidirectional mapping.

We already have this in `Student`:

```java
@OneToOne
private StudentProfile studentProfile;
```

And now:

```java
@OneToOne(mappedBy = "studentProfile")
private Student student;
```

The value:

```java
"studentProfile"
```

refers to the **field name inside the `Student` class**.

Look at `Student.java`:

```java
@OneToOne
private StudentProfile studentProfile;
```

The field name is:

```text
studentProfile
```

Therefore:

```java
mappedBy = "studentProfile"
```

means:

> "The relationship is already mapped by the `studentProfile` field inside the Student class."

---

# 4. Very Important — `mappedBy` Does NOT Mean Column Name

Beginners often make this mistake.

This:

```java
@OneToOne(mappedBy = "studentProfile")
```

does **not** mean:

```text
mappedBy = database column name
```

It means:

```text
mappedBy = Java field name
```

For example:

```java
private StudentProfile studentProfile;
```

Therefore:

```java
mappedBy = "studentProfile"
```

Correct.

---

# 5. Who Owns the Relationship?

In our example:

```java
Student
    |
    | @OneToOne
    ↓
StudentProfile
```

The `Student` side is the **owning side**.

Why?

Because `Student` contains:

```java
@OneToOne
private StudentProfile studentProfile;
```

Hibernate uses this side to manage the relationship.

The `StudentProfile` side has:

```java
@OneToOne(mappedBy = "studentProfile")
private Student student;
```

This is the **inverse side**.

---

# 6. Simple Way to Remember

Think:

```text
Student.java
@OneToOne
private StudentProfile studentProfile;
```

⬆️

This side **owns** the relationship.

And:

```text
StudentProfile.java
@OneToOne(mappedBy = "studentProfile")
private Student student;
```

⬆️

This side says:

> "I am mapped by the relationship that already exists in Student."

---

# 7. Update `Main.java`

Now replace your previous `Main.java` with:

```java
package com.example;

import com.example.model.Student;
import com.example.model.StudentProfile;
import com.example.util.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class Main {

    public static void main(String[] args) {

        SessionFactory sessionFactory =
                HibernateUtil.getSessionFactory();

        Session session =
                sessionFactory.openSession();

        Transaction transaction = null;

        try {

            transaction = session.beginTransaction();

            StudentProfile profile =
                    new StudentProfile(
                            "9876543210",
                            "Delhi"
                    );

            Student student =
                    new Student(
                            "Rahul",
                            "rahul@gmail.com",
                            profile
                    );

            profile.setStudent(student);

            session.persist(profile);

            session.persist(student);

            transaction.commit();

            System.out.println("Data inserted successfully.");

        } catch (Exception e) {

            if (transaction != null) {
                transaction.rollback();
            }

            e.printStackTrace();

        } finally {

            session.close();
            sessionFactory.close();
        }
    }
}
```

---

# 8. New Line in `Main.java`

The important new line is:

```java
profile.setStudent(student);
```

Why did we add this?

Because now our relationship exists in **both directions**.

We have:

```text
student → profile
```

because of:

```java
Student student =
        new Student(
                "Rahul",
                "rahul@gmail.com",
                profile
        );
```

And:

```text
profile → student
```

because of:

```java
profile.setStudent(student);
```

So our Java objects look like:

```text
             Student
          ┌─────────────┐
          │ Rahul       │
          │ rahul@...   │
          └──────┬──────┘
                 │
                 ↓
          StudentProfile
          ┌─────────────┐
          │ Delhi       │
          │ 9876543210  │
          └──────┬──────┘
                 │
                 ↓
              Student
```

Both objects know about each other.

---

# 9. Why Do We Need `profile.setStudent(student)`?

Suppose we only write:

```java
Student student =
        new Student(
                "Rahul",
                "rahul@gmail.com",
                profile
        );
```

Then:

```text
student.getStudentProfile()
```

works.

But:

```java
profile.getStudent()
```

will return:

```text
null
```

because we never told the profile about the student.

Therefore:

```java
profile.setStudent(student);
```

makes the Java object relationship consistent in both directions.

---

# 10. Very Important Concept

There are actually **two different things** to understand:

### Java object relationship

```java
student.setStudentProfile(profile);
profile.setStudent(student);
```

### Database relationship

Hibernate uses the **owning side** to manage the database relationship.

In our case:

```java
Student
```

is the owning side.

The `mappedBy` side is not used to create another foreign key.

---

# 11. Why Doesn't `StudentProfile` Create Another Foreign Key?

You might think:

```text
student
student_profile
```

should have:

```text
student_profile_id
student_id
```

But that's not necessary.

A one-to-one relationship normally needs **one foreign key** to connect the two tables.

Conceptually:

```text
student
--------------------------------
id
name
email
student_profile_id
```

and:

```text
student_profile
--------------------------------
id
phone
city
```

The relationship is:

```text
student.student_profile_id
             ↓
student_profile.id
```

---

# 12. What `mappedBy` Prevents

Without `mappedBy`, you could accidentally tell Hibernate:

> "Create another independent relationship from StudentProfile to Student."

That can lead to unnecessary relationship columns/join structures.

Using:

```java
mappedBy = "studentProfile"
```

tells Hibernate:

> "Don't create a separate relationship here. Use the relationship already defined in Student."

---

# 13. Now Let's Fetch the Data

The real benefit of bidirectional mapping is that we can navigate in either direction.

Let's create a new example.

Replace `Main.java` with:

```java
package com.example;

import com.example.model.Student;
import com.example.model.StudentProfile;
import com.example.util.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class Main {

    public static void main(String[] args) {

        SessionFactory sessionFactory =
                HibernateUtil.getSessionFactory();

        Session session =
                sessionFactory.openSession();

        Transaction transaction = null;

        try {

            transaction = session.beginTransaction();

            StudentProfile profile =
                    new StudentProfile(
                            "9876543210",
                            "Delhi"
                    );

            Student student =
                    new Student(
                            "Rahul",
                            "rahul@gmail.com",
                            profile
                    );

            profile.setStudent(student);

            session.persist(profile);
            session.persist(student);

            transaction.commit();

            System.out.println("Data inserted successfully.");

        } catch (Exception e) {

            if (transaction != null) {
                transaction.rollback();
            }

            e.printStackTrace();

        } finally {

            session.close();
            sessionFactory.close();
        }
    }
}
```

For now, this is enough for insertion.

---

# 14. Fetch Student

Let's understand fetching separately.

We can use:

```java
Student student = session.find(Student.class, 1);
```

This means:

> Find the Student whose primary key is `1`.

Then:

```java
System.out.println(student);
```

prints the student.

And:

```java
System.out.println(student.getStudentProfile());
```

gets the associated profile.

Conceptually:

```text
Student
   ↓
getStudentProfile()
   ↓
StudentProfile
```

---

# 15. Fetch Profile

Because we now have a bidirectional relationship, we can also do:

```java
StudentProfile profile =
        session.find(StudentProfile.class, 1);
```

Then:

```java
System.out.println(profile);
```

And:

```java
System.out.println(profile.getStudent());
```

Now:

```text
StudentProfile
      ↓
getStudent()
      ↓
Student
```

That's what **bidirectional** means.

---

# 16. Complete Fetch Example

If you already have data in your database, you can use:

```java
package com.example;

import com.example.model.Student;
import com.example.model.StudentProfile;
import com.example.util.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.SessionFactory;

public class Main {

    public static void main(String[] args) {

        SessionFactory sessionFactory =
                HibernateUtil.getSessionFactory();

        Session session =
                sessionFactory.openSession();

        Student student =
                session.find(Student.class, 1);

        System.out.println("Student:");
        System.out.println(student);

        System.out.println("Student Profile:");
        System.out.println(student.getStudentProfile());

        StudentProfile profile =
                session.find(StudentProfile.class, 1);

        System.out.println("Profile:");
        System.out.println(profile);

        System.out.println("Student from Profile:");
        System.out.println(profile.getStudent());

        session.close();
        sessionFactory.close();
    }
}
```

---

# 17. Understanding `session.find()`

This:

```java
session.find(Student.class, 1);
```

means:

```text
Student.class
     ↓
Which entity?
Student

1
↓
Primary key
```

So Hibernate essentially performs the equivalent of:

```sql
SELECT *
FROM student
WHERE id = 1;
```

You don't manually write that SQL.

Hibernate does it.

---

# 18. Complete Relationship Diagram

Now the whole concept should look like this:

```text
                    JAVA OBJECTS

        ┌─────────────────────────┐
        │        Student          │
        │─────────────────────────│
        │ id                      │
        │ name                    │
        │ email                   │
        │                         │
        │ StudentProfile profile  │
        └────────────┬────────────┘
                     │
                  @OneToOne
                     │
                     ↓
        ┌─────────────────────────┐
        │    StudentProfile       │
        │─────────────────────────│
        │ id                      │
        │ phone                   │
        │ city                    │
        │                         │
        │ Student student         │
        └────────────┬────────────┘
                     │
              mappedBy = "studentProfile"
                     │
                     └──────────────→ Student
```

---

# 19. Database Side

Conceptually:

```text
┌──────────────────────┐
│       student        │
├──────────────────────┤
│ id                   │
│ name                 │
│ email                │
│ student_profile_id   │──────┐
└──────────────────────┘      │
                              │
                              ↓
                    ┌──────────────────────┐
                    │   student_profile    │
                    ├──────────────────────┤
                    │ id                   │
                    │ phone                │
                    │ city                 │
                    └──────────────────────┘
```

---

# 20. The Most Important Rules to Remember

### Rule 1

`@Entity`:

```java
@Entity
```

means:

> Java class → database entity/table.

---

### Rule 2

`@Id`:

```java
@Id
```

means:

> Primary key.

---

### Rule 3

`@OneToOne`:

```java
@OneToOne
```

means:

> One entity → one related entity.

---

### Rule 4

`mappedBy` uses the **Java field name**.

If Student has:

```java
private StudentProfile studentProfile;
```

then:

```java
mappedBy = "studentProfile"
```

---

### Rule 5

The side without `mappedBy` is the owning side in this mapping.

```java
Student
@OneToOne
private StudentProfile studentProfile;
```

is the owning side.

---

### Rule 6

The side with:

```java
mappedBy
```

is the inverse/non-owning side.

```java
StudentProfile
@OneToOne(mappedBy = "studentProfile")
private Student student;
```

---

# 21. Unidirectional vs Bidirectional

### What we learned first

```text
Student ─────→ StudentProfile
```

Only Student knows Profile.

This is:

**Unidirectional One-to-One**

---

### What we learned now

```text
Student ─────→ StudentProfile
   ↑                │
   └────────────────┘
```

Both know each other.

This is:

**Bidirectional One-to-One**

---

# 22. One Last Important Point

Don't confuse:

```java
@OneToOne
```

with:

```java
@OneToOne(mappedBy = "studentProfile")
```

They don't mean exactly the same thing.

```java
@OneToOne
```

says:

> I am defining the relationship here.

While:

```java
@OneToOne(mappedBy = "studentProfile")
```

says:

> The relationship is already defined on the other side, specifically by the `studentProfile` field.

So remember this simple picture:

```text
Student.java

@OneToOne
private StudentProfile studentProfile;
        ↑
        │
        │ mappedBy points here
        │
StudentProfile.java

@OneToOne(mappedBy = "studentProfile")
private Student student;
```

**Next topic:** `@JoinColumn` — this is the key step for understanding **exactly where the foreign key is created, how to control its column name, and what the generated database tables actually look like.

# `@OneToOne` — Part 3: `@JoinColumn`

Now we will learn **`@JoinColumn`**, which is one of the most important concepts in `@OneToOne` mapping.

So far we used:

```java
@OneToOne
private StudentProfile studentProfile;
```

Hibernate decides how to create the relationship column.

Now we will explicitly tell Hibernate:

> **"Create the foreign key column here, and call it `profile_id`."**

---

# 1. What We Are Building

Our relationship will be:

```text
Student                         StudentProfile
---------                       --------------
id                              id
name                            phone
email                           city
profile_id  -----------------> id
```

The `student` table will contain:

```text
profile_id
```

This `profile_id` will be a **foreign key** referring to:

```text
student_profile.id
```

---

# 2. Project Structure

We continue with the same project:

```text
OneToOneMapping
│
├── pom.xml
│
└── src
    └── main
        ├── java
        │   └── com
        │       └── example
        │           ├── Main.java
        │           │
        │           ├── model
        │           │   ├── Student.java
        │           │   └── StudentProfile.java
        │           │
        │           └── util
        │               └── HibernateUtil.java
        │
        └── resources
            └── hibernate.cfg.xml
```

We don't need to change:

```text
pom.xml
HibernateUtil.java
hibernate.cfg.xml
```

Only our entity mapping changes.

---

# 3. Update `Student.java`

Replace your `Student.java` with this:

```java
package com.example.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.JoinColumn;
import jakarta.persistence.OneToOne;

@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;

    private String email;

    @OneToOne
    @JoinColumn(name = "profile_id")
    private StudentProfile studentProfile;

    public Student() {
    }

    public Student(String name, String email, StudentProfile studentProfile) {
        this.name = name;
        this.email = email;
        this.studentProfile = studentProfile;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public StudentProfile getStudentProfile() {
        return studentProfile;
    }

    public void setStudentProfile(StudentProfile studentProfile) {
        this.studentProfile = studentProfile;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}
```

The important addition is:

```java
@JoinColumn(name = "profile_id")
```

---

# 4. What Is `@JoinColumn`?

Let's understand the name first.

```java
@JoinColumn
```

means:

> **This is the database column that will be used to connect the two tables.**

In our case:

```java
@JoinColumn(name = "profile_id")
```

means:

> Create/use a column called `profile_id` in the Student table to store the ID of the StudentProfile.

---

# 5. Before `@JoinColumn`

We had:

```java
@OneToOne
private StudentProfile studentProfile;
```

Hibernate had to decide the column name.

It might generate something similar to:

```text
studentProfile_id
```

But we don't want to depend on Hibernate's default naming.

We want to explicitly say:

```text
profile_id
```

Therefore:

```java
@OneToOne
@JoinColumn(name = "profile_id")
private StudentProfile studentProfile;
```

---

# 6. Understand This Line by Line

Look at:

```java
@OneToOne
@JoinColumn(name = "profile_id")
private StudentProfile studentProfile;
```

### First:

```java
@OneToOne
```

means:

> Student has one StudentProfile.

### Second:

```java
@JoinColumn
```

means:

> Use a database column to create this relationship.

### Third:

```java
name = "profile_id"
```

means:

> Name that database column `profile_id`.

### Finally:

```java
private StudentProfile studentProfile;
```

is the Java object reference.

So:

```text
Java
-------------------------
studentProfile

Database
-------------------------
profile_id
```

They are connected through the ORM mapping.

---

# 7. Update `StudentProfile.java`

We keep the bidirectional relationship.

Use:

```java
package com.example.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.OneToOne;

@Entity
public class StudentProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String phone;

    private String city;

    @OneToOne(mappedBy = "studentProfile")
    private Student student;

    public StudentProfile() {
    }

    public StudentProfile(String phone, String city) {
        this.phone = phone;
        this.city = city;
    }

    public int getId() {
        return id;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public Student getStudent() {
        return student;
    }

    public void setStudent(Student student) {
        this.student = student;
    }

    @Override
    public String toString() {
        return "StudentProfile{" +
                "id=" + id +
                ", phone='" + phone + '\'' +
                ", city='" + city + '\'' +
                '}';
    }
}
```

Notice:

```java
@OneToOne(mappedBy = "studentProfile")
```

is still here.

Why?

Because `Student` owns the relationship:

```java
@OneToOne
@JoinColumn(name = "profile_id")
private StudentProfile studentProfile;
```

And `StudentProfile` says:

```java
mappedBy = "studentProfile"
```

---

# 8. Database Structure

Now our database relationship becomes much clearer.

### Student table

```text
student
-----------------------------------------
id
name
email
profile_id
```

### Student Profile table

```text
student_profile
-----------------------------------------
id
phone
city
```

Relationship:

```text
student.profile_id
       |
       ↓
student_profile.id
```

---

# 9. What Is a Foreign Key?

Suppose:

```text
student_profile
----------------
id
1
```

Now:

```text
student
--------------------------------
id | name  | profile_id
--------------------------------
1  | Rahul | 1
```

Here:

```text
student.profile_id = 1
```

points to:

```text
student_profile.id = 1
```

That is the relationship.

The `profile_id` is acting as a **foreign key**.

---

# 10. Why Is It Called a Foreign Key?

Because the value comes from the primary key of **another table**.

We have:

```text
StudentProfile
    |
    └── id = 1
```

And:

```text
Student
    |
    └── profile_id = 1
```

The `1` in `profile_id` refers to another table.

Hence:

```text
Foreign Key
```

---

# 11. `@JoinColumn` Does Not Create a New Java Field

This is important.

We already have:

```java
private StudentProfile studentProfile;
```

We do **not** write:

```java
private int profile_id;
```

Don't do this:

```java
@OneToOne
@JoinColumn(name = "profile_id")
private StudentProfile studentProfile;

private int profile_id;   // ❌ Don't do this
```

Hibernate manages the foreign key column through the relationship.

---

# 12. Update `Main.java`

Now let's insert data.

Use:

```java
package com.example;

import com.example.model.Student;
import com.example.model.StudentProfile;
import com.example.util.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class Main {

    public static void main(String[] args) {

        SessionFactory sessionFactory =
                HibernateUtil.getSessionFactory();

        Session session =
                sessionFactory.openSession();

        Transaction transaction = null;

        try {

            transaction = session.beginTransaction();

            StudentProfile profile =
                    new StudentProfile(
                            "9876543210",
                            "Delhi"
                    );

            Student student =
                    new Student(
                            "Rahul",
                            "rahul@gmail.com",
                            profile
                    );

            profile.setStudent(student);

            session.persist(profile);

            session.persist(student);

            transaction.commit();

            System.out.println("Data inserted successfully.");

        } catch (Exception e) {

            if (transaction != null) {
                transaction.rollback();
            }

            e.printStackTrace();

        } finally {

            session.close();
            sessionFactory.close();
        }
    }
}
```

---

# 13. What Happens Internally?

Let's follow the complete process.

We create:

```java
StudentProfile profile =
        new StudentProfile(
                "9876543210",
                "Delhi"
        );
```

Java memory:

```text
profile
----------------
phone = 9876543210
city = Delhi
```

Then:

```java
Student student =
        new Student(
                "Rahul",
                "rahul@gmail.com",
                profile
        );
```

Now:

```text
student
------------------------
name = Rahul
email = rahul@gmail.com
studentProfile
       |
       ↓
     profile
```

Then:

```java
profile.setStudent(student);
```

Now the relationship is bidirectional:

```text
student
   ↓
profile
   ↓
student
```

---

# 14. Persist Profile

```java
session.persist(profile);
```

Hibernate saves the profile.

Conceptually:

```sql
INSERT INTO student_profile
(phone, city)
VALUES
('9876543210', 'Delhi');
```

Suppose MySQL generates:

```text
id = 1
```

So:

```text
student_profile
--------------------------------
id | phone      | city
--------------------------------
1  | 9876543210 | Delhi
```

---

# 15. Persist Student

Then:

```java
session.persist(student);
```

Hibernate knows:

```java
studentProfile
```

is connected to:

```text
StudentProfile ID = 1
```

So it can insert:

```sql
INSERT INTO student
(name, email, profile_id)
VALUES
('Rahul', 'rahul@gmail.com', 1);
```

Result:

```text
student
-----------------------------------------
id | name  | email           | profile_id
-----------------------------------------
1  | Rahul | rahul@gmail.com | 1
```

---

# 16. The Complete Database Relationship

Finally:

```text
                 Student
        ┌───────────────────────┐
        │ id = 1                │
        │ name = Rahul          │
        │ email = rahul@gmail   │
        │ profile_id = 1 ───────┼──────┐
        └───────────────────────┘      │
                                       │
                                       ↓
                         StudentProfile
                    ┌───────────────────────┐
                    │ id = 1                │
                    │ phone = 9876543210    │
                    │ city = Delhi          │
                    └───────────────────────┘
```

---

# 17. `@JoinColumn` Is the Key Idea

Remember this:

```java
@JoinColumn(name = "profile_id")
```

means:

> **The Student table will have a column named `profile_id` that joins Student with StudentProfile.**

So:

```java
@OneToOne
@JoinColumn(name = "profile_id")
private StudentProfile studentProfile;
```

can be mentally translated as:

> "Student has one StudentProfile, and use `profile_id` in the Student table to connect them."

---

# 18. `@JoinColumn` vs `mappedBy`

This distinction is extremely important.

### `@JoinColumn`

```java
@JoinColumn(name = "profile_id")
```

is about the **database column**.

It tells Hibernate:

> Which column stores the relationship?

---

### `mappedBy`

```java
mappedBy = "studentProfile"
```

is about the **Java field**.

It tells Hibernate:

> Where is the relationship already defined?

So:

```text
@JoinColumn
      ↓
Database column
      ↓
profile_id
```

while:

```text
mappedBy
      ↓
Java field
      ↓
studentProfile
```

---

# 19. Easy Memory Trick

Remember:

```text
@JoinColumn → COLUMN
mappedBy    → FIELD
```

That's a very useful rule.

For example:

```java
@JoinColumn(name = "profile_id")
```

`profile_id` is a **database column**.

And:

```java
mappedBy = "studentProfile"
```

`studentProfile` is a **Java field**.

---

# 20. Verify in MySQL

Run:

```sql
USE onetoone_db;
```

Then:

```sql
SHOW TABLES;
```

Then:

```sql
DESCRIBE student;
```

You should see something similar to:

```text
id
name
email
profile_id
```

Now:

```sql
DESCRIBE student_profile;
```

You should see:

```text
id
phone
city
```

---

# 21. Check the Actual Data

Run:

```sql
SELECT * FROM student;
```

Expected conceptually:

```text
id | name  | email           | profile_id
1  | Rahul | rahul@gmail.com | 1
```

Then:

```sql
SELECT * FROM student_profile;
```

Expected:

```text
id | phone      | city
1  | 9876543210 | Delhi
```

---

# 22. Check the Relationship Using SQL

Now we can use a SQL `JOIN`.

```sql
SELECT
    s.id,
    s.name,
    s.email,
    p.phone,
    p.city
FROM student s
JOIN student_profile p
ON s.profile_id = p.id;
```

Result:

```text
id | name  | email           | phone      | city
-----------------------------------------------------
1  | Rahul | rahul@gmail.com | 9876543210 | Delhi
```

This is exactly what our Hibernate relationship represents.

---

# 23. What If We Want `profile_id` to Be Unique?

There is one more important thing.

We are saying:

```text
One Student → One Profile
```

But merely having:

```java
@OneToOne
```

doesn't mean you should ignore the database constraint.

We can explicitly make the join column unique:

```java
@OneToOne
@JoinColumn(
        name = "profile_id",
        unique = true
)
private StudentProfile studentProfile;
```

Now:

```text
profile_id
```

must be unique.

That prevents multiple students from using the same profile.

Conceptually, without uniqueness:

```text
Student 1 → Profile 1
Student 2 → Profile 1
Student 3 → Profile 1
```

could potentially happen at the database level.

With:

```java
unique = true
```

the database prevents duplicate relationship values.

---

# 24. Use This Version

For our proper one-to-one example, I recommend changing:

```java
@JoinColumn(name = "profile_id")
```

to:

```java
@JoinColumn(
        name = "profile_id",
        unique = true
)
```

So the complete relationship becomes:

```java
@OneToOne
@JoinColumn(
        name = "profile_id",
        unique = true
)
private StudentProfile studentProfile;
```

---

# 25. One More Concept — Foreign Key

We can also explicitly give the foreign key a name:

```java
@OneToOne
@JoinColumn(
        name = "profile_id",
        unique = true,
        foreignKey = @ForeignKey(name = "fk_student_profile")
)
private StudentProfile studentProfile;
```

But **don't worry about this yet**.

For a beginner, keep it simple:

```java
@OneToOne
@JoinColumn(name = "profile_id", unique = true)
private StudentProfile studentProfile;
```

---

# 26. What You Should Understand After This Part

At this point you should understand these four concepts:

```text
@Entity
   ↓
Class is mapped to table

@Id
   ↓
Primary key

@OneToOne
   ↓
One entity has one related entity

@JoinColumn
   ↓
Defines the relationship column
```

And for bidirectional mapping:

```text
mappedBy
   ↓
Points to the Java field on the owning side
```

---

# 27. Final Mapping

### `Student.java`

```java
@OneToOne
@JoinColumn(name = "profile_id", unique = true)
private StudentProfile studentProfile;
```

### `StudentProfile.java`

```java
@OneToOne(mappedBy = "studentProfile")
private Student student;
```

Therefore:

```text
              Java
                │
       ┌────────┴────────┐
       ↓                 ↓
    Student       StudentProfile
       │                 │
       │                 │
       └──── @OneToOne ───┘
                │
                ↓
           @JoinColumn
                │
                ↓
          profile_id
                │
                ↓
       Foreign Key Relationship
                │
                ↓
             MySQL
```

### The three things to memorize:

> **`@OneToOne` = relationship**

> **`@JoinColumn` = where the foreign key column is**

> **`mappedBy` = relationship is already owned by the other side**

The next step is **`@OneToOne` with `@JoinColumn` in the other direction**, including **which table should own the foreign key, `optional = false`, `nullable = false`, and how to decide the owning side in real projects**.


# `@OneToOne` — Part 4: Owning Side, `optional`, `nullable`, and Foreign Key

Now we will understand the **most confusing part of `@OneToOne`**:

* Which side should own the relationship?
* Where should the foreign key be?
* What does `optional = false` mean?
* What does `nullable = false` mean?
* Why do we sometimes put `@JoinColumn` on `Student` and sometimes on `StudentProfile`?

We will keep the project **copy-paste ready** and still **not use CascadeType**.

---

# 1. Our Current Relationship

Currently we have:

```text id="6e4f7e"
Student                    StudentProfile
--------                   --------------
id                         id
name                       phone
email                      city
profile_id  -------------> id
```

Our Java mapping is:

```java id="2fbyv3"
@OneToOne
@JoinColumn(name = "profile_id", unique = true)
private StudentProfile studentProfile;
```

and:

```java id="y8g4ph"
@OneToOne(mappedBy = "studentProfile")
private Student student;
```

So the foreign key is in:

```text id="bnhlqm"
student.profile_id
```

---

# 2. What Does "Owning Side" Mean?

The **owning side** is the entity that controls the relationship in the database.

In our example:

```java id="ryv3pq"
@OneToOne
@JoinColumn(name = "profile_id")
private StudentProfile studentProfile;
```

is the owning side.

Why?

Because it contains:

```java id="w2g1k0"
@JoinColumn
```

The other side:

```java id="3w7u9d"
@OneToOne(mappedBy = "studentProfile")
private Student student;
```

is the inverse side.

---

# 3. Easy Rule

For a bidirectional relationship:

```text id="q2k9kj"
@JoinColumn
       ↓
Owning Side
```

and:

```text id="m4h9c3"
mappedBy
       ↓
Non-owning / Inverse Side
```

So remember:

> **`@JoinColumn` → Owner**

> **`mappedBy` → Inverse**

---

# 4. Why Does the Owning Side Matter?

Suppose we have:

```java id="x2kz8p"
student.setStudentProfile(profile);
```

Hibernate needs to know:

> Which side controls the database relationship?

It looks at the owning side.

In our case:

```text id="f8bq20"
Student
   |
   | owns relationship
   ↓
StudentProfile
```

The foreign key is:

```text id="mb3s5r"
student.profile_id
```

---

# 5. Can StudentProfile Be the Owning Side?

Yes.

There is nothing special about `Student`.

We could instead decide:

```text id="b3f4d7"
StudentProfile
      |
      | owns relationship
      ↓
Student
```

Then the foreign key would be stored in the `student_profile` table.

For example:

```text id="z0a8bc"
student_profile
---------------------------
id
phone
city
student_id  ─────────────→ student.id
```

This is completely valid.

---

# 6. Which Side Should Own the Relationship?

This is a **database design decision**.

Ask:

> Which entity should contain the foreign key?

For our example, we decided:

```text id="x9by8j"
student.profile_id
```

because the student is associated with a profile.

But another design could be:

```text id="8v4x4v"
student_profile.student_id
```

Both can represent a one-to-one relationship.

---

# 7. Real-Life Example

Consider:

```text id="8xv7e1"
User
UserProfile
```

You could have:

```text id="o0tq5k"
user
--------------------
id
name
profile_id
```

Or:

```text id="w3y5m4"
user_profile
--------------------
id
phone
user_id
```

Both are possible.

You choose based on your database design.

---

# 8. Now Learn `optional = false`

Let's say every student **must have** a profile.

Currently:

```java id="9xv4za"
@OneToOne
@JoinColumn(name = "profile_id", unique = true)
private StudentProfile studentProfile;
```

The relationship can potentially be absent.

For example:

```text id="w9q7c0"
Student
-----------------------------
id | name | profile_id
-----------------------------
1  | Rahul| NULL
```

That means Rahul doesn't have a profile.

If our application says:

> Every student must have a profile.

then we can write:

```java id="i8tq5a"
@OneToOne(optional = false)
@JoinColumn(name = "profile_id", unique = true)
private StudentProfile studentProfile;
```

---

# 9. What Does `optional = false` Mean?

```java id="v9u7wl"
@OneToOne(optional = false)
```

means:

> The relationship is required.

In simple English:

```text id="c3ckzo"
Student MUST have StudentProfile
```

instead of:

```text id="kz5m0v"
Student MAY have StudentProfile
```

---

# 10. `optional = true`

By default, the relationship can be optional.

Conceptually:

```java id="t8pk3f"
@OneToOne(optional = true)
```

means:

```text id="6cgy6q"
Student
   |
   ├── Profile
   |
   └── OR no Profile
```

While:

```java id="21x9fs"
@OneToOne(optional = false)
```

means:

```text id="apxv26"
Student
   |
   └── MUST have Profile
```

---

# 11. Now `nullable = false`

Look at:

```java id="d7ftl2"
@JoinColumn(
        name = "profile_id",
        unique = true,
        nullable = false
)
```

`nullable = false` means:

> The `profile_id` database column cannot contain `NULL`.

So the database will enforce:

```text id="2x3i5k"
profile_id = NULL ❌
```

and require:

```text id="gd0jvb"
profile_id = 1 ✅
```

---

# 12. `optional = false` vs `nullable = false`

This is very important.

### `optional = false`

```java id="8j9x9s"
@OneToOne(optional = false)
```

describes the **relationship requirement**.

### `nullable = false`

```java id="ezgqjv"
@JoinColumn(nullable = false)
```

describes the **database column constraint**.

So:

```text id="wqy44u"
optional = false
        ↓
Relationship must exist

nullable = false
        ↓
Database column cannot be NULL
```

They are related concepts but not exactly the same thing.

---

# 13. Our Stronger Mapping

For our example, let's make the relationship mandatory.

Change `Student.java` to:

```java id="6ff8ov"
package com.example.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.JoinColumn;
import jakarta.persistence.OneToOne;

@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;

    private String email;

    @OneToOne(optional = false)
    @JoinColumn(
            name = "profile_id",
            unique = true,
            nullable = false
    )
    private StudentProfile studentProfile;

    public Student() {
    }

    public Student(
            String name,
            String email,
            StudentProfile studentProfile
    ) {
        this.name = name;
        this.email = email;
        this.studentProfile = studentProfile;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public StudentProfile getStudentProfile() {
        return studentProfile;
    }

    public void setStudentProfile(StudentProfile studentProfile) {
        this.studentProfile = studentProfile;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}
```

---

# 14. StudentProfile Remains the Same

Use:

```java id="zeb6ft"
package com.example.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.OneToOne;

@Entity
public class StudentProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String phone;

    private String city;

    @OneToOne(mappedBy = "studentProfile")
    private Student student;

    public StudentProfile() {
    }

    public StudentProfile(String phone, String city) {
        this.phone = phone;
        this.city = city;
    }

    public int getId() {
        return id;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public Student getStudent() {
        return student;
    }

    public void setStudent(Student student) {
        this.student = student;
    }

    @Override
    public String toString() {
        return "StudentProfile{" +
                "id=" + id +
                ", phone='" + phone + '\'' +
                ", city='" + city + '\'' +
                '}';
    }
}
```

---

# 15. Complete `Main.java`

Now use:

```java id="l9dfw5"
package com.example;

import com.example.model.Student;
import com.example.model.StudentProfile;
import com.example.util.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class Main {

    public static void main(String[] args) {

        SessionFactory sessionFactory =
                HibernateUtil.getSessionFactory();

        Session session =
                sessionFactory.openSession();

        Transaction transaction = null;

        try {

            transaction = session.beginTransaction();

            StudentProfile profile =
                    new StudentProfile(
                            "9876543210",
                            "Delhi"
                    );

            Student student =
                    new Student(
                            "Rahul",
                            "rahul@gmail.com",
                            profile
                    );

            profile.setStudent(student);

            session.persist(profile);

            session.persist(student);

            transaction.commit();

            System.out.println(
                    "Student and Profile saved successfully."
            );

        } catch (Exception e) {

            if (transaction != null) {
                transaction.rollback();
            }

            e.printStackTrace();

        } finally {

            session.close();
            sessionFactory.close();
        }
    }
}
```

---

# 16. Let's Understand the Complete Mapping

Look at this:

```java id="e9r7aq"
@OneToOne(optional = false)
@JoinColumn(
        name = "profile_id",
        unique = true,
        nullable = false
)
private StudentProfile studentProfile;
```

There are now **four concepts**.

### 1. `@OneToOne`

```text id="j6jyl6"
One Student → One Profile
```

### 2. `optional = false`

```text id="nqf0f0"
Student must have Profile
```

### 3. `name = "profile_id"`

```text id="3qkv3q"
Foreign-key column name = profile_id
```

### 4. `unique = true`

```text id="px9e6v"
One profile ID cannot be used by multiple students
```

### 5. `nullable = false`

```text id="0rbdqk"
profile_id cannot be NULL
```

---

# 17. Why `unique = true` Is Important

This is subtle.

Suppose we have:

```text id="7ovjli"
student
--------------------------------
id | name  | profile_id
--------------------------------
1  | Rahul | 1
2  | Amit  | 1
```

Both students point to:

```text id="01xyh8"
student_profile.id = 1
```

Then:

```text id="j9t4z1"
Rahul → Profile 1
Amit  → Profile 1
```

That's actually **many-to-one**, not one-to-one.

To prevent this, we use:

```java id="2cqfqc"
unique = true
```

Now:

```text id="4e2xv7"
Profile 1 → Rahul
```

and another student cannot also use:

```text id="7l4ygu"
profile_id = 1
```

---

# 18. Database View

Our table should conceptually become:

```text id="m1r04a"
STUDENT
--------------------------------------------------
id | name  | email           | profile_id
--------------------------------------------------
1  | Rahul | rahul@gmail.com | 1
2  | Amit  | amit@gmail.com  | 2
```

And:

```text id="yj40f0"
STUDENT_PROFILE
------------------------------------
id | phone      | city
------------------------------------
1  | 9876543210 | Delhi
2  | 9999999999 | Mumbai
```

Relationship:

```text id="6w6f0x"
Rahul → Profile 1
Amit  → Profile 2
```

---

# 19. What Happens If We Try Duplicate Profile?

Suppose:

```text id="p5z3ty"
Student 1 → Profile 1
Student 2 → Profile 1
```

Because:

```java id="f6cbrk"
unique = true
```

the database will reject the second relationship.

That's how the database helps enforce our one-to-one design.

---

# 20. What Happens If Profile Is Null?

Because we have:

```java id="w5p9u4"
optional = false
```

and:

```java id="25m2v3"
nullable = false
```

we are saying:

```text id="1cpl8g"
Student
   |
   └── StudentProfile is REQUIRED
```

So this is not valid:

```java id="04yr7h"
Student student =
        new Student(
                "Rahul",
                "rahul@gmail.com",
                null
        );
```

The relationship should not be missing.

---

# 21. A Very Important Real-World Point

Don't automatically use:

```java id="nqj4dc"
optional = false
nullable = false
```

for every `@OneToOne`.

Use them when the business rule actually says the relationship is mandatory.

For example:

### Student → Profile

Maybe:

```text id="7r33zq"
Every student MUST have profile
```

Then:

```java id="1iyjco"
optional = false
nullable = false
```

makes sense.

But maybe:

```text id="kqg1lb"
Student can register first
Profile can be created later
```

Then the relationship should be optional.

---

# 22. Another Example

Imagine:

```text id="31x3w9"
Employee
EmployeeParkingSpace
```

Maybe every employee doesn't have a parking space.

Then:

```java id="e6uq3a"
@OneToOne
@JoinColumn(name = "parking_space_id")
private ParkingSpace parkingSpace;
```

makes more sense.

You wouldn't necessarily use:

```java id="44w3n0"
optional = false
```

because some employees may not have parking.

---

# 23. The Most Important Question in an Interview

You may be asked:

> **What is the owning side in a bidirectional `@OneToOne` relationship?**

Answer:

> The owning side is the side that contains the foreign-key mapping, typically using `@JoinColumn`. The other side uses `mappedBy` to indicate that the relationship is already mapped by the owning side.

Example:

```java id="g7wqk7"
@OneToOne
@JoinColumn(name = "profile_id")
private StudentProfile studentProfile;
```

is owning side.

And:

```java id="o5y0n2"
@OneToOne(mappedBy = "studentProfile")
private Student student;
```

is inverse side.

---

# 24. Another Important Interview Question

> **What does `mappedBy` represent?**

Answer:

> `mappedBy` points to the Java field on the owning side that manages the relationship. It is not the database column name.

Example:

```java id="x6ufgb"
private StudentProfile studentProfile;
```

Therefore:

```java id="6i6m4x"
mappedBy = "studentProfile"
```

Correct.

---

# 25. Another Interview Question

> **What does `@JoinColumn` do?**

Answer:

> `@JoinColumn` specifies the database column used to join the associated entities. In our example, `profile_id` is stored in the Student table and references the StudentProfile's primary key.

---

# 26. One-to-One Mental Model

When you see:

```java id="yqumnj"
@OneToOne
@JoinColumn(name = "profile_id")
private StudentProfile studentProfile;
```

think:

```text id="x5p6nz"
Java:

Student
   |
   ↓
StudentProfile


Database:

student.profile_id
       |
       ↓
student_profile.id
```

When you see:

```java id="6o4d9j"
@OneToOne(mappedBy = "studentProfile")
private Student student;
```

think:

```text id="l4w9f4"
"I don't own the foreign key.
Student owns it."
```

---

# 27. Final Cheat Sheet

| Annotation/Property | Meaning                                 |
| ------------------- | --------------------------------------- |
| `@Entity`           | Java class becomes an entity            |
| `@Id`               | Primary key                             |
| `@GeneratedValue`   | Automatically generate ID               |
| `@OneToOne`         | One-to-one relationship                 |
| `@JoinColumn`       | Defines relationship/foreign-key column |
| `name`              | Name of database column                 |
| `unique = true`     | Prevent duplicate referenced IDs        |
| `nullable = false`  | Database column cannot be NULL          |
| `optional = false`  | Relationship is required                |
| `mappedBy`          | Points to owning-side Java field        |

---

# 28. Final Architecture

```text
                    JAVA
                      │
                      ↓
             ┌─────────────────┐
             │     Student     │
             │─────────────────│
             │ id              │
             │ name            │
             │ email           │
             │                 │
             │ studentProfile  │
             └────────┬────────┘
                      │
                  @OneToOne
                      │
               @JoinColumn
                      │
                profile_id
                      │
                      ↓
              ┌─────────────────┐
              │ StudentProfile  │
              │─────────────────│
              │ id              │
              │ phone           │
              │ city            │
              │                 │
              │ student         │
              └─────────────────┘
                      ↑
                      │
              mappedBy =
              "studentProfile"
```

The key thing to understand now is:

```text id="6ocqv0"
@OneToOne
       ↓
Relationship

@JoinColumn
       ↓
Foreign-key column

mappedBy
       ↓
Other side does not own relationship

unique = true
       ↓
Enforce one-to-one at DB level

optional = false
       ↓
Relationship is required

nullable = false
       ↓
Foreign-key column cannot be NULL
```

**Next:** we can build a proper CRUD application around this `@OneToOne` mapping — **INSERT, SELECT, UPDATE, DELETE**, with separate methods and simple Hibernate code, so you understand how `Session.persist()`, `find()`, `merge()`, and `remove()` work with the relationship.


# Part 5 — CRUD Operations with `@OneToOne`

Now we will use the same project and learn **CRUD**:

* **C** → Create → `persist()`
* **R** → Read → `find()`
* **U** → Update → `setters` / `merge()`
* **D** → Delete → `remove()`

We will continue with the same **Student ↔ StudentProfile** project using Hibernate ORM, MySQL and Maven.

---

## 1. Our Current Relationship

We already have:

```text
Student
-----------------------
id
name
email
profile_id  -----------+
                       |
                       v
StudentProfile
-----------------------
id
phone
city
```

The important point is:

```text
Student.profile_id → StudentProfile.id
```

And in Java:

```java
Student
   |
   | @OneToOne
   v
StudentProfile
```

---

# 2. CREATE — Insert Data

We will first insert a `StudentProfile`, then a `Student`.

Why?

Because `Student` contains:

```java
@JoinColumn(name = "profile_id")
```

So the student needs an existing profile ID.

### `Main.java`

Replace your existing `Main.java` with:

```java
package com.example;

import com.example.model.Student;
import com.example.model.StudentProfile;
import com.example.util.HibernateUtil;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class Main {

    public static void main(String[] args) {

        Session session = HibernateUtil.getSessionFactory().openSession();

        Transaction transaction = null;

        try {

            transaction = session.beginTransaction();

            // Create StudentProfile
            StudentProfile profile = new StudentProfile();

            profile.setPhone("9876543210");
            profile.setCity("Delhi");

            // Save StudentProfile first
            session.persist(profile);


            // Create Student
            Student student = new Student();

            student.setName("Rahul");
            student.setEmail("rahul@gmail.com");

            // Connect Student with StudentProfile
            student.setStudentProfile(profile);

            // Connect Profile with Student
            profile.setStudent(student);

            // Save Student
            session.persist(student);


            transaction.commit();

            System.out.println("Student and StudentProfile saved successfully!");

        } catch (Exception e) {

            if (transaction != null) {
                transaction.rollback();
            }

            e.printStackTrace();

        } finally {

            session.close();
        }

        HibernateUtil.shutdown();
    }
}
```

---

# 3. Understand CREATE Line by Line

### Open a Hibernate Session

```java
Session session =
        HibernateUtil.getSessionFactory().openSession();
```

A `Session` is used to communicate with the database through Hibernate.

Think:

```text
Java Application
       ↓
   Hibernate
       ↓
    Session
       ↓
    MySQL
```

---

### Start Transaction

```java
Transaction transaction = null;
```

We create a variable to store our transaction.

Then:

```java
transaction = session.beginTransaction();
```

This starts a database transaction.

Think:

```text
BEGIN TRANSACTION
       ↓
   INSERT data
       ↓
   COMMIT
```

---

# 4. Create Profile

```java
StudentProfile profile = new StudentProfile();
```

This creates a normal Java object.

At this point:

```text
profile
   |
   +-- phone = null
   +-- city  = null
```

Then:

```java
profile.setPhone("9876543210");
profile.setCity("Delhi");
```

Now:

```text
profile
   |
   +-- phone = 9876543210
   +-- city  = Delhi
```

---

# 5. Save Profile

```java
session.persist(profile);
```

Hibernate now knows:

> "This Java object should be inserted into the database."

Hibernate eventually executes SQL similar to:

```sql
INSERT INTO student_profile
(phone, city)
VALUES
('9876543210', 'Delhi');
```

The database generates an ID.

For example:

```text
student_profile

id    phone         city
1     9876543210    Delhi
```

---

# 6. Create Student

```java
Student student = new Student();
```

Then:

```java
student.setName("Rahul");
student.setEmail("rahul@gmail.com");
```

Now:

```text
student
   |
   +-- name  = Rahul
   +-- email = rahul@gmail.com
```

---

# 7. Connect Both Objects

This is extremely important.

```java
student.setStudentProfile(profile);
```

Means:

```text
Student
   |
   +----> StudentProfile
```

But because we made our relationship **bidirectional**, we also do:

```java
profile.setStudent(student);
```

So now:

```text
Student
   |
   | studentProfile
   ↓
StudentProfile
   |
   | student
   ↓
Student
```

---

# 8. Save Student

```java
session.persist(student);
```

Hibernate now knows that the student must be inserted.

Because the profile already has ID `1`, Hibernate can create:

```text
student

id    name     email             profile_id
1     Rahul    rahul@gmail.com   1
```

So:

```text
student.profile_id = student_profile.id
```

---

# 9. Commit

```java
transaction.commit();
```

This permanently commits the changes to the database.

Without `commit()`, your transaction may be rolled back/closed without the changes becoming permanent.

---

# 10. READ — Fetch Data

Now let's learn how to retrieve a student.

Replace `Main.java` with this:

```java
package com.example;

import com.example.model.Student;
import com.example.model.StudentProfile;
import com.example.util.HibernateUtil;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class Main {

    public static void main(String[] args) {

        Session session =
                HibernateUtil.getSessionFactory().openSession();

        try {

            Student student =
                    session.find(Student.class, 1);

            if (student != null) {

                System.out.println("Student ID: "
                        + student.getId());

                System.out.println("Student Name: "
                        + student.getName());

                System.out.println("Student Email: "
                        + student.getEmail());

                StudentProfile profile =
                        student.getStudentProfile();

                if (profile != null) {

                    System.out.println("Phone: "
                            + profile.getPhone());

                    System.out.println("City: "
                            + profile.getCity());
                }

            } else {

                System.out.println("Student not found!");
            }

        } catch (Exception e) {

            e.printStackTrace();

        } finally {

            session.close();
        }

        HibernateUtil.shutdown();
    }
}
```

---

# 11. The Most Important READ Line

```java
Student student =
        session.find(Student.class, 1);
```

This means:

> Find the `Student` whose primary key is `1`.

Hibernate converts this into SQL conceptually similar to:

```sql
SELECT *
FROM student
WHERE id = 1;
```

---

## Why `Student.class`?

Because Hibernate needs to know:

> Which entity are you trying to find?

So:

```java
Student.class
```

means:

```text
Student entity
```

And:

```java
1
```

means:

```text
Primary key = 1
```

Therefore:

```java
session.find(Student.class, 1);
```

means:

```text
Find Student where ID = 1
```

---

# 12. Getting the Profile

Once we have:

```java
Student student = session.find(Student.class, 1);
```

we can do:

```java
StudentProfile profile =
        student.getStudentProfile();
```

Because our `Student` class contains:

```java
private StudentProfile studentProfile;
```

So Java allows:

```java
student.getStudentProfile();
```

Now we can access:

```java
profile.getPhone();
profile.getCity();
```

---

# 13. UPDATE — Updating Data

Suppose Rahul changes his email.

We can load the student:

```java
Student student =
        session.find(Student.class, 1);
```

Then:

```java
student.setEmail("rahul123@gmail.com");
```

Then commit.

Complete example:

```java
package com.example;

import com.example.model.Student;
import com.example.util.HibernateUtil;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class Main {

    public static void main(String[] args) {

        Session session =
                HibernateUtil.getSessionFactory().openSession();

        Transaction transaction = null;

        try {

            transaction = session.beginTransaction();

            Student student =
                    session.find(Student.class, 1);

            if (student != null) {

                student.setEmail("rahul123@gmail.com");

                System.out.println("Email updated!");

            } else {

                System.out.println("Student not found!");
            }

            transaction.commit();

        } catch (Exception e) {

            if (transaction != null) {
                transaction.rollback();
            }

            e.printStackTrace();

        } finally {

            session.close();
        }

        HibernateUtil.shutdown();
    }
}
```

---

# 14. Something Very Important About UPDATE

Notice that we did **not** write:

```java
session.update(student);
```

We simply did:

```java
student.setEmail("rahul123@gmail.com");
```

and:

```java
transaction.commit();
```

Why?

Because:

```java
session.find(Student.class, 1);
```

returns a **managed entity**.

Hibernate is monitoring that object.

So when we do:

```java
student.setEmail("rahul123@gmail.com");
```

Hibernate detects the change.

This mechanism is called:

## Dirty Checking

Example:

```text
Database
   ↓
session.find()
   ↓
Java Object
   ↓
Change property
   ↓
Hibernate detects change
   ↓
UPDATE SQL
   ↓
commit()
```

Hibernate may generate something like:

```sql
UPDATE student
SET email = 'rahul123@gmail.com'
WHERE id = 1;
```

---

# 15. UPDATE Using `merge()`

Now let's understand `merge()`.

`merge()` becomes especially useful when an object is **detached**.

For example:

```java
Student student;
```

is loaded in one session.

Then that session closes.

The object is now detached.

```text
Session 1
   ↓
find()
   ↓
Student object
   ↓
session.close()
   ↓
DETACHED OBJECT
```

We can modify it:

```java
student.setEmail("newemail@gmail.com");
```

Then open another session and use:

```java
session.merge(student);
```

---

## Complete `merge()` Example

```java
package com.example;

import com.example.model.Student;
import com.example.util.HibernateUtil;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class Main {

    public static void main(String[] args) {

        Student student;

        // -------------------------------
        // SESSION 1
        // -------------------------------

        Session session1 =
                HibernateUtil.getSessionFactory().openSession();

        try {

            student =
                    session1.find(Student.class, 1);

        } finally {

            session1.close();
        }


        // Object is now DETACHED


        student.setEmail("newemail@gmail.com");


        // -------------------------------
        // SESSION 2
        // -------------------------------

        Session session2 =
                HibernateUtil.getSessionFactory().openSession();

        Transaction transaction = null;

        try {

            transaction = session2.beginTransaction();

            Student managedStudent =
                    session2.merge(student);

            transaction.commit();

            System.out.println("Student updated using merge()!");

        } catch (Exception e) {

            if (transaction != null) {
                transaction.rollback();
            }

            e.printStackTrace();

        } finally {

            session2.close();
        }

        HibernateUtil.shutdown();
    }
}
```

---

# 16. What Exactly Does `merge()` Do?

This is important for interviews.

Suppose:

```java
Student student
```

is detached.

You call:

```java
Student managedStudent =
        session2.merge(student);
```

Hibernate does **not simply reattach the same object**.

Conceptually:

```text
Detached Student
       |
       | merge()
       ↓
Hibernate
       |
       ↓
Managed Student
```

The returned object:

```java
managedStudent
```

is the managed object.

Therefore, remember:

```java
Student managedStudent = session.merge(student);
```

is better than thinking:

```text
merge() attaches the original object
```

---

# 17. DELETE — Delete Student

Now let's delete a student.

Because our relationship is:

```text
Student
   |
   | profile_id
   ↓
StudentProfile
```

the `Student` contains the foreign key.

Therefore, **do not delete the profile first** while the student still references it.

We will delete:

```text
Student
   ↓
StudentProfile
```

---

## Delete Example

```java
package com.example;

import com.example.model.Student;
import com.example.model.StudentProfile;
import com.example.util.HibernateUtil;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class Main {

    public static void main(String[] args) {

        Session session =
                HibernateUtil.getSessionFactory().openSession();

        Transaction transaction = null;

        try {

            transaction = session.beginTransaction();

            Student student =
                    session.find(Student.class, 1);

            if (student != null) {

                StudentProfile profile =
                        student.getStudentProfile();

                session.remove(student);

                if (profile != null) {
                    session.remove(profile);
                }

                System.out.println(
                        "Student and Profile deleted!"
                );

            } else {

                System.out.println("Student not found!");
            }

            transaction.commit();

        } catch (Exception e) {

            if (transaction != null) {
                transaction.rollback();
            }

            e.printStackTrace();

        } finally {

            session.close();
        }

        HibernateUtil.shutdown();
    }
}
```

---

# 18. Understanding `remove()`

This:

```java
session.remove(student);
```

tells Hibernate:

> Delete this entity from the database.

Hibernate will eventually execute something similar to:

```sql
DELETE FROM student
WHERE id = 1;
```

Then:

```java
session.remove(profile);
```

deletes:

```sql
DELETE FROM student_profile
WHERE id = 1;
```

---

# 19. Why We Don't Delete Profile First

Suppose database contains:

```text
student

id    name     profile_id
1     Rahul    1
```

and:

```text
student_profile

id
1
```

There is a relationship:

```text
student.profile_id
       ↓
student_profile.id
```

If we execute:

```sql
DELETE FROM student_profile
WHERE id = 1;
```

the student is still pointing to profile `1`.

Depending on the foreign-key configuration, MySQL can reject the deletion because of the foreign-key constraint.

So, without cascade, understand this order:

```text
DELETE Student
      ↓
DELETE StudentProfile
```

---

# 20. CRUD Summary

| Operation              | Hibernate Method    |
| ---------------------- | ------------------- |
| Create                 | `persist()`         |
| Read                   | `find()`            |
| Update managed entity  | Setter + `commit()` |
| Update detached entity | `merge()`           |
| Delete                 | `remove()`          |

The complete flow is:

```text
             Hibernate CRUD

CREATE
   ↓
session.persist()

READ
   ↓
session.find()

UPDATE
   ↓
setter + commit()
       OR
session.merge()

DELETE
   ↓
session.remove()
```

---

# 21. Entity Lifecycle — Very Important

Now you are ready for one of the most important Hibernate concepts: **Entity Lifecycle**.

An entity generally moves through these states:

```text
       new Student()
            |
            ↓
        TRANSIENT
            |
       persist()
            ↓
         MANAGED
            |
       session.close()
            ↓
        DETACHED
            |
         remove()
            ↓
         REMOVED
```

Let's understand them.

### 1. Transient

```java
Student student = new Student();
```

The object exists only in Java.

It is not associated with a Hibernate session.

```text
Java Object
     ↓
No database relationship yet
```

---

### 2. Managed

```java
session.persist(student);
```

Now Hibernate manages the entity.

```text
Java Object
     ↕
Hibernate Session
     ↕
Database
```

Hibernate can track its changes.

---

### 3. Detached

```java
session.close();
```

The entity object still exists in Java, but it is no longer managed by that session.

```text
Student object
     ↓
Detached
```

You can later use:

```java
session.merge(student);
```

to copy its state into a managed entity.

---

### 4. Removed

```java
session.remove(student);
```

The entity is scheduled for deletion.

After the transaction is committed, the corresponding database row is deleted.

---

# 22. The Big Picture

You now have the complete basic understanding:

```text
                    Hibernate ORM
                         |
                         ↓
                 Object Relationship
                         |
                     @OneToOne
                         |
          +--------------+--------------+
          |                             |
       Student                   StudentProfile
          |                             |
          +---------- profile_id -------+
                         |
                     Database
                         |
                       MySQL
```

And CRUD:

```text
CREATE → persist()
READ   → find()
UPDATE → dirty checking / merge()
DELETE → remove()
```

### One important rule for this project

We are **still not using CascadeType**.

So whenever we work with:

```text
Student ↔ StudentProfile
```

we explicitly manage both entities ourselves. This is intentional so you understand what Hibernate is actually doing before introducing cascade.
