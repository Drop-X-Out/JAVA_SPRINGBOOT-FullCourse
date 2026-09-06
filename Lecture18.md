# Object-Relational Mapping (ORM)

## 1. What is Object-Relational Mapping?

**Object-Relational Mapping (ORM)** is a technique that allows us to work with a **relational database using programming language objects instead of writing SQL queries manually for every operation**.

In simple words:

> ORM creates a bridge between **Objects in your programming language** and **Tables in a database**.

For example, suppose we have a Java application and a database.

In Java, we might have:

```java
class Student {
    int id;
    String name;
    String email;
}
```

In a relational database, we might have:

```text
STUDENT
--------------------------------
id | name | email
--------------------------------
1  | Rahul | rahul@gmail.com
2  | Aman  | aman@gmail.com
```

The Java `Student` object and the database `STUDENT` table represent the same concept, but in two completely different worlds.

ORM helps us connect them.

```text
Java Application                    Relational Database

Student Object      <---------->    STUDENT Table
      |                                  |
      |                                  |
   id = 1                            id = 1
   name = Rahul                      name = Rahul
   email = ...                       email = ...
```

So instead of constantly thinking:

> "How do I convert this Java object into SQL?"

ORM handles much of that conversion for us.

---

# 2. Why Do We Need ORM?

Before understanding ORM, we should understand the problem it solves.

Imagine we have a `Student` class:

```java
class Student {

    private int id;
    private String name;
    private String email;

}
```

And we want to save this student in a database.

Without ORM, we might write SQL manually:

```sql
INSERT INTO student (id, name, email)
VALUES (1, 'Rahul', 'rahul@gmail.com');
```

If we want to retrieve the student:

```sql
SELECT * FROM student WHERE id = 1;
```

Then we receive a database row:

```text
1 | Rahul | rahul@gmail.com
```

Now our Java program needs to convert that row into:

```java
Student student = new Student();

student.setId(1);
student.setName("Rahul");
student.setEmail("rahul@gmail.com");
```

So we have two worlds:

```text
Java World
    ↓
Java Objects
    ↓
SQL Queries
    ↓
Database
    ↓
Database Rows
    ↓
Java Objects
```

We repeatedly perform conversions.

ORM attempts to reduce this manual work.

---

# 3. The Basic Idea of ORM

ORM works on a simple mapping concept.

```text
Programming Language          Database
------------------------------------------------

Class                         Table

Object                        Row

Object Attribute              Column

Object Relationship           Foreign Key Relationship
```

For example:

```java
class Student {

    int id;
    String name;
    String email;

}
```

can be mapped to:

```text
STUDENT TABLE

+----+-------+-------------------+
| id | name  | email             |
+----+-------+-------------------+
| 1  | Rahul | rahul@gmail.com   |
| 2  | Aman  | aman@gmail.com     |
+----+-------+-------------------+
```

The mapping is:

```text
Student class       → STUDENT table

id                  → id column

name                → name column

email               → email column
```

This is the fundamental concept behind ORM.

---

# 4. Understanding the Word "Object-Relational Mapping"

The name itself tells us what ORM does.

## Object

An **object** belongs to the object-oriented programming world.

Example:

```java
Student student = new Student();

student.setId(1);
student.setName("Rahul");
```

## Relational

A relational database stores data in structures such as:

* Tables
* Rows
* Columns
* Primary Keys
* Foreign Keys
* Relationships

Example:

```text
STUDENT

id | name  | email
----------------------------
1  | Rahul | rahul@gmail.com
```

## Mapping

Mapping means creating a relationship between the two.

```text
Java                     Database

Class       ---------->  Table

Object      ---------->  Row

Field       ---------->  Column
```

Therefore:

```text
Object + Relational Database + Mapping
                    =
                   ORM
```

---

# 5. ORM Without Any Framework

ORM is a concept.

It is important to understand this because many beginners think:

> ORM = Hibernate

That is not correct.

**ORM is a technique/concept.**

Hibernate is one technology that implements ORM.

Other ORM technologies exist for different programming languages.

For example:

```text
Java       → Hibernate, EclipseLink
Python     → Django ORM, SQLAlchemy
C#         → Entity Framework
Ruby       → Active Record
JavaScript → TypeORM, Prisma
```

The exact tools differ, but the fundamental idea remains similar:

```text
Programming Objects
        ↓
       ORM
        ↓
Relational Database
```

---

# 6. Traditional JDBC Approach

Before understanding Hibernate or another ORM framework, it is useful to understand what happens when we interact with a database manually.

Suppose we want to insert a student.

Using JDBC, we might write:

```java
String sql =
    "INSERT INTO student (name, email) VALUES (?, ?)";

PreparedStatement statement =
    connection.prepareStatement(sql);

statement.setString(1, "Rahul");
statement.setString(2, "rahul@gmail.com");

statement.executeUpdate();
```

