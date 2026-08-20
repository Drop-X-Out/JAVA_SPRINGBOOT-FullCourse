# Hibernate Bulk Operation: Native SQL (
Let's build a **real-world example**.

Imagine you are working at a university.

The management says:

> "The Java course has been replaced with Spring Boot. Update all students who are enrolled in Java."

There are **50,000 students** in the database.

Should you do this?

```java
Student s1 = session.find(Student.class, 1);
s1.setCourse("Spring Boot");

Student s2 = session.find(Student.class, 2);
s2.setCourse("Spring Boot");

Student s3 = session.find(Student.class, 3);
s3.setCourse("Spring Boot");
```

No.

---

# Option 1: Updating records one by one

```text
Application
        ↓

Hibernate
        ↓

Student 1
        ↓

UPDATE Query

Student 2
        ↓

UPDATE Query

Student 3
        ↓

UPDATE Query

...

50,000 UPDATE queries
```

This is very slow.

---

# Option 2: HQL Bulk Update

```java
String hql =
        "UPDATE Student " +
        "SET course='Spring Boot' " +
        "WHERE course='Java'";
```

Hibernate converts this into SQL.

---

# But what if the database administrator already wrote this SQL query?

```sql
UPDATE student
SET course='Spring Boot'
WHERE course='Java'
AND status='active'
AND registration_date<'2026-01-01';
```

Or what if they wrote a stored procedure?

```sql
CALL update_student_courses();
```

HQL may not support everything.

This is why **Native SQL exists**.

---

# Project Structure

```text
HibernateNativeSQL
│
├── pom.xml
│
├── src
│
└── main
    │
    ├── java
    │
    └── com.example
        │
        ├── Student.java
        │
        ├── HibernateUtil.java
        │
        ├── InsertStudents.java
        │
        └── NativeSQLBulkUpdate.java
    │
    └── resources
        │
        └── hibernate.cfg.xml
```

---

# Step 1: Create the database

```sql
CREATE DATABASE university;
```

---

# Step 2: Create the table

```sql
CREATE TABLE student
(
    id INT PRIMARY KEY,
    name VARCHAR(100),
    course VARCHAR(100),
    status VARCHAR(50),
    registration_date DATE
);
```

---

# Step 3: Insert sample data

```sql
INSERT INTO student VALUES
(1,'Jack','Java','active','2025-10-10');

INSERT INTO student VALUES
(2,'Sara','Java','active','2025-11-15');

INSERT INTO student VALUES
(3,'Ahmed','Python','active','2026-03-10');

INSERT INTO student VALUES
(4,'Pooja','Java','inactive','2025-09-12');

INSERT INTO student VALUES
(5,'Zain','Java','active','2025-08-05');
```

---

# Database before the update

| id | name   | course | status   |
| -- | ------ | ------ | -------- |
| 1  | Jack    | Java   | active   |
| 2  | Sara   | Java   | active   |
| 3  | Ahmed  | Python | active   |
| 4  | Pooja | Java   | inactive |
| 5  | Zain   | Java   | active   |

---

# Step 4: pom.xml

```xml
<project>

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>

    <artifactId>HibernateNativeSQL</artifactId>

    <version>1.0</version>

    <dependencies>

        <dependency>

            <groupId>org.hibernate</groupId>

            <artifactId>hibernate-core</artifactId>

            <version>5.6.15.Final</version>

        </dependency>

        <dependency>

            <groupId>mysql</groupId>

            <artifactId>mysql-connector-java</artifactId>

            <version>8.0.33</version>

        </dependency>

        <dependency>

            <groupId>javax.persistence</groupId>

            <artifactId>javax.persistence-api</artifactId>

            <version>2.2</version>

        </dependency>

    </dependencies>

</project>
```

---

# Step 5: Student.java

```java
package com.example;

import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "student")
public class Student
{
    @Id
    private int id;

    private String name;

    private String course;

    private String status;

    private java.sql.Date registration_date;

    public Student()
    {
    }

    public Student(
            int id,
            String name,
            String course,
            String status,
            java.sql.Date registration_date)
    {
        this.id = id;
        this.name = name;
        this.course = course;
        this.status = status;
        this.registration_date =
                registration_date;
    }

    @Override
    public String toString()
    {
        return id + " "
                + name + " "
                + course;
    }
}
```

---

# Step 6: hibernate.cfg.xml

