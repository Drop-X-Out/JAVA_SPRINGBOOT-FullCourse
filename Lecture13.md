# Hibernate Persistence Operation: `refresh()`

> `refresh()` reloads an entity from the database and replaces the current object's values with the latest values stored in the database.

In simple words:

* Database = Original copy
* Java object = Local copy

`refresh()` tells Hibernate:

> "Ignore the values currently stored inside my Java object. Go back to the database and load the latest values again."

---

# Real-world example

Imagine you have a student's record.

| Database   | Java Object |
| ---------- | ----------- |
| Name = Jack | Name = Jack  |

The data is the same.

---

## Step 1: Load the student

```java
Student student = session.find(Student.class, 1);

System.out.println(student);
```

Output:

```text
Student{id=1, name=Jack, course=Java}
```

---

## Step 2: Change the object

```java
student.setName("John");

System.out.println(student);
```

Output:

```text
Student{id=1, name=John, course=Java}
```

The database still contains:

```text
Student{id=1, name=Jack, course=Java}
```

Only the Java object has changed.

---

## Step 3: Call `refresh()`

```java
session.refresh(student);

System.out.println(student);
```

Output:

```text
Student{id=1, name=Jack, course=Java}
```

Hibernate went back to the database, fetched the latest record, and replaced the object's values.

---

# Complete example

---

## Student.java

```java
package com.example;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
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

## RefreshDemo.java

```java
package com.example;

import org.hibernate.Session;

public class RefreshDemo
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

        System.out.println("Original object");

        System.out.println(student);

        student.setName("John");

        System.out.println();

        System.out.println("After changing the object");

        System.out.println(student);

        session.refresh(student);

        System.out.println();

        System.out.println("After refresh");

        System.out.println(student);

        session.close();
    }
}
```

---

# What happens internally?

```text
Database
    ↓

session.find()

    ↓

Student Object

    ↓

student.setName("John")

    ↓

Java Object Updated

    ↓

session.refresh(student)

    ↓

Database Read Again

    ↓

Java Object Replaced
```

---

# Why do we use `refresh()`?

## 1. Another user changed the database

Suppose two users are working on the same record.

```text
User 1 → Loads Student 1

User 2 → Updates Student 1

User 1 → Calls refresh()
```

`refresh()` reloads the latest data.

---

## 2. A database trigger changed the data

Suppose your database automatically updates the `updated_at` column.

```text
Java Object

↓

Database Trigger Executes

↓

updated_at Changes

↓

refresh()

↓

Java Object Gets the New Value
```

---

# Important point

`refresh()` works only with a **persistent object**.

```java
Student student =
        session.find(
                Student.class,
                1
        );

session.refresh(student);
```

This works because `student` is attached to the current session.

---

# What if we don't use `refresh()`?

```java
Student student =
        session.find(
                Student.class,
                1
        );

student.setName("John");

System.out.println(student);
```

Output:

```text
Student{id=1, name=John}
```

The database may still contain:

```text
Student{id=1, name=Jack}
```

The Java object and the database become different.

`refresh()` synchronizes them again.

---

# `refresh()` vs `merge()`

| `refresh()`             | `merge()`              |
| ----------------------- | ---------------------- |
| Database → Object       | Object → Database      |
| Reloads data            | Updates data           |
| Discards local changes  | Saves local changes    |
| Reads from the database | Writes to the database |

Think of it like this:

```text
refresh()

Database  →  Java Object

merge()

Java Object  →  Database
```

---

# Hibernate Persistence Operations: `commit()` and `rollback()`

Before learning `commit()` and `rollback()`, you must understand one word:

# What is a transaction?

A **transaction** is a group of database operations that Hibernate treats as **one unit of work**.

For example:

```text
Insert Student

↓

Insert Course

↓

Insert Fee Details

↓

Transaction Ends
```

If all operations succeed, the transaction is **committed**.

If any operation fails, the transaction is **rolled back**.

---

# Real-world example: Online banking

Suppose you transfer ₹10,000 from Account A to Account B.

```text
Step 1: Deduct ₹10,000 from Account A.

Step 2: Add ₹10,000 to Account B.
```

What happens if Step 1 succeeds but Step 2 fails?

```text
Account A = ₹40,000

