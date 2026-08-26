## 1. First: What is caching?

Imagine you have a restaurant.

Every time a customer asks:

> "Give me a glass of water."

The waiter could go to the kitchen every single time and bring water.

But if the waiter already has water nearby, why go to the kitchen again?

That nearby water is like a **cache**.

### Simple definition

**Caching means temporarily storing frequently used data somewhere faster so that we don't have to fetch it again from the slower original source.**

The idea is:

```text
Without Cache

Application
     ↓
Database
     ↓
Data
```

Every time we need data → go to database.

With caching:

```text
First Request

Application
     ↓
   Cache ❌
     ↓
 Database
     ↓
   Data
     ↓
   Cache


Second Request

Application
     ↓
   Cache ✅
     ↓
   Data
```

So the second request can be much faster.

---

# 2. Then what is Hibernate Caching?

Now connect this with Hibernate.

Hibernate sits between our Java application and the database.

Normally:

```text
Java Application
       ↓
    Hibernate
       ↓
    Database
```

Suppose we have:

```text
Student ID = 101
Name = Rahul
Course = Java
```

The first time we ask Hibernate:

```java
Student student = session.get(Student.class, 101);
```

Hibernate may need to go to the database:

```text
Application
     ↓
 Hibernate
     ↓
 Database
     ↓
 Student 101
```

But what if we ask for **Student 101 again**?

Instead of always going to the database, Hibernate can keep previously loaded data in its **cache**.

So:

```text
Application
     ↓
  Hibernate
     ↓
   Cache
     ↓
Student 101
```

That's the basic idea of **Hibernate caching**.

### In one sentence:

> **Hibernate caching is the mechanism by which Hibernate stores previously fetched data so that it can reuse that data instead of repeatedly accessing the database.**

---

# 3. Why do we need it?

Because **database access is relatively expensive**.

Imagine an application receives:

```text
10,000 requests
```

And all 10,000 requests ask:

```text
Give me course information for Java
```

Without caching:

```text
10,000 requests
       ↓
10,000 database calls
```

With caching:

```text
First request
      ↓
 Database
      ↓
 Cache

Next 9,999 requests
      ↓
 Cache
```

Obviously, this can reduce database load and improve response time.

---

# 4. Real-world example

Think about an **e-commerce application**.

Suppose 1 lakh people are visiting a product page:

```text
iPhone 17
Price: ₹79,999
Brand: Apple
Rating: 4.6
```

The information doesn't necessarily change every second.

If every visitor causes a database query:

```text
User 1 → DB
User 2 → DB
User 3 → DB
User 4 → DB
...
User 100000 → DB
```

That's unnecessary pressure on the database.

Instead:

```text
User 1
  ↓
Database
  ↓
Cache

User 2 ──→ Cache
User 3 ──→ Cache
User 4 ──→ Cache
...
User 100000 ─→ Cache
```

This is where caching becomes extremely useful.

---

# 5. Is Hibernate caching the same as normal caching?

**The basic idea is the same.**

Both are trying to solve the same problem:

> **Don't repeatedly fetch something expensive when we already have it available somewhere faster.**

But **Hibernate caching is specifically integrated with Hibernate's ORM mechanism and entities/database access.**

General caching can be used for many things:

```text
API responses
Images
Web pages
Configuration
User sessions
Database results
Objects
etc.
```

Hibernate caching is particularly concerned with data that Hibernate manages.

For example:

```java
Student
Employee
Product
Customer
Order
```

---

# 6. One very important distinction

Students often think:

> "Cache means database is replaced."

❌ No.

The database is still the **main source of persistent data**.

Cache is more like a **temporary shortcut**.

Think:

```text
Database = Main Store
Cache    = Nearby Store
```

If the nearby store has what you need → use it.

If not → go to the main store.

---

# 7. Where does Hibernate caching exist?

Don't go deep into this yet. Just tell students that Hibernate provides **different levels/mechanisms of caching**.

At a very high level, they will encounter:

```text
Hibernate Caching
       │
       ├── First-Level Cache
       │
       ├── Second-Level Cache
       │
       └── Query Cache
```

**Don't explain these yet.**

Just tell them:

> "Hibernate has different caching mechanisms. We will understand each one separately after we first understand the basic caching concept."

That keeps the mental model clean.

---

# 8. The complete picture

For a beginner, I'd leave them with this picture:

```text
             JAVA APPLICATION
                    │
                    ↓
                HIBERNATE
                    │
              ┌─────┴─────┐
              ↓           ↓
            CACHE       DATABASE
              │
              ↓
        Previously used
             data
```

And the fundamental idea is:

```text
CACHE = Faster temporary access
DATABASE = Permanent source of data
```

### The key question caching answers:

> **"Do I really need to go to the database again, or do I already have this data somewhere faster?"**
---
**Hibernate First-Level Cache** 

# 1. Create Database in MySQL

First create a database:

```sql
CREATE DATABASE hibernate_demo;

USE hibernate_demo;
```

Create the `student` table:

```sql
CREATE TABLE student (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    city VARCHAR(100)
);
```

Insert some data:

```sql
INSERT INTO student (id, name, city)
VALUES
(12424, 'Rahul', 'Lucknow'),
(12425, 'Aman', 'Delhi'),
(12426, 'Priya', 'Mumbai');
```

Check:

```sql
SELECT * FROM student;
```

You should get something like:

|    id | name  | city    |
| ----: | ----- | ------- |
| 12424 | Rahul | Lucknow |
| 12425 | Aman  | Delhi   |
| 12426 | Priya | Mumbai  |

---

# 2. Create Maven Project

In Eclipse:

**File → New → Maven Project**

For example:

```text
Project Name: HibernateFirstLevelCache
```

Structure:

```text
HibernateFirstLevelCache
│
├── src/main/java
│   └── com.example
│       ├── Student.java
│       └── FirstDemo.java
│
├── src/main/resources
│   └── hibernate.cfg.xml
│
└── pom.xml
```

---

# 3. `pom.xml`

Add Hibernate and MySQL dependencies.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>HibernateFirstLevelCache</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <dependencies>

        <!-- Hibernate ORM -->
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>6.6.1.Final</version>
        </dependency>

        <!-- MySQL Driver -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>9.0.0</version>
        </dependency>

        <!-- SLF4J logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>2.0.16</version>
        </dependency>

    </dependencies>

</project>
```

After saving:

**Right Click Project → Maven → Update Project**

---

# 4. Create `Student.java`

Create:

```text
src/main/java
    └── com.example
        └── Student.java
```

Code:

```java
package com.example;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "student")
public class Student {

    @Id
    private int id;

    private String name;

    private String city;

    public Student() {
    }

    public Student(int id, String name, String city) {
        this.id = id;
        this.name = name;
        this.city = city;
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

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", city='" + city + '\'' +
                '}';
    }
}
```

---

# 5. Create `hibernate.cfg.xml`

Create this inside:

```text
src/main/resources
```

File:

```text
hibernate.cfg.xml
```

Code:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "https://hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <!-- Database Connection -->
        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/hibernate_demo
        </property>

        <property name="hibernate.connection.username">
            root
        </property>

        <property name="hibernate.connection.password">
            YOUR_PASSWORD
        </property>

        <!-- Hibernate Dialect -->
        <property name="hibernate.dialect">
            org.hibernate.dialect.MySQLDialect
        </property>

        <!-- Show SQL -->
        <property name="hibernate.show_sql">
            true
        </property>

        <!-- Format SQL -->
        <property name="hibernate.format_sql">
            true
        </property>

        <!-- Entity -->
        <mapping class="com.example.Student"/>

    </session-factory>

</hibernate-configuration>
```

Replace:

```text
YOUR_PASSWORD
```

with your MySQL password.

---

# 6. First Basic Hibernate Program

Before demonstrating cache, understand this:

```java
SessionFactory factory =
        new Configuration()
        .configure()
        .buildSessionFactory();
```

This creates the:

```text
SessionFactory
```

Then:

```java
Session session = factory.openSession();
```

creates a:

```text
Session
```

Think:

```text
Configuration
      ↓
SessionFactory
      ↓
Session
      ↓
Database
```

---

# 7. Complete `FirstDemo.java`

Now create:

```text
FirstDemo.java
```

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class FirstDemo {

    public static void main(String[] args) {

        // Step 1: Create SessionFactory
        SessionFactory factory =
                new Configuration()
                        .configure()
                        .buildSessionFactory();

        // Step 2: Open Session
        Session session = factory.openSession();

        // Step 3: Fetch Student
        Student student =
                session.get(Student.class, 12424);

        // Step 4: Print Student
        System.out.println(student);

        // Step 5: Some other work
        System.out.println("Working something...");

        // Step 6: Fetch SAME Student again
        Student student1 =
                session.get(Student.class, 12424);

        // Step 7: Print Student
        System.out.println(student1);

        // Step 8: Check whether both references point
        // to the same object
        System.out.println(student == student1);

        // Step 9: Check whether object is present
        // inside the first-level cache
        System.out.println(session.contains(student1));

        // Step 10: Close Session
        session.close();

        // Step 11: Close SessionFactory
        factory.close();
    }
}
```

---

# 8. What Actually Happens?

This is the **most important part**.

When we execute:

```java
Student student =
        session.get(Student.class, 12424);
