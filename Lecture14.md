# Hibernate Bulk Operation: HQL (Hibernate Query Language)

If **`persist()`**, **`find()`**, **`get()`**, **`merge()`**, and **`remove()`** are called **single-row operations**, then **HQL is called a bulk operation** because it can affect **multiple records at the same time**.

Think about a student database.

| Operation   | Type       | Example                     |
| ----------- | ---------- | --------------------------- |
| `persist()` | Single-row | Insert one student          |
| `find()`    | Single-row | Find one student            |
| `merge()`   | Single-row | Update one student          |
| `remove()`  | Single-row | Delete one student          |
| HQL         | Bulk       | Update 100 students at once |

---

# What is HQL?

**HQL (Hibernate Query Language)** is a query language provided by Hibernate.

It looks similar to SQL, but there is one very important difference.

## SQL works with tables and columns.

```sql
SELECT * FROM students;
```

* `students` → Table name

---

## HQL works with entities and class properties.

```java
from Student
```

* `Student` → Java class (Entity)

---

# SQL vs HQL

Suppose we have the following entity.

```java
@Entity
@Table(name = "students")
public class Student
{
    @Id
    private int id;

    private String name;

    private int age;
}
```

---

## SQL

```sql
SELECT * FROM students WHERE age > 20;
```

* `students` → Database table

---

## HQL

```java
from Student where age > 20
```

* `Student` → Java class

---

# Why do we use HQL?

Imagine there are 1,000 students.

Suppose you want to:

* Delete all students whose age is less than 18.
* Increase every student's age by 1.
* Find all students in a particular city.

Will you call `find()` 1,000 times?

```java
for(int i=1;i<=1000;i++)
{
    Student student = session.find(Student.class,i);

    student.setAge(student.getAge()+1);

    session.merge(student);
}
```

No.

This approach is very slow.

Instead, use HQL.

```java
update Student set age=age+1
```

One query can update all records.

---

# Types of HQL Bulk Operations

There are three major bulk operations.

1. Select
2. Update
3. Delete

---

# Project Structure

```text
src
│
├── Student.java
│
├── HibernateUtil.java
│
├── SelectDemo.java
│
├── UpdateDemo.java
│
└── DeleteDemo.java
```

---

# Student Entity

```java
package com.example;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "students")
public class Student
{
    @Id
    private int id;

    private String name;

    private int age;

    public Student()
    {
    }

    public Student(int id, String name, int age)
    {
        this.id = id;
        this.name = name;
        this.age = age;
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

    public int getAge()
    {
        return age;
    }

    public void setAge(int age)
    {
        this.age = age;
    }

    @Override
    public String toString()
    {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

---

# HibernateUtil.java

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

# 1. HQL Select Operation

## Fetch all students

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.query.Query;

import java.util.List;

public class SelectDemo
{
    public static void main(String[] args)
    {
        Session session =
                HibernateUtil
                        .getSessionFactory()
                        .openSession();

        Query<Student> query =
                session.createQuery(
                        "from Student",
                        Student.class
                );

        List<Student> students =
                query.getResultList();

        for(Student student : students)
        {
            System.out.println(student);
        }

        session.close();
    }
}
```

---

## Understanding the code

### `session.createQuery()`

```java
session.createQuery(
        "from Student",
        Student.class
);
```

Creates an HQL query.

---

### `"from Student"`

```java
"from Student"
```

Equivalent SQL:

```sql
SELECT * FROM students;
```

---

### `Student.class`

```java
Student.class
```

Tells Hibernate that the query will return `Student` objects.

---

### `getResultList()`

```java
query.getResultList();
```

Returns a `List<Student>`.

---

# Select with a condition

```java
Query<Student> query =
        session.createQuery(
                "from Student where age>20",
                Student.class
        );
```

Equivalent SQL:

```sql
SELECT * FROM students
WHERE age>20;
```

---

# 2. HQL Update Operation

Increase every student's age by 1.

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query;

public class UpdateDemo
{
    public static void main(String[] args)
    {
        Session session =
                HibernateUtil
                        .getSessionFactory()
                        .openSession();

        Transaction transaction =
                session.beginTransaction();

        Query query =
                session.createQuery(
                        "update Student set age=age+1"
                );

        int rows =
                query.executeUpdate();

        transaction.commit();

        System.out.println(
                rows + " rows updated."
        );

        session.close();
    }
}
```

---

## Understanding `executeUpdate()`

```java
int rows =
        query.executeUpdate();
```

This method:

* Executes the query.
* Returns the number of affected rows.

Suppose 100 students exist.

```text
100 rows updated.
```

---

# 3. HQL Delete Operation

Delete all students whose age is less than 18.

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query;

public class DeleteDemo
{
    public static void main(String[] args)
    {
        Session session =
                HibernateUtil
                        .getSessionFactory()
                        .openSession();

        Transaction transaction =
                session.beginTransaction();

        Query query =
                session.createQuery(
                        "delete from Student where age<18"
                );

        int rows =
                query.executeUpdate();

        transaction.commit();

        System.out.println(
                rows + " rows deleted."
        );

        session.close();
    }
}
```

---

# Why is a transaction required?

This will fail:

```java
Query query =
        session.createQuery(
                "update Student set age=30"
        );

query.executeUpdate();
```

Without a transaction, Hibernate will throw an exception.

---

Always do this:

```java
Transaction transaction =
        session.beginTransaction();

query.executeUpdate();

transaction.commit();
```

---

# Important Methods

| Method              | Purpose                              |
| ------------------- | ------------------------------------ |
| `createQuery()`     | Creates an HQL query                 |
| `getResultList()`   | Returns multiple objects             |
| `getSingleResult()` | Returns one object                   |
| `executeUpdate()`   | Executes update or delete operations |
| `setParameter()`    | Passes values to a query             |

---

# Using Parameters (Very Important)

Never write this:

```java
String name = "John";

Query<Student> query =
        session.createQuery(
                "from Student where name='" + name + "'",
                Student.class
        );
```

Use parameters.

```java
Query<Student> query =
        session.createQuery(
                "from Student where name=:studentName",
                Student.class
        );

query.setParameter(
        "studentName",
        "John"
);
```

---

# Real-World Example

Suppose your college has 5,000 students.

The director says:

> Increase every student's attendance by 5%.

Will you fetch all 5,000 students one by one?

```java
find();

merge();

find();

merge();

find();

merge();
```

No.

Use one HQL query.

```java
update Student
set attendance=attendance+5
```

Hibernate will update all rows with a single database query.

---

# Remember This Rule

```text
persist()
find()
get()
merge()
remove()

↓

Single-row operations

-------------------------

HQL

↓

Multi-row (bulk) operations
```