```xml
<?xml version="1.0"?>

<!DOCTYPE hibernate-configuration PUBLIC
"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <property name="hibernate.connection.driver_class">

            com.mysql.cj.jdbc.Driver

        </property>

        <property name="hibernate.connection.url">

            jdbc:mysql://localhost:3306/university

        </property>

        <property name="hibernate.connection.username">

            root

        </property>

        <property name="hibernate.connection.password">

            root

        </property>

        <property name="hibernate.dialect">

            org.hibernate.dialect.MySQL8Dialect

        </property>

        <property name="hibernate.show_sql">

            true

        </property>

        <property name="hibernate.hbm2ddl.auto">

            update

        </property>

        <mapping class="com.example.Student"/>

    </session-factory>

</hibernate-configuration>
```

---

# Step 7: HibernateUtil.java

```java
package com.example;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil
{
    private static SessionFactory
            sessionFactory;

    static
    {
        sessionFactory =
                new Configuration()
                        .configure()
                        .buildSessionFactory();
    }

    public static SessionFactory
    getSessionFactory()
    {
        return sessionFactory;
    }
}
```

---

# Step 8: NativeSQLBulkUpdate.java

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.NativeQuery;

public class NativeSQLBulkUpdate
{
    public static void main(
            String[] args)
    {
        Session session =
                HibernateUtil
                        .getSessionFactory()
                        .openSession();

        Transaction transaction =
                session.beginTransaction();

        String sql =
                "UPDATE student " +
                "SET course='Spring Boot' " +
                "WHERE course='Java' " +
                "AND status='active' " +
                "AND registration_date<'2026-01-01'";

        NativeQuery query =
                session.createNativeQuery(sql);

        int updatedRows =
                query.executeUpdate();

        transaction.commit();

        System.out.println(
                updatedRows
                + " students updated");

        session.close();
    }
}
```

---

# Output

```text
3 students updated
```

---

# Database after the update

| id | name   | course      | status   |
| -- | ------ | ----------- | -------- |
| 1  | Jack    | Spring Boot | active   |
| 2  | Sara   | Spring Boot | active   |
| 3  | Ahmed  | Python      | active   |
| 4  | Pooja | Java        | inactive |
| 5  | Zain   | Spring Boot | active   |

---

# A more powerful real-world example: Using a MySQL function

Suppose the manager asks:

> Show all students who registered this month.

MySQL provides a function:

```sql
MONTH()
```

SQL:

```sql
SELECT *
FROM student
WHERE MONTH(registration_date)
=
MONTH(CURDATE());
```

Many companies use hundreds of database-specific functions.

That's why Native SQL is still important.

---

# The biggest reason companies use Native SQL

```text
Banking systems
       ↓

Hospital systems
       ↓

ERP systems
       ↓

Legacy applications
```

These systems may contain **thousands of SQL queries** written over many years.

Instead of rewriting everything in HQL, developers simply execute the existing SQL through Hibernate.

That's the real reason Native SQL still exists, even though HQL is usually the preferred choice.

> If HQL is so useful, why didn't Hibernate's developers simply add every SQL feature to HQL?

Because doing that would completely defeat the purpose of HQL.

---

# What is the main goal of HQL?

The goal of HQL is:

```text
Write once.

Run on any database.
```

---

Suppose you write this HQL query:

```java
String hql =
        "FROM Student " +
        "WHERE course = 'Java'";
```

Hibernate converts it.

MySQL:

```sql
SELECT *
FROM student
WHERE course = 'Java';
```

PostgreSQL:

```sql
SELECT *
FROM student
WHERE course = 'Java';
```

Oracle:

```sql
SELECT *
FROM student
WHERE course = 'Java';
```

The same HQL works everywhere.

---

# The problem: databases are different

Every database has its own special features.

| Feature     | MySQL | PostgreSQL | Oracle |
| ----------- | ----- | ---------- | ------ |
| `LIMIT`     | ✅     | ✅          | ❌      |
| `TOP`       | ❌     | ❌          | ❌      |
| `ROWNUM`    | ❌     | ❌          | ✅      |
| `CURDATE()` | ✅     | ❌          | ❌      |
| `SYSDATE`   | ❌     | ❌          | ✅      |
| `IFNULL()`  | ✅     | ❌          | ❌      |
| `NVL()`     | ❌     | ❌          | ✅      |

---

Suppose Hibernate added this to HQL:

```java
String hql =
        "FROM Student LIMIT 5";
