# Hibernate Operations

1. `persist()` → Insert a new record.
2. `find()` → Find a record by its primary key.
3. `get()` → Get a record by its primary key.
4. `merge()` → Update an existing record.
5. `remove()` → Delete a record.

---

# Project Structure

```text
HibernateCRUD
│
├── pom.xml
│
└── src
    └── main
        ├── java
        │
        │── com.example
        │    │
        │    ├── Student.java
        │    ├── HibernateUtil.java
        │    ├── PersistDemo.java
        │    ├── FindDemo.java
        │    ├── GetDemo.java
        │    ├── MergeDemo.java
        │    └── RemoveDemo.java
        │
        └── resources
             │
             └── hibernate.cfg.xml
```

---

# Step 1: Create Database

```sql
CREATE DATABASE hibernate_db;

USE hibernate_db;

CREATE TABLE student
(
    id INT PRIMARY KEY,
    name VARCHAR(100),
    course VARCHAR(100)
);
```

---

# Step 2: Add Dependencies (`pom.xml`)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>

    <artifactId>HibernateCRUD</artifactId>

    <version>1.0</version>

    <properties>

        <maven.compiler.source>21</maven.compiler.source>

        <maven.compiler.target>21</maven.compiler.target>

    </properties>

    <dependencies>

        <dependency>

            <groupId>org.hibernate.orm</groupId>

            <artifactId>hibernate-core</artifactId>

            <version>6.6.0.Final</version>

        </dependency>

        <dependency>

            <groupId>com.mysql</groupId>

            <artifactId>mysql-connector-j</artifactId>

            <version>9.0.0</version>

        </dependency>

        <dependency>

            <groupId>jakarta.persistence</groupId>

            <artifactId>jakarta.persistence-api</artifactId>

            <version>3.1.0</version>

        </dependency>

    </dependencies>

</project>
```

---

# Step 3: Create `hibernate.cfg.xml`

```xml
<?xml version='1.0' encoding='utf-8'?>

<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "https://hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/hibernate_db
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

        <property name="hibernate.format_sql">
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

# Step 4: Create Entity Class (`Student.java`)

```java
package com.example;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "student")
public class Student
{
    @Id
    private int id;

    private String name;

    private String course;

    public Student()
    {
    }

    public Student(int id, String name, String course)
    {
        this.id = id;
        this.name = name;
        this.course = course;
    }

    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    public String getCourse()
    {
        return course;
    }

    public void setCourse(String course)
    {
        this.course = course;
    }

    @Override
    public String toString()
    {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", course='" + course + '\'' +
                '}';
    }
}
```

---

# Step 5: Create `HibernateUtil.java`

```java
package com.example;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil
{
    private static final SessionFactory sessionFactory =
            new Configuration()
                    .configure()
                    .buildSessionFactory();

    public static SessionFactory getSessionFactory()
    {
        return sessionFactory;
    }
}
```

---

# 1. `persist()` (INSERT)

## PersistDemo.java

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class PersistDemo
{
    public static void main(String[] args)
    {
        Session session =
                HibernateUtil
                        .getSessionFactory()
                        .openSession();

        Transaction transaction =
                session.beginTransaction();

        Student student =
                new Student(
                        1,
                        "john",
                        "Java"
                );

        session.persist(student);

        transaction.commit();

        session.close();

        System.out.println("Data inserted.");
    }
}
```

---

## SQL Generated

```sql
insert into student
(id, course, name)
values
(1, 'Java', 'john');
```

---

# 2. `find()` (READ)

## FindDemo.java

```java
package com.example;

import org.hibernate.Session;

public class FindDemo
{
    public static void main(String[] args)
    {
        Session session =
                HibernateUtil
                        .getSessionFactory()
                        .openSession();

        Student student =
                session.find(
                        Student.class,
                        1
                );

        System.out.println(student);

        session.close();
    }
}
```

---

## SQL Generated

```sql
select *
from student
where id = 1;
```

---

# 3. `get()` (READ)

## GetDemo.java

```java
package com.example;

import org.hibernate.Session;

public class GetDemo
{
    public static void main(String[] args)
    {
        Session session =
                HibernateUtil
                        .getSessionFactory()
                        .openSession();

        Student student =
                session.get(
                        Student.class,
                        1
                );

        System.out.println(student);

        session.close();
    }
}
```

---

# Difference Between `find()` and `get()`

| find()            | get()              |
| ----------------- | ------------------ |
| JPA method        | Hibernate method   |
| Returns an entity | Returns an entity  |
| Standard method   | Hibernate-specific |
| Portable          | Not portable       |

---

# 4. `merge()` (UPDATE)

## MergeDemo.java

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class MergeDemo
{
    public static void main(String[] args)
    {
        Session session =
                HibernateUtil
                        .getSessionFactory()
                        .openSession();

        Transaction transaction =
                session.beginTransaction();

        Student student =
                new Student(
                        1,
                        "john cena",
                        "Advanced Java"
                );

        session.merge(student);

        transaction.commit();

        session.close();

        System.out.println("Data updated.");
    }
}
```

---

## SQL Generated

```sql
update student
set
course = 'Advanced Java',
name = 'john cena'
where id = 1;
```

---

# 5. `remove()` (DELETE)

## RemoveDemo.java

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class RemoveDemo
{
    public static void main(String[] args)
    {
        Session session =
                HibernateUtil
                        .getSessionFactory()
                        .openSession();

        Transaction transaction =
                session.beginTransaction();

        Student student =
                session.find(
                        Student.class,
                        1
                );

        session.remove(student);

        transaction.commit();

        session.close();

        System.out.println("Data deleted.");
    }
}
```

---

## SQL Generated

```sql
delete
from student
where id = 1;
```

---

# Complete CRUD Flow

```text
Create Object
       ↓

Student student =
new Student(1,"john","Java");

       ↓

persist()
       ↓

Database
       ↓

find() or get()
       ↓

merge()
       ↓

remove()
```

---

# Remember This Shortcut

```text
persist() → INSERT

find() → SELECT

get() → SELECT

merge() → UPDATE

remove() → DELETE
```

---

# One Important Thing

`persist()`, `merge()`, and `remove()` **change the database**, so they need a transaction.

```java
Transaction tx =
session.beginTransaction();

session.persist(student);

tx.commit();
```

`find()` and `get()` only read data.



No transaction is required for simple read operations.
