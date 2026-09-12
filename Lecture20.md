# Hibernate ORM: Cascade

If you already understand **OneToOne, OneToMany, ManyToOne, and ManyToMany**, then **Cascade** is the next important concept.

The easiest way to understand Cascade is:

> **Cascade tells Hibernate what should happen to the related entity when an operation is performed on the parent entity.**

For example:

```text
Department
    |
    |--- Employee 1
    |--- Employee 2
    |--- Employee 3
```

If you save a `Department`, should Hibernate automatically save its Employees?

If you delete a `Department`, should Hibernate automatically delete its Employees?

**Cascade answers these questions.**

---

# 1. What Does Cascade Mean?

Suppose we have:

```text
Department
   |
   ├── Employee 1
   ├── Employee 2
   └── Employee 3
```

Without Cascade:

```java
session.persist(department);
```

Hibernate may save the `Department`, but it will **not automatically perform the same operation on the Employees**.

With:

```java
cascade = CascadeType.ALL
```

Hibernate understands:

```text
Operation on Department
        ↓
Apply the operation to Employees
```

For example:

```text
persist Department
       ↓
persist Employees

remove Department
       ↓
remove Employees
```

---

# 2. Real-Life Example

Imagine a company has a department.

```text
IT Department
    |
    ├── Rahul
    ├── Amit
    └── Priya
```

You create the department and employees together.

Without Cascade, you might have to do:

```java
session.persist(employee1);
session.persist(employee2);
session.persist(employee3);

session.persist(department);
```

With Cascade:

```java
session.persist(department);
```

Hibernate can automatically persist the employees if Cascade is configured.

---

# 3. Cascade Types

Hibernate/JPA provides several Cascade types:

```text
CascadeType.PERSIST
CascadeType.MERGE
CascadeType.REMOVE
CascadeType.REFRESH
CascadeType.DETACH
CascadeType.ALL
```

Let's understand them one by one.

---

# 4. CascadeType.PERSIST

`PERSIST` means:

> When the parent is persisted, automatically persist the child.

Example:

```java
@OneToMany(cascade = CascadeType.PERSIST)
private List<Employee> employees;
```

Now:

```java
session.persist(department);
```

will cause:

```text
Department
    ↓
Employee 1
Employee 2
Employee 3
```

to be persisted.

### Without Cascade

```java
session.persist(department);
```

Only the department is explicitly persisted.

### With PERSIST

```java
session.persist(department);
```

Department **and its employees** are persisted.

---

# 5. CascadeType.MERGE

`MERGE` means:

> When the parent is merged, automatically merge the child.

Suppose you retrieve an object, modify it, and then use:

```java
session.merge(department);
```

With:

```java
cascade = CascadeType.MERGE
```

Hibernate also merges the associated employees.

Example:

```java
@OneToMany(cascade = CascadeType.MERGE)
private List<Employee> employees;
```

Then:

```java
session.merge(department);
```

means approximately:

```text
merge Department
      ↓
merge Employees
```

---

# 6. CascadeType.REMOVE

This one is extremely important.

`REMOVE` means:

> When the parent is deleted, automatically delete the child.

Example:

```java
@OneToMany(cascade = CascadeType.REMOVE)
private List<Employee> employees;
```

Suppose:

```text
Department ID = 1

Employees:
ID 101
ID 102
ID 103
```

You execute:

```java
session.remove(department);
```

Hibernate can perform:

```text
DELETE Employee 101
DELETE Employee 102
DELETE Employee 103

DELETE Department 1
```

So:

```text
remove Department
       ↓
remove Employees
```

### ⚠️ Important

Be careful with `CascadeType.REMOVE`.

You should not blindly use it everywhere.

For example, imagine:

```text
Student
   |
   └── Course
```

If many students share the same course, deleting one student should **not delete the course**.

So Cascade REMOVE depends heavily on the relationship and ownership.

---

# 7. CascadeType.REFRESH

`REFRESH` means:

> When the parent is refreshed from the database, refresh the associated child as well.

Example:

```java
@OneToMany(cascade = CascadeType.REFRESH)
private List<Employee> employees;
```

Then:

```java
session.refresh(department);
```

will also refresh the employees.

Think:

```text
Database
   ↓
refresh Department
   ↓
refresh Employees
```

This is less commonly used in beginner projects.

---