```

It would work in MySQL.

But Oracle doesn't support `LIMIT`.

What should Hibernate do?

```text
MySQL
↓

LIMIT 5

Oracle
↓

ROWNUM <= 5

PostgreSQL
↓

LIMIT 5
```

Now Hibernate must translate every database-specific feature into every other database's syntax.

That's extremely difficult.

---

# Another example: date functions

MySQL:

```sql
CURDATE()
```

Oracle:

```sql
SYSDATE
```

PostgreSQL:

```sql
CURRENT_DATE
```

If HQL supported every function from every database, HQL itself would become as large and complicated as SQL.

---

# The philosophy behind Hibernate

Hibernate was designed around **Java objects**, not database tables.

SQL thinks like this:

```text
Table
↓

Column
↓

Row
```

HQL thinks like this:

```text
Entity
↓

Property
↓

Object
```

---

SQL:

```sql
SELECT *
FROM student;
```

HQL:

```java
FROM Student
```

Notice the difference:

* SQL uses the table name: `student`
* HQL uses the entity name: `Student`

---

# If HQL supported everything, it would become SQL again

```text
HQL
↓

+ LIMIT

+ Stored procedures

+ Database functions

+ Vendor-specific features

+ Triggers

+ Window functions

↓

SQL
```

At that point, there would be no reason to have HQL.

---

# So Hibernate's developers made a design decision

```text
Simple operations?
↓

Use HQL.
```

```text
Need full database power?
↓

Use Native SQL.
```

They intentionally kept HQL smaller and database-independent and provided **Native SQL as an escape hatch** whenever developers needed direct access to the database.

This is why almost every ORM framework in the world provides **both** an object-oriented query language and a way to execute raw SQL.

---

# What is QBC (Query By Criteria) in Hibernate?

**QBC = Query By Criteria**

It is a **programmatic way of writing database queries**.

Instead of writing a query as a string:

```java
String hql = "from Student where course='Java'";
```

You build the query using Java objects:

```java
Criteria criteria = session.createCriteria(Student.class);
criteria.add(Restrictions.eq("course", "Java"));
```

---

# Three ways to query data in Hibernate

| Feature                              | HQL          | Native SQL   | QBC      |
| ------------------------------------ | ------------ | ------------ | -------- |
| Query language                       | Hibernate    | Database     | Java API |
| Database independent                 | ✅            | ❌            | ✅        |
| Uses SQL syntax                      | Similar      | Yes          | No       |
| Dynamic query building               | ⚠️ Difficult | ⚠️ Difficult | ✅ Easy   |
| Supports database-specific functions | ❌            | ✅            | ❌        |
| Type safety                          | ❌            | ❌            | Better   |

---

# Imagine a real-world search form

Suppose your website contains these optional fields:

```text
Student Name: ______

Course: ______

Minimum ID: ______
```

The user can fill:

* Only the name.
* Only the course.
* Both.
* All three.

---

# Using HQL

```java
String hql = "from Student where 1=1";

if (name != null) {
    hql += " and name=:name";
}

if (course != null) {
    hql += " and course=:course";
}

if (minId != 0) {
    hql += " and id>=:id";
}

Query query = session.createQuery(hql);

if (name != null) {
    query.setParameter("name", name);
}

if (course != null) {
    query.setParameter("course", course);
}

if (minId != 0) {
    query.setParameter("id", minId);
}

List<Student> students = query.list();
```

Notice something?

You are **building a query string manually**.

This becomes messy when there are many conditions.

---

# Using QBC

```java
Criteria criteria = session.createCriteria(Student.class);

if (name != null) {
    criteria.add(Restrictions.eq("name", name));
}

if (course != null) {
    criteria.add(Restrictions.eq("course", course));
}

if (minId != 0) {
    criteria.add(Restrictions.ge("id", minId));
}

List<Student> students = criteria.list();
```

No string concatenation.

No manual query building.

This is why QBC was created.

---

# Full Example

---

## Folder Structure

```text
src
│
├── Student.java
├── HibernateUtil.java
├── App.java
│
resources
│
└── hibernate.cfg.xml
```

---

# Student.java

```java
package com.hibernateexample;

import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "student")
public class Student {

    @Id
    private int id;

    private String name;

    private String course;

    public Student() {
    }