```

Hibernate checks:

```text
           session.get()
                ↓
     Is Student#12424
     already in Session?
          ↙          ↘
        YES           NO
         ↓             ↓
   Return object    Query Database
                       ↓
                 Create object
                       ↓
                 Store in cache
                       ↓
                 Return object
```

So the first call generates SQL similar to:

```sql
SELECT
    s1_0.id,
    s1_0.city,
    s1_0.name
FROM student s1_0
WHERE s1_0.id = ?
```

Hibernate gets:

```text
12424
Rahul
Lucknow
```

and creates a Java object:

```text
Student object
       ↓
id = 12424
name = Rahul
city = Lucknow
```

That object is placed inside the **First-Level Cache**.

---

# 9. Then We Call `get()` Again

Now:

```java
Student student1 =
        session.get(Student.class, 12424);
```

Many beginners think:

> "Hibernate will execute another SELECT."

But **not necessarily**.

Hibernate first checks the current `Session`.

It finds:

```text
Student#12424
```

already present.

Therefore:

```text
Database
   ↑
   X
   │
Session First-Level Cache
   │
   ↓
student1
```

No second SQL query is required.

---

# 10. The Most Interesting Experiment

Add:

```java
System.out.println(student == student1);
```

Output:

```text
true
```

Why?

Because both variables refer to the **same Java object**.

Conceptually:

```text
student ───────┐
               ↓
        ┌───────────────┐
        │ Student       │
        │ id = 12424    │
        │ name = Rahul  │
        │ city = Lucknow │
        └───────────────┘
               ↑
               │
student1 ──────┘
```

They are not two different objects.

---

# 11. `session.contains()`

This line from your screenshot is also important:

```java
System.out.println(session.contains(student1));
```

Output:

```text
true
```

Because:

```java
student1
```

is currently associated with the Hibernate `Session`.

In simple terms:

> "Hibernate's Session knows about this object."

---

# 12. Complete Execution Flow

You can explain the entire program to students like this:

```text
START
  │
  ↓
Create Configuration
  │
  ↓
Create SessionFactory
  │
  ↓
Open Session
  │
  ↓
session.get(Student.class, 12424)
  │
  ↓
Check First-Level Cache
  │
  ├── Object NOT FOUND
  │        ↓
  │    Execute SELECT
  │        ↓
  │    Create Student object
  │        ↓
  │    Put object in Session Cache
  │
  ↓
Return Student object
  │
  ↓
Do some other work
  │
  ↓
session.get(Student.class, 12424)
  │
  ↓
Check First-Level Cache
  │
  ├── Object FOUND
  │        ↓
  │    Don't execute SELECT
  │        ↓
  │    Return existing object
  │
  ↓
student == student1
  │
  ↓
true
  │
  ↓
session.close()
  │
  ↓
First-Level Cache destroyed
```

---

# 13. The Most Important Rule

### First-Level Cache belongs to `Session`.

Not:

```text
SessionFactory
```

Not:

```text
Database
```

Not:

```text
Application
```

It belongs to:

```text
Session
```

Therefore:

```java
Session session1 = factory.openSession();

Student s1 =
        session1.get(Student.class, 12424);

Student s2 =
        session1.get(Student.class, 12424);
```

Usually:

```text
1 SELECT
```

because both operations use the **same Session**.

But:

```java
Session session1 = factory.openSession();

Student s1 =
        session1.get(Student.class, 12424);

session1.close();


Session session2 = factory.openSession();

Student s2 =
        session2.get(Student.class, 12424);
```

Now Hibernate needs to query again.

```text
Session 1
   │
   └── First-Level Cache
          └── Student#12424

session1.close()
   ↓
Cache gone


Session 2
   │
   └── Empty First-Level Cache
          ↓
       SELECT
```

That's why we say:

> **First-Level Cache is enabled by default and is associated with a Hibernate Session.**

---

# 14. One Excellent Demo for Students

Use this version in class:

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class FirstDemo {

    public static void main(String[] args) {

        SessionFactory factory =
                new Configuration()
                        .configure()
                        .buildSessionFactory();

        Session session = factory.openSession();

        System.out.println("----- FIRST GET -----");

        Student student =
                session.get(Student.class, 12424);

        System.out.println(student);

        System.out.println("----- SECOND GET -----");

        Student student1 =
                session.get(Student.class, 12424);

        System.out.println(student1);

        System.out.println("----- COMPARISON -----");

        System.out.println(
                "Same Object? " + (student == student1)
        );

        System.out.println(
                "Student in Session? " +
                session.contains(student)
        );

        session.close();
        factory.close();
    }
}
```

With SQL logging enabled, students should observe:

```text
----- FIRST GET -----

SELECT ...
FROM student
WHERE id = ?

Student{id=12424, name='Rahul', city='Lucknow'}

----- SECOND GET -----

Student{id=12424, name='Rahul', city='Lucknow'}

----- COMPARISON -----

Same Object? true
Student in Session? true
```