# 8. CascadeType.DETACH

`DETACH` means:

> When the parent is detached from the Hibernate persistence context, detach the associated child too.

Example:

```java
@OneToMany(cascade = CascadeType.DETACH)
private List<Employee> employees;
```

Then:

```java
session.detach(department);
```

also detaches the employees.

Again, this is less commonly used in basic applications.

---

# 9. CascadeType.ALL

`ALL` means:

> Apply all cascade operations.

So:

```java
cascade = CascadeType.ALL
```

is equivalent to:

```java
cascade = {
    CascadeType.PERSIST,
    CascadeType.MERGE,
    CascadeType.REMOVE,
    CascadeType.REFRESH,
    CascadeType.DETACH
}
```

Therefore:

```java
@OneToMany(cascade = CascadeType.ALL)
```

allows the parent operations to cascade to its children.

---

# 10. Most Important Difference

Remember this table:

| Cascade Type | Meaning                        |
| ------------ | ------------------------------ |
| `PERSIST`    | Parent save → Child save       |
| `MERGE`      | Parent merge → Child merge     |
| `REMOVE`     | Parent delete → Child delete   |
| `REFRESH`    | Parent refresh → Child refresh |
| `DETACH`     | Parent detach → Child detach   |
| `ALL`        | All of the above               |

For beginners, focus especially on:

```text
PERSIST
MERGE
REMOVE
ALL
```

---

# 11. Complete Mini Project

Let's build a complete example.

We will create:

```text
Database
   ↓
Department table
   ↓
Employee table
   ↓
Hibernate
   ↓
OneToMany
   ↓
Cascade
```

Our relationship will be:

```text
Department 1 -------- * Employee
```

One department can have many employees.

---

# 12. Database Creation

We'll use MySQL.

Open MySQL Workbench or MySQL command line.

Create a database:

```sql
CREATE DATABASE hibernate_cascade_db;
```

Select it:

```sql
USE hibernate_cascade_db;
```

At this point:

```text
MySQL
  |
  └── hibernate_cascade_db
```

We can allow Hibernate to create the tables automatically.

---

# 13. Create Maven Project

Create a Maven project.

Project structure:

```text
hibernate-cascade
│
├── pom.xml
│
└── src
    └── main
        ├── java
        │   └── com/example
        │       ├── model
        │       │   ├── Department.java
        │       │   └── Employee.java
        │       │
        │       ├── util
        │       │   └── HibernateUtil.java
        │       │
        │       └── Main.java
        │
        └── resources
            └── hibernate.cfg.xml
```

---

# 14. pom.xml

Create:

```text
pom.xml
```

Use:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>hibernate-cascade</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>

        <!-- Hibernate ORM -->
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>6.6.5.Final</version>
        </dependency>

        <!-- MySQL Driver -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>9.1.0</version>
        </dependency>

        <!-- Jakarta Persistence API -->
        <dependency>
            <groupId>jakarta.persistence</groupId>
            <artifactId>jakarta.persistence-api</artifactId>
            <version>3.2.0</version>
        </dependency>

    </dependencies>

</project>
```

Maven will download the required libraries.

---

# 15. Hibernate Configuration

Create:

```text
src/main/resources/hibernate.cfg.xml
```

Code:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "https://hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <!-- Database URL -->
        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/hibernate_cascade_db
        </property>

        <!-- Database username -->
        <property name="hibernate.connection.username">
            root
        </property>

        <!-- Database password -->
        <property name="hibernate.connection.password">
            YOUR_PASSWORD
        </property>

        <!-- MySQL Driver -->
        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <!-- Automatically create/update tables -->
        <property name="hibernate.hbm2ddl.auto">
            update
        </property>

        <!-- Display SQL queries -->
        <property name="hibernate.show_sql">
            true
        </property>

        <!-- Format SQL -->
        <property name="hibernate.format_sql">
            true
        </property>

    </session-factory>

</hibernate-configuration>
```

Replace:

```text
YOUR_PASSWORD
```

with your MySQL password.

---

# 16. Department Entity

Create:

```text
Department.java
```

Use:

```java
package com.example.model;

import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "departments")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(
            mappedBy = "department",
            cascade = CascadeType.ALL
    )
    private List<Employee> employees = new ArrayList<>();

    public Department() {
    }

    public Department(String name) {
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Employee> getEmployees() {
        return employees;
    }

    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }

    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);
    }
}
```

