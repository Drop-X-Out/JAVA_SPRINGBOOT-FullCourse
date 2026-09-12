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