This is a **very important distinction**: L1 and L2 cache are not two fixed-size memory areas like "L1 = 10 MB, L2 = 100 MB." Their memory usage depends on configuration and the cache provider.

Exactly — **this is the reason Second-Level Cache exists.** Your understanding of First-Level Cache leads directly to it.

### The problem with First-Level Cache

Suppose your application has:

```text
Session 1
   │
   └── First-Level Cache
          └── Student#12424


Session 2
   │
   └── First-Level Cache
          └── Student#12424


Session 3
   │
   └── First-Level Cache
          └── Student#12424
```

Each Session has its **own separate cache**.

So imagine:

```java
// Session 1
Student s1 = session1.get(Student.class, 12424);
session1.close();
```

Hibernate loads:

```text
Database
   ↓
Student#12424
   ↓
Session 1 First-Level Cache
```

Then Session 1 closes:

```text
Session 1
   ↓
CLOSE
   ↓
First-Level Cache destroyed
```

Now:

```java
// Session 2
Student s2 = session2.get(Student.class, 12424);
```

Hibernate doesn't have that student in Session 2's First-Level Cache.

So:

```text
Session 2
   ↓
First-Level Cache
   ↓
NOT FOUND
   ↓
Database
   ↓
SELECT
```

### Now imagine 1,000 users

This is where it becomes interesting.

```text
             Database
                ↑
       ┌────────┼────────┐
       │        │        │
   Session 1 Session 2 Session 3
       │        │        │
    Cache    Cache     Cache
```

Every Session has its own cache.

You could potentially keep asking the database for the **same Student/Product/Department**.

---

# Second-Level Cache solves this problem

Second-Level Cache belongs to the **SessionFactory**, not an individual Session.

Think:

```text
                  SessionFactory
                       │
              ┌────────┴────────┐
              │                 │
       Second-Level Cache       │
              │                 │
       Student#12424            │
              │                 │
       ┌──────┼──────┐          │
       ↓      ↓      ↓          ↓
   Session1 Session2 Session3 ...
   L1 Cache L1 Cache L1 Cache
```

Now:

```text
Session 1
    ↓
L1 Cache → MISS
    ↓
L2 Cache → MISS
    ↓
Database
    ↓
L2 Cache
    ↓
L1 Cache
```

Later:

```text
Session 2
    ↓
L1 Cache → MISS
    ↓
L2 Cache → HIT ✅
    ↓
No Database Query
```

That's the **main reason Second-Level Cache exists**.

---

# The hierarchy becomes

When Hibernate needs an entity:

```text
             session.get()
                   │
                   ↓
        ┌────────────────────┐
        │ First-Level Cache  │
        │     (Session)      │
        └─────────┬──────────┘
                  │
             NOT FOUND
                  ↓
        ┌────────────────────┐
        │ Second-Level Cache │
        │  (SessionFactory)  │
        └─────────┬──────────┘
                  │
             NOT FOUND
                  ↓
        ┌────────────────────┐
        │     Database       │
        └────────────────────┘
```

So you can remember:

> **L1 Cache = one Session**

> **L2 Cache = shared across Sessions belonging to the same SessionFactory**

---

## Real-world example

Imagine an e-commerce application.

A product:

```text
iPhone 17
₹79,999
Apple
```

might be requested thousands of times.

User 1:

```text
Session 1 → Product 101
```

User 2:

```text
Session 2 → Product 101
```

User 3:

```text
Session 3 → Product 101
```

Without L2:

```text
User 1 → DB
User 2 → DB
User 3 → DB
User 4 → DB
...
```

With L2:

```text
User 1
  ↓
L1 MISS
  ↓
L2 MISS
  ↓
DB
  ↓
L2 stores Product 101


User 2
  ↓
L1 MISS
  ↓
L2 HIT ✅
  ↓
No DB


User 3
  ↓
L1 MISS
  ↓
L2 HIT ✅
  ↓
No DB
```

This can significantly reduce repeated database reads for suitable data.

---

# But there is an important catch

**Second-Level Cache is not automatically enabled just because Hibernate has it.**

First-Level Cache:

```text
✅ Enabled by default
✅ Associated with Session
```

Second-Level Cache:

```text
❌ Not simply automatic by default
✅ Must be configured
✅ Requires a cache provider/implementation
```

Common providers/technologies include things such as **Ehcache, Infinispan, or other JCache-compatible implementations**, depending on your Hibernate version and application architecture.

---

# One more important difference

Suppose:

```java
Session session1 = factory.openSession();

Student s1 =
    session1.get(Student.class, 12424);
```

Then:

```java
session1.close();
```

L1 disappears.