---

# 17. Understand the Cascade Code

The most important part is:

```java
@OneToMany(
        mappedBy = "department",
        cascade = CascadeType.ALL
)
```

Let's break it down.

### `@OneToMany`

Means:

```text
One Department
       ↓
Many Employees
```

### `mappedBy = "department"`

This tells Hibernate:

> The `department` field inside Employee owns the relationship mapping.

### `cascade = CascadeType.ALL`

This tells Hibernate:

> Apply persistence operations performed on Department to its Employees.

Therefore:

```text
Department
    |
    ├── Employee
    ├── Employee
    └── Employee
```

and:

```java
session.persist(department);
```

will cascade to the employees.

---

# 18. Employee Entity

Create:

```text
Employee.java
```

Code:

```java
package com.example.model;

import jakarta.persistence.*;

@Entity
@Table(name = "employees")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String designation;

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;

    public Employee() {
    }

    public Employee(String name, String designation) {
        this.name = name;
        this.designation = designation;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDesignation() {
        return designation;
    }

    public void setDesignation(String designation) {
        this.designation = designation;
    }

    public Department getDepartment() {
        return department;
    }

    public void setDepartment(Department department) {
        this.department = department;
    }
}
```

---

# 19. Relationship in Database

Our Java relationship:

```text
Department
@OneToMany
     ↓
Employee
@ManyToOne
```

will result in something conceptually like:

```text
departments
----------------
id
name


employees
----------------
id
name
designation
department_id
```

The foreign key exists in:

```text
employees.department_id
```

because many employees belong to one department.

---

# 20. HibernateUtil

Create:

```text
HibernateUtil.java
```

Code:

```java
package com.example.util;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {

    private static final SessionFactory sessionFactory =
            new Configuration()
                    .configure()
                    .buildSessionFactory();

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
}
```

This creates the Hibernate `SessionFactory`.

---

# 21. Main.java

Now let's test Cascade.

Create:

```text
Main.java
```

Code:

```java
package com.example;

import com.example.model.Department;
import com.example.model.Employee;
import com.example.util.HibernateUtil;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class Main {

    public static void main(String[] args) {

        SessionFactory sessionFactory =
                HibernateUtil.getSessionFactory();

        Session session = sessionFactory.openSession();

        Transaction transaction = session.beginTransaction();

        // Create Department
        Department department =
                new Department("IT");

        // Create Employees
        Employee employee1 =
                new Employee("Rahul", "Java Developer");

        Employee employee2 =
                new Employee("Amit", "Backend Developer");

        Employee employee3 =
                new Employee("Priya", "Software Engineer");

        // Add employees to department
        department.addEmployee(employee1);
        department.addEmployee(employee2);
        department.addEmployee(employee3);

        // Save Department
        session.persist(department);

        transaction.commit();

        session.close();

        sessionFactory.close();
    }
}
```

---

# 22. The Most Important Part

Look at:

```java
department.addEmployee(employee1);
department.addEmployee(employee2);
department.addEmployee(employee3);
```

Then:

```java
session.persist(department);
```

We did **not** write:

```java
session.persist(employee1);
session.persist(employee2);
session.persist(employee3);
```

Why?

Because we configured:

```java
cascade = CascadeType.ALL
```

Therefore:

```text
session.persist(department)
            |
            ↓
       Cascade ALL
            |
       ┌────┼────┐
       ↓    ↓    ↓
      E1   E2   E3
```

Hibernate persists them automatically.

---

# 23. What SQL Does Hibernate Perform?

You may see SQL similar to:

```sql
insert into departments (name) values (?);
```

Then:

```sql
insert into employees
(name, designation, department_id)
values (?, ?, ?);
```

three times.

Conceptually:

```text
INSERT Department

INSERT Employee 1
INSERT Employee 2
INSERT Employee 3
```

This happened because of:

```java
cascade = CascadeType.ALL
```

---

# 24. Test Without Cascade

Now change:

```java
@OneToMany(
        mappedBy = "department",
        cascade = CascadeType.ALL
)
```

to:

```java
@OneToMany(
        mappedBy = "department"
)
```

Now run:

```java
session.persist(department);
```

Hibernate will **not cascade the persist operation** to employees.

So you cannot assume that:

```java
session.persist(department);
```