Account B = ₹20,000
```

₹10,000 disappears.

This is a serious problem.

Transactions solve this problem.

```text
Begin Transaction

↓

Deduct ₹10,000

↓

Add ₹10,000

↓

Commit Transaction
```

If any operation fails:

```text
Begin Transaction

↓

Deduct ₹10,000

↓

Error

↓

Rollback Transaction
```

The database returns to its previous state.

---

# `commit()`

`commit()` permanently saves all changes to the database.

```java
transaction.commit();
```

---

# Flow of `commit()`

```text
Begin Transaction

↓

Perform Operations

↓

commit()

↓

Changes Saved Permanently
```

---

# Example: `commit()`

## Insert a student

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class CommitDemo
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
                        "John",
                        "Java"
                );

        session.persist(student);

        transaction.commit();

        session.close();
    }
}
```

---

# What happens internally?

```text
Student Object Created

↓

session.persist(student)

↓

Object Stored Inside Hibernate

↓

transaction.commit()

↓

SQL Query Executed

↓

Data Saved in the Database
```

Without `commit()`, the data may never be saved.

---

# `rollback()`

`rollback()` cancels all changes made during the current transaction.

```java
transaction.rollback();
```

---

# Flow of `rollback()`

```text
Begin Transaction

↓

Perform Operations

↓

Error Occurs

↓

rollback()

↓

All Changes Cancelled
```

---

# Example: `rollback()`

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class RollbackDemo
{
    public static void main(String[] args)
    {
        Session session =
                HibernateUtil
                        .getSessionFactory()
                        .openSession();

        Transaction transaction =
                session.beginTransaction();

        try
        {
            Student student =
                    new Student(
                            1,
                            "John",
                            "Java"
                    );

            session.persist(student);

            int number = 10 / 0;

            transaction.commit();
        }
        catch (Exception e)
        {
            transaction.rollback();

            System.out.println(
                    "Transaction rolled back."
            );
        }

        session.close();
    }
}
```

---

# What happens here?

```text
Transaction Starts

↓

Student Created

↓

session.persist(student)

↓

10 / 0

↓

ArithmeticException

↓

rollback()

↓

Student Not Saved
```

---

# Why do we write `commit()` inside `try`?

```java
try
{
    transaction.commit();
}
catch (Exception e)
{
    transaction.rollback();
}
```

If `commit()` fails, `rollback()` can undo all changes.

---

# The most common pattern in Hibernate

```java
Session session = null;

Transaction transaction = null;

try
{
    session =
            HibernateUtil
                    .getSessionFactory()
                    .openSession();

    transaction =
            session.beginTransaction();

    Student student =
            new Student(
                    1,
                    "John",
                    "Java"
            );

    session.persist(student);

    transaction.commit();
}
catch (Exception e)
{
    if (transaction != null)
    {
        transaction.rollback();
    }

    e.printStackTrace();
}
finally
{
    if (session != null)
    {
        session.close();
    }
}
```

---

# `commit()` vs `rollback()`

| `commit()`                                           | `rollback()`                 |
| ---------------------------------------------------- | ---------------------------- |
| Saves changes                                        | Cancels changes              |
| Makes data permanent                                 | Restores the previous state  |
| Used after successful operations                     | Used after failed operations |
| Executes `INSERT`, `UPDATE`, and `DELETE` operations | Undoes pending changes       |

---

# Easy way to remember

```text
commit()

I am happy.

Everything worked.

Save everything.


rollback()

Something went wrong.

Cancel everything.
```

---

# Hibernate Persistence Operation: `flush()`

`flush()` is one of the **most confusing** Hibernate operations for beginners.

Many students think:

> `flush()` = `commit()`

This is **wrong**.

`flush()` and `commit()` are two different operations.

---

# One-line definition

> `flush()` synchronizes the current state of the Hibernate session with the database.

In simple words:

```text
Java Object
        ↓
Hibernate Session (Cache)
        ↓
flush()
        ↓
Database
```

`flush()` sends SQL statements (`INSERT`, `UPDATE`, `DELETE`) to the database.

**But it does not permanently save the transaction.**

---

# Before understanding `flush()`, understand the Session Cache

When you write:

```java
session.persist(student);
```

Most beginners think:

```text
Student Object
        ↓
Database
```

This is not what happens.

Hibernate first stores the object in the **first-level cache (Session Cache).**

```text
Student Object
        ↓