But if L2 is configured:

```text
Session 1
   ↓
L1
   ↓
close()
   ↓
L1 disappears ❌

       BUT

L2
   ↓
Student#12424
   ↓
Still available ✅
```

Then a new Session can potentially reuse it:

```java
Session session2 = factory.openSession();

Student s2 =
    session2.get(Student.class, 12424);
```

---

## 🧠 The easiest way to teach this

Tell students to imagine **three levels**:

```text
                APPLICATION
                     │
          ┌──────────┴──────────┐
          │                     │
      Session 1             Session 2
          │                     │
       L1 Cache              L1 Cache
          │                     │
          └──────────┬──────────┘
                     │
                L2 Cache
                     │
                     ↓
                 Database
```

And the golden rule:

| Cache            | Belongs to       | Scope              | Default         |
| ---------------- | ---------------- | ------------------ | --------------- |
| **First-Level**  | `Session`        | One Session        | ✅ Yes           |
| **Second-Level** | `SessionFactory` | Multiple Sessions  | ⚙️ Configure it |
| Database         | Database         | Entire application | —               |

So the **business reason** for L2 is simply:

> **L1 prevents repeated DB calls within the same Session. L2 prevents repeated DB calls across different Sessions.**

That's the conceptual bridge you should make before teaching the actual L2 configuration.

# 1. What we are going to build

Our final project:

```text
HibernateL2Cache
│
├── pom.xml
│
├── src/main/java
│   └── com.example
│       ├── Student.java
│       └── SecondLevelCacheDemo.java
│
└── src/main/resources
    ├── hibernate.cfg.xml
    └── ehcache.xml
```

Architecture:

```text
                    Java Application
                           │
                           ↓
                    SessionFactory
                           │
                 ┌─────────┴─────────┐
                 │                   │
             Session 1           Session 2
                 │                   │
              L1 Cache             L1 Cache
                 │                   │
                 └─────────┬─────────┘
                           ↓
                    ┌──────────────┐
                    │ L2 Cache     │
                    │  Ehcache     │
                    └──────┬───────┘
                           │
                      Cache MISS
                           ↓
                       MySQL DB
```

---

# 2. Create Database

Open MySQL.

```sql
CREATE DATABASE hibernate_l2_demo;

USE hibernate_l2_demo;
```

Create table:

```sql
CREATE TABLE student (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    city VARCHAR(100)
);
```

Insert data:

```sql
INSERT INTO student (id, name, city)
VALUES
(12424, 'Rahul', 'Lucknow'),
(12425, 'Aman', 'Delhi'),
(12426, 'Priya', 'Mumbai');
```

Check:

```sql
SELECT * FROM student;
```

You should see:

```text
12424 | Rahul | Lucknow
12425 | Aman  | Delhi
12426 | Priya | Mumbai
```

---

# 3. Create Maven Project

Create:

```text
HibernateL2Cache
```

with Maven.

Your structure should eventually look like:

```text
HibernateL2Cache
│
├── src/main/java
│   └── com.example
│
├── src/main/resources
│
└── pom.xml
```

---

# 4. `pom.xml`

For the demonstration, we'll use Hibernate **6.6.55.Final**, which is documented as a Hibernate 6.6 release compatible with Java 11/17/21. Hibernate recommends keeping its artifacts aligned through the Hibernate platform. ([Hibernate Documentation][2])

Use:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>HibernateL2Cache</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>

        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>

        <hibernate.version>6.6.55.Final</hibernate.version>

    </properties>

    <dependencies>

        <!-- Hibernate Core -->
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>

        <!-- Hibernate JCache Integration -->
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-jcache</artifactId>
            <version>${hibernate.version}</version>
        </dependency>

        <!-- Ehcache -->
        <dependency>
            <groupId>org.ehcache</groupId>
            <artifactId>ehcache</artifactId>
            <version>3.10.8</version>
        </dependency>

        <!-- MySQL -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>9.0.0</version>
        </dependency>

        <!-- Logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>2.0.16</version>
        </dependency>

    </dependencies>

</project>
```

`hibernate-jcache` is the Hibernate integration layer; Ehcache provides the actual cache implementation. ([Maven Central][3])

**Important:** Ehcache 3.10.8 is an older but commonly used JCache implementation. Its published artifact uses JCache 1.1.0. ([Maven Central][4])

After saving:

```text
Right Click Project
        ↓
Maven
        ↓
Update Project
```

---

# 5. Create `Student.java`

Location:

```text
src/main/java/com/example/Student.java
```

Code:

```java
package com.example;

import jakarta.persistence.Cacheable;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

import org.hibernate.annotations.Cache;
import org.hibernate.annotations.CacheConcurrencyStrategy;

@Entity
@Table(name = "student")