will automatically persist:

```text
employee1
employee2
employee3
```

This is the fundamental purpose of Cascade.

---

# 25. Cascade vs Relationship

This is a very important beginner concept.

These two things are **not the same**.

### Relationship

```java
@OneToMany
```

answers:

> How are these two entities related?

For example:

```text
Department → Employees
```

### Cascade

```java
cascade = CascadeType.ALL
```

answers:

> Should operations on one entity automatically be propagated to the related entity?

So:

```text
@OneToMany
      ↓
defines relationship

cascade
      ↓
defines operation propagation
```

---

# 26. Cascade Does NOT Mean Foreign Key

Another common mistake is thinking:

```java
cascade = CascadeType.ALL
```

creates the foreign key.

It doesn't.

The relationship mapping creates the association:

```java
@JoinColumn(name = "department_id")
```

Cascade controls operations.

Think:

```text
@JoinColumn
     ↓
Database relationship

cascade
     ↓
Hibernate operation propagation
```

---

# 27. Cascade and orphanRemoval

You will often see:

```java
@OneToMany(
    mappedBy = "department",
    cascade = CascadeType.ALL,
    orphanRemoval = true
)
```

These are related but **not identical**.

### Cascade REMOVE

```text
Delete Department
       ↓
Delete Employees
```

### orphanRemoval

```text
Remove Employee from Department collection
       ↓
Employee can be deleted from database
```

For example:

```java
department.removeEmployee(employee1);
```

With:

```java
orphanRemoval = true
```

Hibernate can delete that employee from the database.

Without `orphanRemoval`, simply removing it from the Java collection does not necessarily mean deleting the database row.

---

# 28. Cascade + orphanRemoval

A common mapping for a true parent-child relationship is:

```java
@OneToMany(
        mappedBy = "department",
        cascade = CascadeType.ALL,
        orphanRemoval = true
)
private List<Employee> employees = new ArrayList<>();
```

This gives you:

```text
Department
    |
    ├── Employee 1
    ├── Employee 2
    └── Employee 3
```

When Department is persisted:

```text
Department
    ↓
Employees persisted
```

When Department is removed:

```text
Department
    ↓
Employees removed
```

When an Employee is removed from the collection:

```text
Department.employees.remove(employee)
                    ↓
             Employee deleted
```

But use this only when the child genuinely belongs to the parent.

---

# 29. One Very Important Rule

Do **not** automatically write:

```java
cascade = CascadeType.ALL
```

on every relationship.

For example:

```text
Student ←→ Course
```

A course can belong to many students.

If you use cascading delete carelessly, you could end up with:

```text
Delete Student
     ↓
Delete Course
     ↓
Other students lose their course
```

That may be completely wrong.

Cascade should represent the **business ownership/lifecycle relationship**, not simply the fact that two entities are related.

---

# 30. Easy Way to Remember

Think of Cascade as a **parent-child instruction**.

```text
PARENT
  |
  | Cascade
  ↓
CHILD
```

### PERSIST

```text
Save Parent
    ↓
Save Child
```

### MERGE

```text
Update/Merge Parent
    ↓
Update/Merge Child
```

### REMOVE

```text
Delete Parent
    ↓
Delete Child
```

### ALL

```text
Save
Update
Delete
Refresh
Detach
  ↓
Child
```

---

# 31. Final Project Structure

Your complete project should look like:

```text
hibernate-cascade
│
├── pom.xml
│
└── src
    └── main
        ├── java
        │   └── com
        │       └── example
        │           │
        │           ├── Main.java
        │           │
        │           ├── model
        │           │   ├── Department.java
        │           │   └── Employee.java
        │           │
        │           └── util
        │               └── HibernateUtil.java
        │
        └── resources
            └── hibernate.cfg.xml
```

---

# 32. The One Line You Should Remember

The most important line in this entire tutorial is:

```java
cascade = CascadeType.ALL
```

It means:

> **When I perform supported operations on the parent entity, Hibernate automatically propagates those operations to the associated child entities.**

And remember:

```text
@OneToMany
     ↓
WHAT is the relationship?

cascade
     ↓
WHAT should happen to the related entity
when I operate on this entity?
```

That distinction is the key to understanding Hibernate Cascade.

# Hibernate ORM: Fetch Type

**Fetch Type** is one of the most important concepts after understanding relationships and Cascade.