    public Student(int id, String name, String course) {
        this.id = id;
        this.name = name;
        this.course = course;
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

    public String getCourse() {
        return course;
    }

    public void setCourse(String course) {
        this.course = course;
    }
}
```

---

# HibernateUtil.java

```java
package com.hibernateexample;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {

    private static SessionFactory factory;

    static {

        factory = new Configuration()
                .configure("hibernate.cfg.xml")
                .addAnnotatedClass(Student.class)
                .buildSessionFactory();
    }

    public static SessionFactory getSessionFactory() {
        return factory;
    }
}
```

---

# hibernate.cfg.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE hibernate-configuration PUBLIC
"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/college
        </property>

        <property name="hibernate.connection.username">
            root
        </property>

        <property name="hibernate.connection.password">
            root
        </property>

        <property name="hibernate.dialect">
            org.hibernate.dialect.MySQL8Dialect
        </property>

        <property name="hibernate.show_sql">
            true
        </property>

        <property name="hibernate.hbm2ddl.auto">
            update
        </property>

    </session-factory>

</hibernate-configuration>
```

---

# Insert Data

```java
Session session =
        HibernateUtil.getSessionFactory().openSession();

Transaction tx = session.beginTransaction();

session.save(new Student(1, "Jack", "Java"));
session.save(new Student(2, "Ahmed", "Python"));
session.save(new Student(3, "John", "Java"));
session.save(new Student(4, "Sara", "Java"));

tx.commit();

session.close();
```

---

# QBC Example

```java
Session session =
        HibernateUtil.getSessionFactory().openSession();

Criteria criteria =
        session.createCriteria(Student.class);

criteria.add(Restrictions.eq("course", "Java"));

List<Student> students = criteria.list();

for (Student student : students) {

    System.out.println(student.getId());

    System.out.println(student.getName());

    System.out.println(student.getCourse());

    System.out.println("----------------");
}

session.close();
```

---

# Restrictions in QBC

## Equal

```java
criteria.add(
    Restrictions.eq("course", "Java")
);
```

Equivalent HQL:

```java
from Student where course='Java'
```

---

## Greater Than

```java
criteria.add(
    Restrictions.gt("id", 2)
);
```

Equivalent HQL:

```java
from Student where id>2
```

---

## Greater Than or Equal

```java
criteria.add(
    Restrictions.ge("id", 2)
);
```

Equivalent HQL:

```java
from Student where id>=2
```

---

## Less Than

```java
criteria.add(
    Restrictions.lt("id", 3)
);
```

Equivalent HQL:

```java
from Student where id<3
```

---

## Like

```java
criteria.add(
    Restrictions.like("name", "A%")
);
```

Equivalent HQL:

```java
from Student where name like 'A%'
```

---

## AND

```java
criteria.add(
    Restrictions.and(
        Restrictions.eq("course", "Java"),
        Restrictions.gt("id", 1)
    )
);
```

Equivalent HQL:

```java
from Student
where course='Java'
and id>1
```

---

## OR

```java
criteria.add(
    Restrictions.or(
        Restrictions.eq("course", "Java"),
        Restrictions.eq("course", "Python")
    )
);
```

Equivalent HQL:

```java
from Student
where course='Java'
or course='Python'
```

---

# Then why do we still have HQL?

Because HQL is simpler.

```java
from Student where course='Java'
```

is easier to read than

```java
criteria.add(
    Restrictions.eq("course", "Java")
);
```

---

# Then why do we still have Native SQL?

Because Hibernate cannot do everything.

Suppose you want a MySQL-specific function:

```sql
SELECT YEAR(NOW());
```

HQL cannot use many database-specific features.

Native SQL can.

```java
SQLQuery query =
        session.createSQLQuery(
                "SELECT YEAR(NOW())");

List list = query.list();
```

---

# Why isn't everyone using QBC?

Because **the old Hibernate Criteria API (QBC) was deprecated**.

Modern applications use:

* JPQL
* HQL
* JPA Criteria API

The old `createCriteria()` approach is mostly taught to help students understand how dynamic queries work.

---

# A simple rule for first-time learners

```text
Simple query?
↓
Use HQL.

Need database-specific SQL?
↓
Use Native SQL.

Need many optional search filters?
↓
Use QBC.
```

Think of it like this:

```text
HQL = Writing a sentence.

Native SQL = Speaking directly to the database.

QBC = Building a query with Java blocks.
```