@Cacheable
@Cache(
    usage = CacheConcurrencyStrategy.READ_WRITE
)
public class Student {

    @Id
    private int id;

    private String name;

    private String city;

    public Student() {
    }

    public Student(int id, String name, String city) {
        this.id = id;
        this.name = name;
        this.city = city;
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

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    @Override
    public String toString() {

        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", city='" + city + '\'' +
                '}';
    }
}
```

---

# 6. Understand these annotations

This is the most important part.

### `@Cacheable`

```java
@Cacheable
```

means:

> This entity is eligible to be stored in the second-level cache.

But simply putting this annotation doesn't magically create a cache provider.

We also need to configure Hibernate's cache infrastructure.

---

### `@Cache`

```java
@Cache(
    usage = CacheConcurrencyStrategy.READ_WRITE
)
```

tells Hibernate how the entity should participate in the second-level cache.

For teaching purposes:

```text
READ_WRITE
```

is a good strategy to demonstrate because we also want to discuss consistency when entities change.

---

# 7. Create `ehcache.xml`

Now comes the **actual cache configuration**.

Create:

```text
src/main/resources/ehcache.xml
```

Use:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<config
    xmlns="http://www.ehcache.org/v3"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.ehcache.org/v3
        http://www.ehcache.org/schema/ehcache-core.xsd">

    <cache alias="com.example.Student">

        <key-type>java.lang.Object</key-type>

        <value-type>java.lang.Object</value-type>

        <resources>
            <heap unit="entries">1000</heap>
        </resources>

        <expiry>
            <ttl unit="minutes">30</ttl>
        </expiry>

    </cache>

</config>
```

This says approximately:

```text
Student Cache

Maximum:
1000 entries

Expiration:
30 minutes
```

So conceptually:

```text
L2 Cache
│
└── Student
     │
     ├── Student#12424
     ├── Student#12425
     ├── Student#12426
     └── ...
     
Maximum = 1000 entries
TTL = 30 minutes
```

---

# 8. Configure `hibernate.cfg.xml`

Create:

```text
src/main/resources/hibernate.cfg.xml
```

Complete configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "https://hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <!-- =============================== -->
        <!-- DATABASE CONFIGURATION          -->
        <!-- =============================== -->

        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/hibernate_l2_demo
        </property>

        <property name="hibernate.connection.username">
            root
        </property>

        <property name="hibernate.connection.password">
            YOUR_PASSWORD
        </property>


        <!-- =============================== -->
        <!-- HIBERNATE CONFIGURATION          -->
        <!-- =============================== -->

        <property name="hibernate.dialect">
            org.hibernate.dialect.MySQLDialect
        </property>

        <property name="hibernate.show_sql">
            true
        </property>

        <property name="hibernate.format_sql">
            true
        </property>


        <!-- =============================== -->
        <!-- SECOND LEVEL CACHE               -->
        <!-- =============================== -->

        <property name="hibernate.cache.use_second_level_cache">
            true
        </property>

        <property name="hibernate.cache.region.factory_class">
            jcache
        </property>

        <property name="hibernate.javax.cache.uri">
            ehcache.xml
        </property>


        <!-- =============================== -->
        <!-- CACHE STATISTICS                  -->
        <!-- =============================== -->

        <property name="hibernate.generate_statistics">
            true
        </property>


        <!-- =============================== -->
        <!-- ENTITY MAPPING                    -->
        <!-- =============================== -->

        <mapping class="com.example.Student"/>