If **Cascade** answers:

> “What should Hibernate do to the related object when I perform an operation?”

Then **Fetch Type** answers:

> **“When should Hibernate load the related object from the database?”**

There are two main fetch types:

```java
FetchType.LAZY
FetchType.EAGER
```

---

# 1. First Understand the Problem

Suppose we have:

```text
Department
    |
    ├── Employee 1
    ├── Employee 2
    ├── Employee 3
    └── Employee 4
```

We execute:

```java
Department department = session.get(Department.class, 1L);
```

The question is:

### Should Hibernate immediately load all employees?

```text
Database
   ↓
Department
   ↓
Employees
```

Or should Hibernate initially load **only the Department**, and load employees later if we actually need them?

That's what Fetch Type controls.

---

# 2. Two Fetch Types

## LAZY

```java
FetchType.LAZY
```

Means:

> **Don't load the related entity immediately. Load it only when it is actually needed.**

---

## EAGER

```java
FetchType.EAGER
```

Means:

> **Load the related entity immediately along with the entity being fetched.**

---

# 3. Simple Real-Life Example

Imagine you open a student profile.

```text
Student
   |
   ├── Name
   ├── Age
   ├── Email
   ├── Courses
   ├── Attendance
   ├── Assignments
   └── Certificates
```

Suppose you only want:

```text
Student Name
Student Email
```

Do you really need Hibernate to immediately load:

```text
100 Courses
500 Attendance records
50 Assignments
20 Certificates
```

Probably not.

That's where:

```java
FetchType.LAZY
```

is useful.

---

# 4. FetchType.LAZY

Let's use our previous example.

```java
@OneToMany(
    mappedBy = "department",
    cascade = CascadeType.ALL,
    fetch = FetchType.LAZY
)
private List<Employee> employees;
```

Now execute:

```java
Department department =
        session.get(Department.class, 1L);
```

Hibernate initially loads the Department.

Conceptually:

```text
SELECT *
FROM departments
WHERE id = 1;
```

It does **not necessarily load employees immediately**.

The employees are loaded when you actually access:

```java
department.getEmployees();
```

Conceptually:

```text
Get Department
      ↓
Department loaded
      ↓
Employee NOT loaded yet
      ↓
getEmployees()
      ↓
Employees loaded
```

---

# 5. Why Is LAZY Useful?

Imagine a department has:

```text
10,000 employees
```

You execute:

```java
Department department =
        session.get(Department.class, 1L);
```

If employees are eagerly loaded, Hibernate may load all 10,000 employees even if you don't need them.

With LAZY:

```text
Load Department
       ↓
Don't load 10,000 employees yet
       ↓
Load them only if needed
```

This can save:

* Database work
* Memory
* Network traffic
* Application processing

---

# 6. FetchType.EAGER

Now:

```java
@OneToMany(
    mappedBy = "department",
    fetch = FetchType.EAGER
)
private List<Employee> employees;
```

When you do:

```java
Department department =
        session.get(Department.class, 1L);
```

Hibernate is instructed to fetch the employees eagerly as part of loading the Department.

Conceptually:

```text
Get Department
      ↓
Department loaded
      ↓
Employees loaded immediately
```

---

# 7. Very Simple Difference

Remember this:

### LAZY

```text
I need Department
       ↓
Load Department
       ↓
Employee?
       ↓
Not yet
       ↓
I ask for Employees
       ↓
Load Employees
```

### EAGER

```text
I need Department
       ↓
Load Department
       +
Load Employees
```

---

# 8. Complete Example

Let's modify our previous `Department.java`.

```java
package com.example.model;

import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "departments")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(
            mappedBy = "department",
            cascade = CascadeType.ALL,
            fetch = FetchType.LAZY
    )
    private List<Employee> employees = new ArrayList<>();

    public Department() {
    }

    public Department(String name) {
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Employee> getEmployees() {
        return employees;
    }

    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }

    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);
    }
}
```

The important line is:

```java
fetch = FetchType.LAZY
```

---

# 9. What Happens Internally?

Suppose:

```java
Department department =
        session.get(Department.class, 1L);
```

Hibernate executes something like:

```sql
SELECT
    id,
    name
FROM departments
WHERE id = 1;
```

At this point, Hibernate has the Department.

Now:

```java
System.out.println(department.getName());
```

No need to load employees.