Session Cache
        ↓
Database
```

---

# Example

```java
Transaction transaction =
        session.beginTransaction();

Student student =
        new Student(
                1,
                "John",
                "Java"
        );

session.persist(student);
```

What happens internally?

```text
Transaction Starts
        ↓
Student Object Created
        ↓
session.persist(student)
        ↓
Object Stored Inside Session Cache
```

At this point, Hibernate may **not** execute an `INSERT` query immediately.

---

# What does `flush()` do?

```java
session.flush();
```

Hibernate checks the session cache.

```text
Session Cache
        ↓
Detect Changes
        ↓
Generate SQL
        ↓
Send SQL to Database
```

---

# Complete example

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class FlushDemo
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
                        "John",
                        "Java"
                );

        session.persist(student);

        System.out.println(
                "Before flush()"
        );

        session.flush();

        System.out.println(
                "After flush()"
        );

        transaction.rollback();

        session.close();
    }
}
```

---

# What happens internally?

```text
Transaction Starts
        ↓
session.persist(student)
        ↓
Student Stored in Session Cache
        ↓
session.flush()
        ↓
INSERT Query Executed
        ↓
transaction.rollback()
        ↓
INSERT Cancelled
```

---

# The most important interview question

**If `flush()` executes the SQL query, why can `rollback()` still undo the changes?**

Because:

```text
flush()
        ↓
SQL Executed
        ↓
Transaction Still Active
        ↓
rollback()
        ↓
Changes Cancelled
```

`flush()` executes SQL.

`commit()` makes those changes permanent.

Until `commit()` happens, the transaction can still be rolled back.

---

# `flush()` vs `commit()`

| `flush()`                                  | `commit()`                    |
| ------------------------------------------ | ----------------------------- |
| Synchronizes the session with the database | Permanently saves changes     |
| Executes SQL statements                    | Ends the transaction          |
| Transaction remains active                 | Transaction ends              |
| Changes can still be rolled back           | Changes cannot be rolled back |

---

# Example: `flush()`

```java
session.persist(student);

session.flush();

transaction.rollback();
```

Result:

```text
INSERT Executed
        ↓
rollback()
        ↓
Data Removed
```

---

# Example: `commit()`

```java
session.persist(student);

transaction.commit();
```

Result:

```text
INSERT Executed
        ↓
Transaction Ends
        ↓
Data Permanently Stored
```

---

# Does `commit()` automatically call `flush()`?

**Yes.**

```java
transaction.commit();
```

Internally:

```text
transaction.commit()
        ↓
session.flush()
        ↓
Execute SQL
        ↓
Commit Transaction
```

This is why most applications never call `flush()` explicitly.

---

# When should we use `flush()`?

---

## 1. To force Hibernate to execute SQL immediately

```java
session.persist(student);

session.flush();

System.out.println(
        "SQL already executed."
);
```

---

## 2. To detect database constraint errors early

Suppose a student with ID `1` already exists.

```java
Student student =
        new Student(
                1,
                "John",
                "Java"
        );

session.persist(student);

session.flush();
```

Hibernate immediately executes:

```sql
INSERT INTO student
VALUES (1, 'John', 'Java');
```

You'll immediately see the primary key violation.

Without `flush()`, the exception might appear only during `commit()`.

---

## 3. Before executing an HQL query

Hibernate may automatically call `flush()`.

```java
student.setName("Ahmed");

session.createQuery(
        "FROM Student"
).list();
```

Internally:

```text
UPDATE Student
        ↓
Automatic flush
        ↓
Execute SELECT Query
```

Hibernate does this to ensure the query sees the latest data.

---

# Automatic flushing

Hibernate automatically calls `flush()` in many situations.

Examples:

```text
Before commit()
Before some queries
Before transaction completion
```

---

# Manual flushing

You can call it yourself.

```java
session.flush();
```

---

# Real-world analogy

Imagine you're writing a document.

```text
Type a paragraph
        ↓
Press Ctrl + S
        ↓
The file is saved
```

`flush()` is like pressing **Ctrl + S**.

```text
Write document
        ↓
Save
        ↓
Continue editing
```

`commit()` is like **closing the document permanently.**

```text
Write document
        ↓
Save
        ↓
Close document
```