    </session-factory>

</hibernate-configuration>
```

Hibernate's 6.6 documentation shows `hibernate.cache.region.factory_class=jcache` and a JCache URI as the basic JCache configuration pattern. ([Hibernate Documentation][5])

---

# 9. One important correction about the XML

You may see tutorials using:

```xml
hibernate.javax.cache.uri
```

even though you're using modern Hibernate/Jakarta APIs.

This is because **JCache 1.x uses the `javax.cache` API**, while Hibernate's entity annotations are using `jakarta.persistence`.

Don't confuse these two:

```text
Hibernate/JPA entity API
        ↓
jakarta.persistence
```

versus:

```text
JCache API
        ↓
javax.cache
```

For this Hibernate 6.6 + JCache setup, that distinction is expected.

---

# 10. Now create the demo

Create:

```text
src/main/java/com/example/SecondLevelCacheDemo.java
```

Start with:

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class SecondLevelCacheDemo {

    public static void main(String[] args) {

        SessionFactory factory =
                new Configuration()
                        .configure()
                        .buildSessionFactory();


        // =====================================
        // SESSION 1
        // =====================================

        System.out.println("========== SESSION 1 ==========");

        Session session1 = factory.openSession();

        Student student1 =
                session1.get(Student.class, 12424);

        System.out.println(student1);

        session1.close();


        // =====================================
        // SESSION 2
        // =====================================

        System.out.println("\n========== SESSION 2 ==========");

        Session session2 = factory.openSession();

        Student student2 =
                session2.get(Student.class, 12424);

        System.out.println(student2);

        session2.close();


        // =====================================
        // CLOSE FACTORY
        // =====================================

        factory.close();
    }
}
```

---

# 11. What should happen?

Run it.

The interesting part is:

```java
Session session1 = factory.openSession();

Student student1 =
        session1.get(Student.class, 12424);

session1.close();
```

First request:

```text
L1 Cache
   ↓
MISS
   ↓
L2 Cache
   ↓
MISS
   ↓
DATABASE
   ↓
Student loaded
   ↓
L1 Cache
   ↓
L2 Cache
```

Therefore:

```sql
SELECT ...
FROM student
WHERE id = 12424
```

is executed.

---

# 12. Then Session 1 closes

```java
session1.close();
```

The First-Level Cache disappears:

```text
Session 1
   │
   └── L1
        └── Student#12424

session1.close()
       ↓
      💥
       ↓
L1 gone
```

But:

```text
L2 Cache
   │
   └── Student#12424
```

is still associated with the SessionFactory.

---

# 13. Session 2 starts

```java
Session session2 = factory.openSession();
```

Now:

```java
Student student2 =
        session2.get(Student.class, 12424);
```

Hibernate checks:

```text
Session 2
    ↓
L1 Cache
    ↓
MISS
    ↓
L2 Cache
    ↓
HIT ✅
    ↓
NO DATABASE QUERY
```

That's the entire demonstration of L2.

---

# 14. But let's prove it properly

We can use Hibernate statistics.

Modify your program:

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.hibernate.stat.Statistics;

public class SecondLevelCacheDemo {

    public static void main(String[] args) {

        SessionFactory factory =
                new Configuration()
                        .configure()
                        .buildSessionFactory();


        Statistics statistics =
                factory.getStatistics();


        statistics.clear();


        // =====================================
        // SESSION 1
        // =====================================

        System.out.println("========== SESSION 1 ==========");

        Session session1 = factory.openSession();

        Student student1 =
                session1.get(Student.class, 12424);

        System.out.println(student1);

        session1.close();


        System.out.println(
                "L2 Cache Puts = "
                + statistics.getSecondLevelCachePutCount()
        );

        System.out.println(
                "L2 Cache Hits = "
                + statistics.getSecondLevelCacheHitCount()
        );

        System.out.println(
                "L2 Cache Misses = "
                + statistics.getSecondLevelCacheMissCount()
        );


        // =====================================
        // SESSION 2
        // =====================================

        System.out.println("\n========== SESSION 2 ==========");

        Session session2 = factory.openSession();

        Student student2 =
                session2.get(Student.class, 12424);

        System.out.println(student2);

        session2.close();


        System.out.println(
                "L2 Cache Puts = "
                + statistics.getSecondLevelCachePutCount()
        );

        System.out.println(
                "L2 Cache Hits = "
                + statistics.getSecondLevelCacheHitCount()
        );

        System.out.println(
                "L2 Cache Misses = "
                + statistics.getSecondLevelCacheMissCount()
        );


        factory.close();
    }
}
```

---

# 15. Expected conceptual result

Something like:

```text
========== SESSION 1 ==========

SELECT ...
FROM student
WHERE id = 12424

Student{id=12424, name='Rahul', city='Lucknow'}

L2 Cache Puts = 1
L2 Cache Hits = 0
L2 Cache Misses = 1


========== SESSION 2 ==========

Student{id=12424, name='Rahul', city='Lucknow'}

L2 Cache Puts = 1
L2 Cache Hits = 1
L2 Cache Misses = 1
```

The exact statistics/output can vary slightly depending on Hibernate/provider behavior, but the key observation is:

```text
SESSION 1

L1 MISS
L2 MISS
DB HIT
       ↓
L2 populated


SESSION 2

L1 MISS
L2 HIT
       ↓
NO DB
```

---

# 16. Now demonstrate L1 + L2 together

This is an even better classroom demo.

```java
Session session1 = factory.openSession();

Student s1 =
        session1.get(Student.class, 12424);

Student s2 =
        session1.get(Student.class, 12424);

session1.close();


Session session2 = factory.openSession();

Student s3 =
        session2.get(Student.class, 12424);

session2.close();
```

Think carefully.

### First `get()`

```text
L1 → MISS
L2 → MISS
DB → HIT
```

### Second `get()` in same Session

```text
L1 → HIT
```

No L2 check is needed.

### Third `get()` in new Session

```text
L1 → MISS
L2 → HIT
```

No database.

So:

```text
                    get()
                      │
                      ↓
                 ┌─────────┐
                 │ L1      │
                 │ Session │
                 └────┬────┘
                      │
                 MISS │
                      ↓
                 ┌─────────┐
                 │ L2      │
                 │ Shared  │
                 └────┬────┘
                      │
                 MISS │
                      ↓
                 ┌─────────┐
                 │   DB    │
                 └─────────┘
```

---

# 17. Very important: `student1 == student2`

Don't use this as your proof that L2 is working.

With L1:

```java
Student s1 =
    session.get(Student.class, 12424);

Student s2 =
    session.get(Student.class, 12424);

System.out.println(s1 == s2);
```

will generally be:

```text
true
```

because the same Session returns the same managed Java instance.

But with two Sessions:

```java
Student s1 =
    session1.get(Student.class, 12424);

Student s2 =
    session2.get(Student.class, 12424);
```

you should **not** expect:

```text
s1 == s2
```

to be `true`.

That's because L2 is **not simply sharing the same managed Java object instance between Sessions**.

Conceptually:

```text
             L2 Cache
                │
        cached entity state
           ↙          ↘
          ↓            ↓
     Session 1      Session 2
          ↓            ↓
     Java object    Java object
```

This is an important difference between L1 and L2.

---

# 18. What if we disable L2?

This makes an excellent experiment.

Change:

```xml
<property name="hibernate.cache.use_second_level_cache">
    true
</property>
```

to:

```xml
<property name="hibernate.cache.use_second_level_cache">
    false
</property>
```

Run:

```text
Session 1
   ↓
L1 MISS
   ↓
Database
```

Then:

```text
Session 1 closes
```

Then:

```text
Session 2
   ↓
L1 MISS
   ↓
L2 disabled
   ↓
Database
```

So:

```text
WITHOUT L2

Session 1 → DB
Session 2 → DB
Session 3 → DB
Session 4 → DB
```

With L2:

```text
WITH L2

Session 1 → DB
             ↓
            L2

Session 2 → L2
Session 3 → L2
Session 4 → L2
```

---

# 19. What exactly is being cached?

This is another concept students frequently misunderstand.

Don't think:

```text
L2
 ↓
Student Java object
```

as if all Sessions simply share the exact same Java object.

Think more like:

```text
Database

Student:
id = 12424
name = Rahul
city = Lucknow