But when you do:

```java
System.out.println(
    department.getEmployees()
);
```

Hibernate may execute another query:

```sql
SELECT
    id,
    name,
    designation,
    department_id
FROM employees
WHERE department_id = 1;
```

That's **Lazy Loading**.

---

# 10. Proxy — Important Concept

With LAZY loading, Hibernate often uses a **proxy** or persistent collection wrapper.

You might imagine:

```text
Department object
       |
       └── employees
              ↓
         "I'll load later"
```

Hibernate essentially keeps enough information to load the employees when they're accessed.

You don't normally have to manually write the SQL.

---

# 11. LAZY Is Usually Preferred

For most relationships, especially collections, LAZY fetching is generally a good default.

For example:

```java
@OneToMany(fetch = FetchType.LAZY)
```

and:

```java
@ManyToMany(fetch = FetchType.LAZY)
```

This prevents unnecessary data from being loaded.

However, **LAZY doesn't mean “one query only”**. If you access the collection, Hibernate may issue another SQL query.

---

# 12. Default Fetch Types

This is very important for exams/interviews.

JPA has defaults based on the relationship annotation.

| Relationship  | Default Fetch Type |
| ------------- | ------------------ |
| `@OneToOne`   | `EAGER`            |
| `@ManyToOne`  | `EAGER`            |
| `@OneToMany`  | `LAZY`             |
| `@ManyToMany` | `LAZY`             |

So:

```java
@OneToMany
```

defaults to:

```java
FetchType.LAZY
```

while:

```java
@ManyToOne
```

defaults to:

```java
FetchType.EAGER
```

---

# 13. Example with ManyToOne

Our Employee has:

```java
@ManyToOne
@JoinColumn(name = "department_id")
private Department department;
```

Because `@ManyToOne` defaults to EAGER, conceptually:

```text
Employee
   |
   └── Department
```

may cause the Department to be fetched eagerly.

You can explicitly specify:

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "department_id")
private Department department;
```

Now you're telling Hibernate:

> Don't load the Department until I actually need it.

---

# 14. Fetch Type vs Cascade

This is **extremely important**.

Many beginners confuse them.

### Cascade

```java
cascade = CascadeType.ALL
```

controls:

```text
SAVE
UPDATE/MERGE
DELETE
REFRESH
DETACH
```

of related entities.

### Fetch

```java
fetch = FetchType.LAZY
```

controls:

```text
WHEN should related data be loaded?
```

So:

```text
CASCADE
   ↓
What happens to related entity?

FETCH
   ↓
When is related entity loaded?
```

---

# 15. Example Together

You can have:

```java
@OneToMany(
    mappedBy = "department",
    cascade = CascadeType.ALL,
    fetch = FetchType.LAZY
)
private List<Employee> employees;
```

Here:

```text
cascade = CascadeType.ALL
```

means:

> Propagate supported operations to employees.

And:

```text
fetch = FetchType.LAZY
```

means:

> Don't load employees until they're needed.

These two settings solve **different problems**.

---

# 16. Fetch Type Does NOT Mean Number of Queries

This is another important point.

Some beginners think:

```text
LAZY = one query
EAGER = two queries
```

That's incorrect.

Fetch type tells Hibernate **when associated data should be available**, not a fixed number of SQL queries.

Depending on the query and mapping, Hibernate can use:

* separate SELECTs
* JOINs
* batch fetching
* subselect fetching
* entity graphs
* fetch joins

So don't memorize:

```text
LAZY = 2 queries
EAGER = 1 query
```

That is not a reliable rule.

---

# 17. The N+1 Problem

Now we reach one of the most important Hibernate concepts.

Suppose:

```text
100 Departments
```

and each department has employees.

You load:

```java
List<Department> departments =
        session.createQuery(
            "from Department",
            Department.class
        ).getResultList();
```

Then you do:

```java
for (Department department : departments) {

    System.out.println(
        department.getEmployees()
    );
}
```

With lazy loading, Hibernate could do:

```text
1 query
   ↓
Load 100 Departments

Then:

100 additional queries
   ↓
Load Employees for each Department
```

Total:

```text
1 + 100 = 101 queries
```

This is called the:

# N+1 Query Problem

---

# 18. How to Solve N+1?

One common solution is a **fetch join**.

For example:

```java
List<Department> departments =
        session.createQuery(
            "SELECT DISTINCT d " +
            "FROM Department d " +
            "LEFT JOIN FETCH d.employees",
            Department.class
        ).getResultList();