---

# Easy way to remember

```text
persist()
        ↓
Put data into the session cache

flush()
        ↓
Send SQL to the database

commit()
        ↓
Make changes permanent

rollback()
        ↓
Cancel everything
```

---
# Hibernate Persistence Operation: `clear()`

`clear()` is another operation that confuses many beginners because it looks similar to `evict()`, `refresh()`, and `close()`.

The simplest definition is:

> `clear()` removes **all objects** from the Hibernate session (first-level cache).

---

# One-line explanation

```java
session.clear();
```

This tells Hibernate:

> "Forget every object you are currently managing."

---

# Before understanding `clear()`, understand the first-level cache

When you load an object:

```java
Student student =
        session.find(
                Student.class,
                1
        );
```

The object is stored in the session cache.

```text
Database
    ↓

session.find()

    ↓

Session Cache
    ↓

Student Object
```

---

# Example

```java
Student student1 =
        session.find(
                Student.class,
                1
        );

Student student2 =
        session.find(
                Student.class,
                2
        );
```

The session cache now contains:

```text
Session Cache

Student 1

Student 2
```

---

# Calling `clear()`

```java
session.clear();
```

Now the session cache becomes:

```text
Session Cache

Empty
```

Both objects are removed from Hibernate's management.

---

# Complete example

```java
package com.example;

import org.hibernate.Session;

public class ClearDemo
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

        session.clear();

        student.setName("John");

        session.close();
    }
}
```

---

# What happens internally?

```text
session.find()

        ↓

Student loaded

        ↓

Student stored in the Session Cache

        ↓

session.clear()

        ↓

Student removed from the Session Cache

        ↓

student.setName("John")

        ↓

Only the Java object changes

        ↓

Database remains unchanged
```

---

# The most important point

Before `clear()`:

```text
Student Object

        ↓

Persistent State

        ↓

Hibernate manages the object
```

After `clear()`:

```text
Student Object

        ↓

Detached State

        ↓

Hibernate no longer manages the object
```

---

# Example with `flush()`

```java
Transaction transaction =
        session.beginTransaction();

Student student =
        session.find(
                Student.class,
                1
        );

student.setName("John");

session.clear();

transaction.commit();
```

Will the database be updated?

**No.**

---

# Why?

```text
Student loaded

        ↓

Student name changed

        ↓

session.clear()

        ↓

Object becomes detached

        ↓

commit()

        ↓

No UPDATE query
```

Hibernate doesn't know about the changes anymore.

---

# `clear()` vs `evict()`

| `clear()`                       | `evict()`                        |
| ------------------------------- | -------------------------------- |
| Removes all objects             | Removes one object               |
| Clears the entire session cache | Removes a specific object        |
| All entities become detached    | Only one entity becomes detached |

---

# Example of `evict()`

```java
Student student1 =
        session.find(
                Student.class,
                1
        );

Student student2 =
        session.find(
                Student.class,
                2
        );

session.evict(student1);
```

Result:

```text
Session Cache

Student 2
```

`student1` is removed.

`student2` is still managed.

---

# `clear()` vs `close()`

| `clear()`                     | `close()`                   |
| ----------------------------- | --------------------------- |
| Clears the session cache      | Closes the entire session   |
| The session can still be used | The session cannot be used  |
| Removes all managed entities  | Releases database resources |

---

# Example

```java
session.clear();

session.find(
        Student.class,
        1
);
```

This works because the session is still open.

---

```java
session.close();

session.find(
        Student.class,
        1
);
```

This throws an exception because the session is already closed.

---

# Real-world example

Imagine a classroom.

```text
Teacher = Hibernate Session

Students = Persistent Objects
```

Before `clear()`:

```text
Teacher remembers every student.
```

After `clear()`:

```text
Teacher forgets every student.
```

The students still exist, but the teacher is no longer tracking them.

---

# Easy way to remember

```text
refresh()

Database → Object


flush()

Object → Database


clear()

Session → Empty


evict()

Remove one object


close()

Destroy the session
```

---
# Hibernate Persistence Operation: `evict()`

`evict()` is used to remove **a specific object** from the Hibernate session (first-level cache).

The simplest definition is:

> `evict()` removes only one entity from the persistence context and changes its state from **persistent** to **detached**.

---

# First, understand the Session Cache