       ↓

L2 Cache

Student#12424
     ↓
cached entity state

       ↓

Session 2

creates/associates its own managed representation
```

This is why L2 can serve multiple Sessions while each Session still has its own persistence context.

---

# 20. Complete project structure

At the end:

```text
HibernateL2Cache
│
├── pom.xml
│
└── src
    │
    └── main
        │
        ├── java
        │   └── com
        │       └── example
        │           │
        │           ├── Student.java
        │           │
        │           └── SecondLevelCacheDemo.java
        │
        └── resources
            │
            ├── hibernate.cfg.xml
            │
            └── ehcache.xml
```

---

# 21. The complete flow students should remember

```text
                 SessionFactory
                       │
                       │
             ┌─────────┴─────────┐
             │                   │
         Session 1           Session 2
             │                   │
             ↓                   ↓
         L1 Cache             L1 Cache
             │                   │
             │                   │
             └─────────┬─────────┘
                       ↓
                  L2 Cache
                   Ehcache
                       │
                       ↓
                    MySQL
```

And the lookup sequence:

```text
session.get(Student.class, 12424)
                  │
                  ↓
          ┌───────────────┐
          │ L1 Cache      │
          │ Current       │
          │ Session       │
          └───────┬───────┘
                  │
             MISS ↓
          ┌───────────────┐
          │ L2 Cache      │
          │ Shared        │
          │ SessionFactory│
          └───────┬───────┘
                  │
             MISS ↓
          ┌───────────────┐
          │   Database    │
          └───────────────┘
```

## The three experiments I'd actually do in class

**Experiment 1 — Same Session**

```java
session.get(Student.class, 12424);
session.get(Student.class, 12424);
```

Result:

```text
1 DB query
L1 handles second request
```

**Experiment 2 — Different Sessions + L2 disabled**

```java
session1.get(Student.class, 12424);
session2.get(Student.class, 12424);
```

Result:

```text
2 DB queries
```

**Experiment 3 — Different Sessions + L2 enabled**

```java
session1.get(Student.class, 12424);
session2.get(Student.class, 12424);
```

Result:

```text
1 DB query
1 L2 hit
```

That progression makes the reason for **Second-Level Cache** extremely obvious: **L1 solves repeated reads inside one Session; L2 extends that caching benefit across Sessions.** Hibernate's current 6.6 documentation also exposes statistics and cache settings specifically for observing and configuring second-level caching. ([Hibernate Documentation][6])