```

The idea is:

```text
Department
     +
Employees
     ↓
Fetch together
```

This can reduce unnecessary repeated queries.

But fetch joins need to be used thoughtfully, especially with multiple collections.

---

# 19. LazyInitializationException

This is another famous Hibernate problem.

Suppose:

```java
Session session = sessionFactory.openSession();

Department department =
        session.get(Department.class, 1L);

session.close();

System.out.println(
        department.getEmployees()
);
```

You may get:

```text
LazyInitializationException
```

Why?

Because:

```text
Session open
    ↓
Department loaded
    ↓
Employees NOT loaded
    ↓
Session closed
    ↓
getEmployees()
    ↓
Hibernate needs database access
    ↓
But Session is closed
    ↓
LazyInitializationException
```

That's one of the most important things to understand about LAZY loading.

---

# 20. How to Avoid It

One simple approach is to access the collection while the session is still open:

```java
Session session = sessionFactory.openSession();

Department department =
        session.get(Department.class, 1L);

System.out.println(
        department.getEmployees()
);

session.close();
```

The employees are initialized while Hibernate still has an active session.

In real applications, better solutions often involve designing the query/service boundary properly, such as using fetch joins or DTO queries rather than keeping sessions open unnecessarily.

---

# 21. Simple Demonstration

Try this:

```java
Session session = sessionFactory.openSession();

Department department =
        session.get(Department.class, 1L);

System.out.println("Department loaded");

System.out.println(department.getName());

System.out.println("Now accessing employees...");

System.out.println(
        department.getEmployees().size()
);

session.close();
```

With LAZY, you'll generally see the Department query first.

Then when:

```java
department.getEmployees().size()
```

is executed, Hibernate loads the employees.

---

# 22. EAGER Demonstration

Change:

```java
fetch = FetchType.LAZY
```

to:

```java
fetch = FetchType.EAGER
```

Now:

```java
Department department =
        session.get(Department.class, 1L);
```

Hibernate is instructed to have the employees available immediately.

So the conceptual flow becomes:

```text
session.get(Department)
        ↓
Department
        +
Employees
```

---

# 23. Should You Always Use LAZY?

For a beginner, a good mental rule is:

> **Prefer LAZY fetching unless you have a specific reason to fetch the relationship eagerly.**

Especially for collections:

```java
@OneToMany
@ManyToMany
```

LAZY is usually much safer from a performance perspective.

But don't blindly change every relationship to LAZY without understanding the application/query requirements.

---

# 24. Complete Mapping

A realistic version of our project can look like:

### Department

```java
@OneToMany(
        mappedBy = "department",
        cascade = CascadeType.ALL,
        fetch = FetchType.LAZY,
        orphanRemoval = true
)
private List<Employee> employees = new ArrayList<>();
```

### Employee

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "department_id")
private Department department;
```

Now we have:

```text
Department
     |
     | OneToMany
     |
     | Cascade ALL
     |
     | Fetch LAZY
     |
     ↓
Employees
     |
     | ManyToOne
     |
     | Fetch LAZY
     |
     ↓
Department
```

---

# 25. Final Comparison

| Concept         | Question it answers                    |
| --------------- | -------------------------------------- |
| `@OneToMany`    | What is the relationship?              |
| `@ManyToOne`    | What is the relationship?              |
| `cascade`       | What operations should propagate?      |
| `fetch`         | When should related data be loaded?    |
| `LAZY`          | Load when needed                       |
| `EAGER`         | Load immediately                       |
| `orphanRemoval` | Delete child when it becomes an orphan |

---

# 26. One-Line Memory Trick

Remember:

```text
CASCADE = WHAT happens?
FETCH   = WHEN does it load?
```

For example:

```java
@OneToMany(
    cascade = CascadeType.ALL,
    fetch = FetchType.LAZY
)
```

means:

> **Cascade:** propagate operations to the employees.

> **Lazy:** don't load the employees until they're needed.

And the two most important values to remember are:

```java
FetchType.LAZY
FetchType.EAGER
```

**For Hibernate ORM, understanding `LAZY` + the N+1 problem + `LazyInitializationException` is much more important than simply memorizing the two enum values.**