When you load an object:

```java
Student student =
        session.find(
                Student.class,
                1
        );
```

Hibernate doesn't read the object from the database every time.

Instead, it stores the object inside the **first-level cache (Session Cache)**.

```text
Database
    ↓

session.find()

    ↓

Session Cache
    ↓

Student Object
```

---

# What does `evict()` do?

```java
session.evict(student);
```

Hibernate says:

> "Stop managing this object, but keep managing everything else."

---

# Object states before and after `evict()`

Before `evict()`:

```text
Student Object
        ↓
Persistent State
        ↓
Managed by Hibernate
```

After `evict()`:

```text
Student Object
        ↓
Detached State
        ↓
Not managed by Hibernate
```

---

# Complete example

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class EvictDemo
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

        System.out.println(student);

        session.evict(student);

        student.setName("John");

        transaction.commit();

        session.close();
    }
}
```

---

# What happens internally?

```text
session.find()
        ↓
Student loaded
        ↓
Student stored in the Session Cache
        ↓
session.evict(student)
        ↓
Student removed from the Session Cache
        ↓
student.setName("John")
        ↓
Java object changes
        ↓
commit()
        ↓
No UPDATE query
```

---

# Why doesn't the database update?

Consider this code:

```java
Student student =
        session.find(
                Student.class,
                1
        );

session.evict(student);

student.setName("John");

transaction.commit();
```

The database is **not** updated.

Why?

Because Hibernate tracks only **persistent objects**.

After `evict()`:

```text
Persistent Object
        ↓
evict()
        ↓
Detached Object
        ↓
Hibernate stops tracking changes
```

---

# Example with two objects

```java
Student student1 =
        session.find(
                Student.class,
                1
        );

Student student2 =
        session.find(
                Student.class,
                2
        );

session.evict(student1);

student1.setName("Jack");

student2.setName("Ahmed");

transaction.commit();
```

---

# Which object is updated?

```text
student1 → Not updated

student2 → Updated
```

Why?

```text
Session Cache

Before evict()

Student 1

Student 2


After evict(student1)

Student 2
```

Only `student2` remains inside the session cache.

---

# `evict()` vs `clear()`

| `evict()`                        | `clear()`                   |
| -------------------------------- | --------------------------- |
| Removes one object               | Removes all objects         |
| Accepts an entity as a parameter | Doesn't need a parameter    |
| Only one object becomes detached | All objects become detached |

---

## `evict()`

```java
session.evict(student);
```

Result:

```text
Session Cache

Student 2

Student 3
```

---

## `clear()`

```java
session.clear();
```

Result:

```text
Session Cache

Empty
```

---

# `evict()` vs `refresh()`

| `evict()`                          | `refresh()`                               |
| ---------------------------------- | ----------------------------------------- |
| Removes an object from the session | Reloads an object from the database       |
| Creates a detached object          | Keeps the object persistent               |
| Stops tracking changes             | Synchronizes the object with the database |

---

# `evict()` vs `close()`

| `evict()`                             | `close()`                    |
| ------------------------------------- | ---------------------------- |
| Removes one object                    | Closes the entire session    |
| The session remains active            | The session becomes unusable |
| Other entities continue to be managed | All entities are detached    |

---

# Real-world example

Imagine a classroom.

```text
Teacher = Hibernate Session

Students = Entity Objects
```

Before `evict()`:

```text
Teacher remembers:

Student 1

Student 2

Student 3
```

---

After `evict(student1)`:

```text
Teacher remembers:

Student 2

Student 3
```

The teacher forgets only **Student 1**.

---

# When should we use `evict()`?

## 1. To reduce memory usage

```java
for (int i = 1; i <= 10000; i++)
{
    Student student =
            session.find(
                    Student.class,
                    i
            );

    process(student);

    session.evict(student);
}
```

Without `evict()`, all 10,000 objects remain in the session cache.

With `evict()`, processed objects are removed immediately.

---

## 2. To stop automatic updates

```java
Student student =
        session.find(
                Student.class,
                1
        );

session.evict(student);

student.setName("John");
```

Hibernate ignores the modification.

---

# Easy way to remember

```text
refresh()

Database → Object


flush()

Object → Database


evict()

Remove one object


clear()

Remove all objects


close()

Destroy the session
```

---