This works.

But notice how much database-specific code we need to write.

For retrieval:

```java
String sql =
    "SELECT * FROM student WHERE id = ?";

PreparedStatement statement =
    connection.prepareStatement(sql);

statement.setInt(1, 1);

ResultSet resultSet =
    statement.executeQuery();
```

Then we manually convert the result:

```java
Student student = new Student();

student.setId(resultSet.getInt("id"));
student.setName(resultSet.getString("name"));
student.setEmail(resultSet.getString("email"));
```

For a small application, this may be manageable.

For a large application, the amount of repetitive database code can become significant.

---

# 7. What ORM Tries to Solve

ORM attempts to handle common database operations through objects.

Instead of thinking primarily in terms of:

```text
INSERT
UPDATE
SELECT
DELETE
```

we can often think in terms of:

```text
save()
update()
find()
delete()
```

Conceptually:

```java
Student student = new Student();

student.setName("Rahul");
student.setEmail("rahul@gmail.com");

studentRepository.save(student);
```

The ORM framework can generate the required SQL internally.

Conceptually, it may generate something similar to:

```sql
INSERT INTO student (name, email)
VALUES ('Rahul', 'rahul@gmail.com');
```

The developer works mainly with objects while the ORM handles much of the database interaction.

---

# 8. ORM Is Not Magic

ORM does not mean:

> "SQL has disappeared."

SQL is still involved.

The database still needs SQL internally.

The difference is that the developer does not necessarily need to manually write SQL for every basic operation.

The flow becomes:

```text
Developer
    ↓
Java Object
    ↓
ORM Framework
    ↓
Generated SQL
    ↓
Database
```

For example:

```java
studentRepository.save(student);
```

may result internally in SQL similar to:

```sql
INSERT INTO student ...
```

The exact SQL depends on the ORM framework, database, mappings, configuration, and operation being performed.

---

# 9. The Most Important ORM Mapping

For a beginner, remember these four mappings:

```text
Class       → Table

Object      → Row

Field       → Column

Relationship → Foreign Key / Relationship
```

Let's understand each one.

---

# 10. Class to Table Mapping

Suppose we have:

```java
class Student {
    int id;
    String name;
    String email;
}
```

We can map the class to:

```text
STUDENT
```

table.

Conceptually:

```text
Student
   ↓
STUDENT
```

This means:

> Objects created from the `Student` class will generally be represented by records in the `STUDENT` table.

---

# 11. Object to Row Mapping

Suppose we create:

```java
Student student = new Student();

student.setId(1);
student.setName("Rahul");
student.setEmail("rahul@gmail.com");
```

This object can correspond to:

```text
STUDENT

id | name  | email
----------------------------
1  | Rahul | rahul@gmail.com
```

So:

```text
Student Object
      ↓
Database Row
```

The object represents one particular record.

---

# 12. Field to Column Mapping

Our Java object contains fields:

```java
int id;
String name;
String email;
```

The database contains columns:

```text
id
name
email
```

So ORM maps:

```text
Java Field       Database Column

id          →    id

name        →    name

email       →    email
```

Sometimes the names are identical.

Sometimes they are different.

For example:

```java
String firstName;
```

could map to:

```text
first_name
```

ORM frameworks allow us to explicitly define such mappings.

---

# 13. Example of a Complete Mapping

Imagine this Java class:

```java
class Student {

    private int id;
    private String name;
    private String email;
    private int age;

}
```

Database:

```text
STUDENT

+----+-------+-------------------+-----+
| id | name  | email             | age |
+----+-------+-------------------+-----+
| 1  | Rahul | rahul@gmail.com   | 20  |
| 2  | Aman  | aman@gmail.com    | 21  |
+----+-------+-------------------+-----+
```

Mapping:

```text
Student class
      ↓
STUDENT table

Student.id
      ↓
STUDENT.id

Student.name
      ↓
STUDENT.name

Student.email
      ↓
STUDENT.email

Student.age
      ↓
STUDENT.age
```

This is ORM at its core.

---

# 14. ORM and CRUD

ORM is particularly useful for CRUD operations.

CRUD means:

```text
C → Create
R → Read
U → Update
D → Delete
```

For a `Student` entity:

```text
Create → Save a Student

Read   → Find a Student

Update → Modify a Student

Delete → Remove a Student
```

Conceptually:

```java
studentRepository.save(student);
```

```java
studentRepository.findById(1);
```

```java
studentRepository.delete(student);
```

The ORM framework translates these operations into appropriate database interactions.

---

# 15. What Is an Entity?

When working with ORM, you will frequently hear the word:

> Entity

An **entity** is an object that is associated with persistent data in a database.

For example:

```text
Student
Teacher
Course
Product
Order
Customer
Employee
```

can all represent entities.

For example:

```java
Student student;
```

can represent one student entity.

A database may contain:

```text
STUDENT TABLE
```

that stores the persistent state of these entities.

---

# 16. Persistence

Another important word is:

> Persistence

Persistence means:

> Data continues to exist even after the program stops running.

Consider:

```java
Student student = new Student();
student.setName("Rahul");
```

This object exists in application memory.

If the application stops, that object disappears from memory.

```text
RAM

Student Object
      ↓
Application stops
      ↓
Object disappears
```

But if we save it into a database:

```text
Student Object
      ↓
ORM
      ↓
Database
```

the information can remain stored:

```text
Database

STUDENT
----------------
1 | Rahul
```

Later, when the application starts again, the object can be reconstructed from the database.

That is persistence.

---

# 17. Object State and Database State

ORM manages the relationship between:

```text
Object State
     ↕
Database State
```

Suppose:

```java
Student student = new Student();

student.setName("Rahul");
```

Later:

```java
student.setName("Aman");
```

The object has changed.

ORM can synchronize changes with the database depending on the framework and persistence context.

Conceptually:

```text
Before

Java Object
name = Rahul

Database
name = Rahul
```

After changing the object:

```text
Java Object
name = Aman

Database
name = Rahul
```

After synchronization:

```text
Java Object
name = Aman

Database
name = Aman
```

This object-database synchronization is one of the major ideas behind ORM.

---

# 18. The Object-Relational Impedance Mismatch

This is one of the most important concepts behind ORM.

Programming languages and relational databases represent data differently.

Object-oriented programming uses:

```text
Classes
Objects
Inheritance
Encapsulation
References
Collections
```

Relational databases use:

```text
Tables
Rows
Columns
Primary Keys
Foreign Keys
Joins
```

For example:

```text
Object-Oriented World

Student
   |
   |---- name
   |---- email
   |
   └---- course object
```

Database world:

```text
STUDENT TABLE
-------------------------
id | name | course_id

COURSE TABLE
-------------------------
id | course_name
```

These models are not identical.

The difference between these models is commonly called:

> Object-Relational Impedance Mismatch

ORM exists largely to bridge this mismatch.

---

# 19. Example of the Impedance Mismatch

Suppose we have:

```java
class Student {

    int id;
    String name;

    Course course;
}
```

A Java object can directly contain another object:

```text
Student
   |
   └── Course
```

But a relational database generally does not store an entire `Course` object inside a student row.

Instead, it might store:

```text
STUDENT

id | name | course_id
---------------------
1  | Rahul| 101
```

And:

```text
COURSE

id  | name
-----------
101 | Java
102 | Python
```

The database uses:

```text
course_id
```

to establish the relationship.

ORM understands how to map:

```text
Java Object Relationship
          ↓
Database Relationship
```

---

# 20. Relationships in ORM

Database applications rarely contain only one table.

Usually, entities are related.

For example:

```text
Student
Course
Teacher
Department
```

may have relationships.

Common relationships are:

```text
One-to-One
One-to-Many
Many-to-One
Many-to-Many
```

Understanding these is extremely important for ORM.

---

# 21. One-to-One Relationship

One-to-one means:

> One object is associated with exactly one other object.

Example:

```text
Person
   |
   └── Passport
```

One person has one passport.

Database:

```text
PERSON
----------------
id | name | passport_id

PASSPORT
----------------
id | passport_number
```

ORM maps the object relationship to the database relationship.

---

# 22. One-to-Many Relationship

One-to-many means:

> One object can be associated with multiple objects.

Example:

```text
Department
    |
    |---- Employee
    |---- Employee
    |---- Employee
```

One department can have many employees.

Database:

```text
DEPARTMENT
----------------
id | name

EMPLOYEE
----------------
id | name | department_id
```

The foreign key:

```text
department_id
```

connects employees to their department.

---

# 23. Many-to-One Relationship

Many-to-one is the reverse perspective of one-to-many.

Example:

```text
Employee → Department
```

Many employees can belong to one department.

```text
Employee 1 ──┐
Employee 2 ──┤
Employee 3 ──┤──→ Department
Employee 4 ──┘
```

In the database:

```text
EMPLOYEE

id | name | department_id
```

Multiple rows can contain the same `department_id`.

---

# 24. Many-to-Many Relationship

Many-to-many means:

> Multiple objects on one side can be related to multiple objects on the other side.

Example:

```text
Student ↔ Course
```

One student can take multiple courses.

One course can have multiple students.

```text
Student 1 ─── Course A
          └── Course B

Student 2 ─── Course A
          └── Course C
```

A relational database generally uses a junction table:

```text
STUDENT
----------------
id | name

COURSE
----------------
id | name

STUDENT_COURSE
-------------------------
student_id | course_id
```

ORM can map this many-to-many relationship.

---

