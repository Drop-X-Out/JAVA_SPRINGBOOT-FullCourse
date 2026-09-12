# Object-Relational Mapping — `@ManyToOne`

## What we will build

We will create:

```text
Employee
   |
   | Many
   |
   ↓
Department
   |
   | One
```

Example:

```text
Rahul ───────┐
Aman ────────┤
Priya ───────┼──→ IT Department
Neha ────────┘
```

Four employees belong to one department.

Therefore:

```text
Many Employees → One Department
```

In Java:

```java
@ManyToOne
private Department department;
```

In the database:

```text
employee.department_id
             |
             ↓
       department.id
```

---

# 1. First understand the real-world relationship

Imagine a company.

There are departments:

```text
IT
HR
Finance
```

And employees:

```text
Rahul → IT
Aman  → IT
Priya → IT
Neha  → HR
```

Look at Rahul.

Rahul belongs to **one department**:

```text
Rahul → IT
```

Aman also belongs to **one department**:

```text
Aman → IT
```

Priya:

```text
Priya → IT
```

So from the employee's perspective:

```text
Many Employees
      ↓
One Department
```

That's exactly:

```java
@ManyToOne
```

---

# 2. Database design first

Before Java, let's understand the database.

We'll create two tables:

```text
department
----------------
id
name
```

and:

```text
employee
----------------
id
name
salary
department_id
```

The important column is:

```text
department_id
```

This will be a **foreign key**.

Relationship:

```text
employee.department_id
          ↓
department.id
```

---

# 3. Create the database

Open MySQL Workbench.

Run:

```sql
CREATE DATABASE hibernate_manytoone;
```

Then:

```sql
USE hibernate_manytoone;
```

Check:

```sql
SHOW DATABASES;
```

You should see:

```text
hibernate_manytoone
```

---

# 4. Create the Department table

Run:

```sql
CREATE TABLE department (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL
);
```

Our table looks like:

```text
department

id        name
--------------------
1         IT
2         HR
3         Finance
```

---

# 5. Create Employee table

Now:

```sql
CREATE TABLE employee (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    salary DOUBLE,
    department_id INT,

    CONSTRAINT fk_employee_department
        FOREIGN KEY (department_id)
        REFERENCES department(id)
);
```

The important part is:

```sql
FOREIGN KEY (department_id)
REFERENCES department(id)
```

This means:

> `employee.department_id` must refer to an existing department.

---

# 6. Insert department data

Let's manually insert some departments.

```sql
INSERT INTO department (name)
VALUES ('IT');

INSERT INTO department (name)
VALUES ('HR');

INSERT INTO department (name)
VALUES ('Finance');
```

Check:

```sql
SELECT * FROM department;
```

You should get something like:

```text
id    name
--------------
1     IT
2     HR
3     Finance
```

---

# 7. Insert employee data

Now:

```sql
INSERT INTO employee (name, salary, department_id)
VALUES ('Rahul', 50000, 1);

INSERT INTO employee (name, salary, department_id)
VALUES ('Aman', 60000, 1);

INSERT INTO employee (name, salary, department_id)
VALUES ('Priya', 55000, 1);

INSERT INTO employee (name, salary, department_id)
VALUES ('Neha', 45000, 2);
```

Check:

```sql
SELECT * FROM employee;
```

You should get:

```text
id    name     salary     department_id
---------------------------------------
1     Rahul    50000      1
2     Aman     60000      1
3     Priya    55000      1
4     Neha     45000      2
```

Now look at this:

```text
department

1 → IT
2 → HR
3 → Finance
```

Employees:

```text
Rahul → 1
Aman  → 1
Priya → 1
Neha  → 2
```

Therefore:

```text
          IT
          ↑
     ┌────┼────┐
     │    │    │
   Rahul Aman Priya


          HR
          ↑
          │
         Neha
```

This is **Many-to-One**.

---

# 8. Verify with JOIN

Run:

```sql
SELECT
    e.id AS employee_id,
    e.name AS employee_name,
    e.salary,
    d.id AS department_id,
    d.name AS department_name
FROM employee e
JOIN department d
ON e.department_id = d.id;
```

Result:

```text
employee_id | employee_name | salary | department_id | department_name
----------------------------------------------------------------------
1           | Rahul         | 50000  | 1             | IT
2           | Aman          | 60000  | 1             | IT
3           | Priya         | 55000  | 1             | IT
4           | Neha          | 45000  | 2             | HR
```

Notice:

```text
Rahul → IT
Aman  → IT
Priya → IT
```

Three employees point to the same department.

That's:

```text
Many → One
```

---

# 9. Now create the Maven project

Create:

```text
hibernate-manytoone
```

Project structure:

```text
hibernate-manytoone
│
├── pom.xml
│
└── src
    └── main
        ├── java
        │   └── com.example
        │       ├── Main.java
        │       │
        │       └── model
        │           ├── Employee.java
        │           └── Department.java
        │
        └── resources
            └── hibernate.cfg.xml
```

---

# 10. pom.xml

Replace your `pom.xml` with:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>

    <artifactId>hibernate-manytoone</artifactId>

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
            <version>6.6.26.Final</version>
        </dependency>

        <!-- MySQL JDBC Driver -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>9.4.0</version>
        </dependency>

        <!-- Jakarta Persistence -->
        <dependency>
            <groupId>jakarta.persistence</groupId>
            <artifactId>jakarta.persistence-api</artifactId>
            <version>3.2.0</version>
        </dependency>

        <!-- Logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>2.0.17</version>
        </dependency>

    </dependencies>

</project>
```

---

# 11. What Maven is doing

Our Java code needs external libraries.

For example:

```text
Java
 ↓
Hibernate
 ↓
MySQL Driver
 ↓
MySQL
```

Maven downloads those libraries automatically from repositories.

The file:

```text
pom.xml
```

is basically the project's dependency/configuration file.

---

# 12. Create `Department.java`

Create:

```text
src/main/java/com/example/model/Department.java
```

Paste:

```java
package com.example.model;

import jakarta.persistence.*;

@Entity
@Table(name = "department")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(name = "name")
    private String name;

    public Department() {
    }

    public Department(String name) {
        this.name = name;
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

    @Override
    public String toString() {
        return "Department{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

Notice something important:

**There is no `@ManyToOne` here.**

Why?

Because Department is the **one side**.

Our focus is:

```text
Many Employees → One Department
```

The `Many` side is Employee.

---

# 13. Understand `@Entity`

```java
@Entity
```

tells Hibernate:

> This Java class represents a database entity.

So:

```text
Department.java
       ↓
Hibernate
       ↓
department table
```

---

# 14. Understand `@Table`

```java
@Table(name = "department")
```

means:

> Map this class to the `department` table.

Therefore:

```java
Department
```

maps to:

```text
department
```

---

# 15. Understand `@Id`

```java
@Id
private int id;
```

means:

> This field represents the primary key.

Database:

```sql
id INT PRIMARY KEY
```

Java:

```java
@Id
private int id;
```

---

# 16. Now create Employee.java

This is where the important part happens.

Create:

```text
src/main/java/com/example/model/Employee.java
```

Paste:

```java
package com.example.model;

import jakarta.persistence.*;

@Entity
@Table(name = "employee")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(name = "name")
    private String name;

    @Column(name = "salary")
    private double salary;

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;

    public Employee() {
    }

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
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

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }

    public Department getDepartment() {
        return department;
    }

    public void setDepartment(Department department) {
        this.department = department;
    }

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", salary=" + salary +
                '}';
    }
}
```

---

# 17. The most important part

Look at:

```java
@ManyToOne
@JoinColumn(name = "department_id")
private Department department;
```

This is the complete Many-to-One mapping.

Let's understand each piece.

---

# 18. `@ManyToOne`

```java
@ManyToOne
```

means:

> Many Employee objects can be associated with one Department object.

For example:

```text
Employee Rahul ──┐
Employee Aman  ──┤
Employee Priya ──┼──→ Department IT
Employee Neha  ──┘
```

From Employee's perspective:

```text
Employee
   ↓
Department
```

Each Employee has **one** Department.

But the same Department can be referenced by **many** Employees.

Therefore:

```java
@ManyToOne
```

---

# 19. Why isn't it `@OneToMany`?

This is a very important question.

Ask:

### What does one Employee have?

One employee belongs to:

```text
ONE Department
```

Therefore:

```java
@ManyToOne
```

Ask the reverse:

### What does one Department have?

Potentially:

```text
MANY Employees
```

Therefore, if we model the reverse side:

```java
@OneToMany
```

So:

```text
Employee side:
@ManyToOne

Department side:
@OneToMany
```

---

# 20. Understand `@JoinColumn`

We wrote:

```java
@JoinColumn(name = "department_id")
```

This tells Hibernate:

> The foreign-key column for this relationship is `department_id`.

Our database:

```text
employee

id
name
salary
department_id
```

Our Java class:

```java
private Department department;
```

Hibernate connects:

```text
Employee.department
        ↓
employee.department_id
        ↓
department.id
```

---

# 21. This is the most important diagram

Keep this in your notes:

```text
JAVA
────────────────────────

Employee
    |
    | @ManyToOne
    |
    ↓
Department


DATABASE
────────────────────────

employee
--------------------------------
id
name
salary
department_id  ← FOREIGN KEY
                    |
                    ↓
              department.id
```

That's basically the entire Many-to-One concept.

---

# 22. Why is Employee the owning side?

Because Employee contains:

```java
@JoinColumn(name = "department_id")
```

Therefore Employee controls the foreign key.

We can say:

```text
Employee = owning side
Department = referenced side
```

The database relationship is actually stored by:

```text
employee.department_id
```

---

# 23. Create `hibernate.cfg.xml`

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

        <!-- MySQL Driver -->
        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <!-- Database URL -->
        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/hibernate_manytoone
        </property>

        <!-- MySQL Username -->
        <property name="hibernate.connection.username">
            root
        </property>

        <!-- MySQL Password -->
        <property name="hibernate.connection.password">
            YOUR_MYSQL_PASSWORD
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

        <!-- Entity Classes -->
        <mapping class="com.example.model.Department"/>
        <mapping class="com.example.model.Employee"/>

    </session-factory>

</hibernate-configuration>
```

Replace:

```text
YOUR_MYSQL_PASSWORD
```

with your MySQL password.

---

# 24. Why do we specify entity classes?

These:

```xml
<mapping class="com.example.model.Department"/>
<mapping class="com.example.model.Employee"/>
```

tell Hibernate:

```text
These are my entities.
```

Hibernate then understands:

```text
Department → department table
Employee   → employee table
```

---

# 25. Create Main.java

Now:

```text
src/main/java/com/example/Main.java
```

Paste:

```java
package com.example;

import com.example.model.Department;
import com.example.model.Employee;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class Main {

    public static void main(String[] args) {

        // Create Hibernate configuration
        Configuration configuration = new Configuration();

        // Read hibernate.cfg.xml
        configuration.configure();

        // Create SessionFactory
        SessionFactory sessionFactory =
                configuration.buildSessionFactory();

        // Open a Hibernate session
        Session session = sessionFactory.openSession();

        // Start transaction
        session.beginTransaction();

        // Create a Department
        Department department = new Department("IT");

        // Save Department first
        session.persist(department);

        // Create Employees
        Employee employee1 =
                new Employee("Rahul", 50000);

        Employee employee2 =
                new Employee("Aman", 60000);

        Employee employee3 =
                new Employee("Priya", 55000);

        // Assign same Department to multiple Employees
        employee1.setDepartment(department);
        employee2.setDepartment(department);
        employee3.setDepartment(department);

        // Save Employees
        session.persist(employee1);
        session.persist(employee2);
        session.persist(employee3);

        // Commit transaction
        session.getTransaction().commit();

        // Close session
        session.close();

        // Close SessionFactory
        sessionFactory.close();

        System.out.println("Many-to-One data saved successfully!");
    }
}
```

---

# 26. Understand the Main method deeply

Let's follow the program.

First:

```java
Configuration configuration = new Configuration();
```

We create Hibernate configuration.

Then:

```java
configuration.configure();
```

Hibernate reads:

```text
hibernate.cfg.xml
```

Then:

```java
SessionFactory sessionFactory =
        configuration.buildSessionFactory();
```

Hibernate creates the SessionFactory.

Then:

```java
Session session =
        sessionFactory.openSession();
```

We open a session.

Then:

```java
session.beginTransaction();
```

We start the transaction.

---

# 27. Create Department

```java
Department department =
        new Department("IT");
```

At this point:

```text
Java memory:

department
    |
    ↓
Department
name = IT
```

It has not necessarily been inserted into the database yet.

---

# 28. Persist Department

```java
session.persist(department);
```

Hibernate makes the object persistent.

Eventually something similar to:

```sql
INSERT INTO department (name)
VALUES ('IT');
```

will execute.

Suppose MySQL generates:

```text
id = 4
```

Now the Java object is associated with:

```text
Department
id = 4
name = IT
```

---

# 29. Create employees

```java
Employee employee1 =
        new Employee("Rahul", 50000);
```

We have:

```text
Employee
name = Rahul
salary = 50000
```

Similarly:

```java
Employee employee2 =
        new Employee("Aman", 60000);
```

and:

```java
Employee employee3 =
        new Employee("Priya", 55000);
```

---

# 30. Now establish Many-to-One relationship

This is the most important code:

```java
employee1.setDepartment(department);
employee2.setDepartment(department);
employee3.setDepartment(department);
```

Think about it.

We have:

```text
employee1 → department
employee2 → department
employee3 → department
```

And all three variables point to the **same Department object**.

So:

```text
Rahul ──┐
Aman  ──┼──→ IT
Priya ──┘
```

That's:

```text
MANY → ONE
```

---

# 31. What does `setDepartment()` actually do?

Our method is:

```java
public void setDepartment(Department department) {
    this.department = department;
}
```

Suppose:

```java
employee1.setDepartment(department);
```

This means:

```text
employee1.department
       ↓
   Department IT
```

Then:

```java
employee2.setDepartment(department);
```

means:

```text
employee2.department
       ↓
   Department IT
```

And:

```java
employee3.setDepartment(department);
```

means:

```text
employee3.department
       ↓
   Department IT
```

So:

```text
                    Department IT
                         ↑
              ┌──────────┼──────────┐
              │          │          │
              │          │          │
            Rahul       Aman       Priya
```

---

# 32. Persist employees

Then:

```java
session.persist(employee1);
session.persist(employee2);
session.persist(employee3);
```

Hibernate knows:

```text
employee1.department = IT
employee2.department = IT
employee3.department = IT
```

Therefore it can store:

```text
employee.department_id = IT.id
```

---

# 33. What SQL does Hibernate generate?

Conceptually:

```sql
INSERT INTO department
(name)
VALUES
('IT');
```

Suppose:

```text
IT.id = 4
```

Then:

```sql
INSERT INTO employee
(name, salary, department_id)
VALUES
('Rahul', 50000, 4);
```

Then:

```sql
INSERT INTO employee
(name, salary, department_id)
VALUES
('Aman', 60000, 4);
```

Then:

```sql
INSERT INTO employee
(name, salary, department_id)
VALUES
('Priya', 55000, 4);
```

The database becomes:

```text
department

id    name
------------
4     IT
```

and:

```text
employee

id    name     salary    department_id
---------------------------------------
5     Rahul    50000     4
6     Aman     60000     4
7     Priya    55000     4
```

---

# 34. Why must Department be saved first?

In our example:

```java
session.persist(department);
```

comes before:

```java
session.persist(employee1);
```

because:

```text
employee.department_id
```

is a foreign key.

The referenced department should exist.

Conceptually:

```text
Department
id = 4
   ↑
   |
   |
Employee.department_id = 4
```

The foreign key cannot point to a nonexistent department.

Hibernate can sometimes handle entity ordering automatically when relationships/cascades are configured appropriately, but for a **first-time learner**, explicitly saving the parent first makes the foreign-key flow very easy to understand.

---

# 35. What if we don't assign a department?

Suppose we do:

```java
Employee employee =
        new Employee("Rahul", 50000);

session.persist(employee);
```

We never do:

```java
employee.setDepartment(department);
```

Then:

```text
employee.department = null
```

So Hibernate may insert:

```text
department_id = NULL
```

unless your database mapping/schema makes the relationship mandatory.

---

# 36. Make the relationship mandatory

If every employee must have a department, you can write:

```java
@ManyToOne(optional = false)
@JoinColumn(name = "department_id", nullable = false)
private Department department;
```

Then conceptually:

```text
Every Employee
      ↓
MUST have
      ↓
Department
```

And database:

```text
department_id NOT NULL
```

This is useful in real applications.

---

# 37. Complete Employee class with mandatory department

For learning, you can use:

```java
package com.example.model;

import jakarta.persistence.*;

@Entity
@Table(name = "employee")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(name = "name", nullable = false)
    private String name;

    @Column(name = "salary")
    private double salary;

    @ManyToOne(optional = false)
    @JoinColumn(name = "department_id", nullable = false)
    private Department department;

    public Employee() {
    }

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
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

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }

    public Department getDepartment() {
        return department;
    }

    public void setDepartment(Department department) {
        this.department = department;
    }

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", salary=" + salary +
                '}';
    }
}
```

---

# 38. Now let's retrieve data

Saving data is only half of ORM.

The really interesting part is retrieving it.

Suppose we want to find employee with ID `1`.

We can write:

```java
Employee employee =
        session.find(Employee.class, 1);
```

Then:

```java
System.out.println(employee);
```

We might get:

```text
Employee{id=1, name='Rahul', salary=50000.0}
```

---

# 39. Get the department of an employee

Now:

```java
Department department =
        employee.getDepartment();
```

Then:

```java
System.out.println(department);
```

Result:

```text
Department{id=1, name='IT'}
```

This is where ORM becomes powerful.

You don't manually perform:

```sql
SELECT *
FROM department
WHERE id = 1;
```

Instead:

```java
employee.getDepartment();
```

Hibernate manages the object relationship.

---

# 40. Complete retrieval example

Inside a transaction/session:

```java
Employee employee =
        session.find(Employee.class, 1);

System.out.println("Employee: "
        + employee.getName());

System.out.println("Department: "
        + employee.getDepartment().getName());
```

Output:

```text
Employee: Rahul
Department: IT
```

The relationship is represented directly in Java.

---

# 41. Very important ORM idea

Without ORM:

```text
Employee
     ↓
department_id
     ↓
SQL query
     ↓
Department row
```

With ORM:

```text
Employee
     ↓
employee.getDepartment()
     ↓
Department object
```

Hibernate handles the relational database details.

---

# 42. Lazy loading

`@ManyToOne` is commonly lazy-loaded in modern JPA/Hibernate usage when configured/used appropriately, although exact behavior should not be assumed blindly from the annotation alone.

The idea of lazy loading is:

Instead of immediately loading:

```text
Employee
+
Department
```

Hibernate can initially load:

```text
Employee
```

and fetch the Department when you actually access:

```java
employee.getDepartment()
```

Conceptually:

```text
session.find(Employee.class, 1)
             ↓
       Employee loaded
             ↓
   getDepartment()
             ↓
      Department loaded
```

This can save unnecessary database work.

---

# 43. One-to-Many vs Many-to-One

This is where students often get confused.

Imagine:

```text
Rahul → IT
Aman  → IT
Priya → IT
```

### From Employee's perspective

Each employee has:

```text
ONE Department
```

Therefore:

```java
@ManyToOne
```

### From Department's perspective

One department has:

```text
MANY Employees
```

Therefore:

```java
@OneToMany
```

So these are two sides of the same relationship.

```text
           ONE
        Department
             ↑
             |
             |
          MANY
        Employees
```

---

# 44. If we add the reverse relationship

We could modify Department:

```java
@OneToMany(mappedBy = "department")
private List<Employee> employees;
```

Then:

```text
Department
     |
     | @OneToMany
     ↓
Employee
     |
     | @ManyToOne
     ↓
Department
```

This is called a **bidirectional relationship**.

But if the requirement is simply:

> Employee should know its Department

then you don't necessarily need the reverse collection.

A unidirectional Many-to-One is perfectly valid:

```text
Employee → Department
```

---

# 45. Unidirectional Many-to-One

Our current project is essentially:

```text
Employee
   |
   | @ManyToOne
   ↓
Department
```

Department doesn't have:

```java
List<Employee>
```

Therefore:

```java
department.getEmployees()
```

doesn't exist.

But:

```java
employee.getDepartment()
```

does exist.

This is **unidirectional**.

---

# 46. Bidirectional Many-to-One relationship

If we add to Department:

```java
@OneToMany(mappedBy = "department")
private List<Employee> employees;
```

then we have:

```text
Employee ─────────→ Department
   ↑                    |
   |                    |
   └────────────────────┘
```

Employee knows Department.

Department knows Employees.

---

# 47. The key difference between `mappedBy` and `JoinColumn`

Remember this carefully.

### `@JoinColumn`

```java
@JoinColumn(name = "department_id")
```

means:

> This side owns/stores the foreign key.

### `mappedBy`

```java
mappedBy = "department"
```

means:

> The relationship is already managed by the `department` field on the other entity.

For Many-to-One, the owning side is normally the side with:

```java
@ManyToOne
@JoinColumn(...)
```

---

# 48. Complete project

Your project should now look like:

```text
hibernate-manytoone
│
├── pom.xml
│
└── src
    └── main
        │
        ├── java
        │   └── com
        │       └── example
        │           │
        │           ├── Main.java
        │           │
        │           └── model
        │               │
        │               ├── Department.java
        │               └── Employee.java
        │
        └── resources
            └── hibernate.cfg.xml
```

---

# 49. Complete flow of the application

You should now be able to visualize the entire project:

```text
                    JAVA
                     |
                     ↓
             Employee object
                     |
                     |
              @ManyToOne
                     |
                     ↓
            Department object
                     |
                     ↓
                  Hibernate
                     |
                     ↓
                  SQL/JDBC
                     |
                     ↓
                  MySQL
                     |
          ┌──────────┴──────────┐
          ↓                     ↓
     department              employee
     ──────────              ────────
     id                      id
     name                    name
                             salary
                             department_id
                                  |
                                  |
                                  ↓
                            department.id
```

---

# 50. What you absolutely need to remember

For **Many-to-One**, memorize this:

```java
@ManyToOne
@JoinColumn(name = "department_id")
private Department department;
```

Meaning:

```text
Many Employee objects
          ↓
    One Department
```

Database:

```text
employee.department_id
          ↓
    department.id
```

Java:

```text
Employee.department
          ↓
     Department
```

---

# 51. Final cheat sheet

| Concept              | Meaning                           |
| -------------------- | --------------------------------- |
| `@Entity`            | Java class is a Hibernate entity  |
| `@Table`             | Maps entity to table              |
| `@Id`                | Primary key                       |
| `@GeneratedValue`    | ID generated automatically        |
| `@Column`            | Maps field to column              |
| `@ManyToOne`         | Many objects reference one object |
| `@JoinColumn`        | Specifies foreign-key column      |
| `SessionFactory`     | Creates Hibernate sessions        |
| `Session`            | Used to interact with database    |
| `beginTransaction()` | Starts transaction                |
| `persist()`          | Makes new object persistent       |
| `commit()`           | Commits database changes          |
| `find()`             | Retrieves entity by primary key   |
| `getDepartment()`    | Accesses related Department       |

And the single most important mental model is:

```text
                 ONE
              Department
                  ↑
                  |
       ┌──────────┼──────────┐
       |          |          |
       |          |          |
     Rahul       Aman       Priya
       |          |          |
       └──────────┴──────────┘
                 MANY


Database:

employee.department_id
            ↓
       department.id
```

# Object-Relational Mapping — `@ManyToMany`

## 1. First understand Many-to-Many

Imagine a college.

A **Student** can enroll in many **Courses**.

And one **Course** can have many **Students**.

For example:

```text
Student                  Course

Rahul  ───────────────→  Java
   │                    /
   ├──────────────────→ DBMS
   │
   └──────────────────→ Spring


Aman ─────────────────→ Java
   │
   └──────────────────→ Python
```

So:

```text
Many Students ↔ Many Courses
```

This is:

```java
@ManyToMany
```

---

# 2. Why Many-to-Many is different

With One-to-Many, we could store the foreign key directly:

```text
employee
-------------------
id
name
department_id
```

But with Many-to-Many, that doesn't work.

Why?

Suppose:

```text
Rahul → Java
Rahul → DBMS
Rahul → Spring
```

Rahul belongs to **many courses**.

If we put:

```text
course_id
```

inside Student:

```text
student
--------------------------------
id | name | course_id
```

we would need:

```text
1 | Rahul | 1,2,3
```

That's not proper relational database design.

The same problem happens on the Course side.

Therefore, Many-to-Many requires a **third table**.

---

# 3. The third table — Join Table

Our database will have:

```text
student
----------------
id
name


course
----------------
id
name


student_course
----------------
student_id
course_id
```

The third table is called a:

* Join table
* Junction table
* Association table

All three terms are commonly used.

---

# 4. Visualize the database

Suppose:

### Student

```text
id    name
-------------
1     Rahul
2     Aman
3     Priya
```

### Course

```text
id    name
-------------
1     Java
2     DBMS
3     Spring
```

### student_course

```text
student_id    course_id
-----------------------
1             1
1             2
1             3
2             1
2             3
3             2
```

Now translate that:

```text
Rahul → Java
Rahul → DBMS
Rahul → Spring

Aman → Java
Aman → Spring

Priya → DBMS
```

This is Many-to-Many.

---

# 5. The most important concept

Remember this:

```text
Student                Course
   |                      |
   |                      |
   └──────────┬───────────┘
              |
              ↓
       student_course
```

The join table connects the two tables.

Therefore:

```text
Student
   ↓
student_course
   ↓
Course
```

and:

```text
Course
   ↓
student_course
   ↓
Student
```

---

# 6. Create database

Open MySQL Workbench.

Run:

```sql
CREATE DATABASE hibernate_manytomany;
```

Then:

```sql
USE hibernate_manytomany;
```

Check:

```sql
SELECT DATABASE();
```

You should get:

```text
hibernate_manytomany
```

---

# 7. Create Student table

Run:

```sql
CREATE TABLE student (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL
);
```

This creates:

```text
student

id    name
-------------
1     Rahul
2     Aman
3     Priya
```

---

# 8. Create Course table

Run:

```sql
CREATE TABLE course (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL
);
```

This creates:

```text
course

id    name
-------------
1     Java
2     DBMS
3     Spring
```

---

# 9. Create Join Table

Now the most important table:

```sql
CREATE TABLE student_course (
    student_id INT NOT NULL,
    course_id INT NOT NULL,

    PRIMARY KEY (student_id, course_id),

    CONSTRAINT fk_student
        FOREIGN KEY (student_id)
        REFERENCES student(id),

    CONSTRAINT fk_course
        FOREIGN KEY (course_id)
        REFERENCES course(id)
);
```

Let's understand this deeply.

---

# 10. Why `student_id`?

```sql
student_id INT
```

This points to:

```text
student.id
```

So:

```text
student_course.student_id
             ↓
        student.id
```

---

# 11. Why `course_id`?

Similarly:

```text
student_course.course_id
             ↓
          course.id
```

Therefore:

```text
student_course

student_id → student
course_id  → course
```

---

# 12. Why two foreign keys?

Because the join table must know:

> Which student is connected to which course?

For example:

```text
student_id = 1
course_id = 2
```

means:

```text
Student 1 → Course 2
```

If:

```text
Student 1 = Rahul
Course 2 = DBMS
```

then:

```text
Rahul → DBMS
```

---

# 13. Why composite primary key?

We wrote:

```sql
PRIMARY KEY (student_id, course_id)
```

This means the **combination** must be unique.

For example:

```text
1 | 1
1 | 2
1 | 3
```

is valid.

But:

```text
1 | 1
1 | 1
```

would be a duplicate relationship.

We don't want to store:

```text
Rahul → Java
Rahul → Java
```

twice.

So:

```sql
PRIMARY KEY (student_id, course_id)
```

prevents duplicate student-course combinations.

---

# 14. Insert data manually

First insert students:

```sql
INSERT INTO student (name)
VALUES ('Rahul');

INSERT INTO student (name)
VALUES ('Aman');

INSERT INTO student (name)
VALUES ('Priya');
```

Check:

```sql
SELECT * FROM student;
```

Expected:

```text
id    name
-------------
1     Rahul
2     Aman
3     Priya
```

---

# 15. Insert courses

```sql
INSERT INTO course (name)
VALUES ('Java');

INSERT INTO course (name)
VALUES ('DBMS');

INSERT INTO course (name)
VALUES ('Spring');
```

Check:

```sql
SELECT * FROM course;
```

Expected:

```text
id    name
-------------
1     Java
2     DBMS
3     Spring
```

---

# 16. Insert relationships

Now:

```sql
INSERT INTO student_course (student_id, course_id)
VALUES (1, 1);

INSERT INTO student_course (student_id, course_id)
VALUES (1, 2);

INSERT INTO student_course (student_id, course_id)
VALUES (1, 3);

INSERT INTO student_course (student_id, course_id)
VALUES (2, 1);

INSERT INTO student_course (student_id, course_id)
VALUES (2, 3);

INSERT INTO student_course (student_id, course_id)
VALUES (3, 2);
```

Now:

```text
Rahul → Java
Rahul → DBMS
Rahul → Spring

Aman → Java
Aman → Spring

Priya → DBMS
```

---

# 17. Verify with JOIN

Run:

```sql
SELECT
    s.name AS student_name,
    c.name AS course_name
FROM student s
JOIN student_course sc
    ON s.id = sc.student_id
JOIN course c
    ON c.id = sc.course_id;
```

Result:

```text
student_name    course_name
---------------------------
Rahul           Java
Rahul           DBMS
Rahul           Spring
Aman            Java
Aman            Spring
Priya           DBMS
```

Now you completely understand the database side.

---

# 18. Now create Maven project

Create:

```text
hibernate-manytomany
```

Project structure:

```text
hibernate-manytomany
│
├── pom.xml
│
└── src
    └── main
        ├── java
        │   └── com.example
        │       ├── Main.java
        │       │
        │       └── model
        │           ├── Student.java
        │           └── Course.java
        │
        └── resources
            └── hibernate.cfg.xml
```

---

# 19. `pom.xml`

Paste:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>

    <artifactId>hibernate-manytomany</artifactId>

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
            <version>6.6.26.Final</version>
        </dependency>

        <!-- MySQL Driver -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>9.4.0</version>
        </dependency>

        <!-- Jakarta Persistence API -->
        <dependency>
            <groupId>jakarta.persistence</groupId>
            <artifactId>jakarta.persistence-api</artifactId>
            <version>3.2.0</version>
        </dependency>

        <!-- Logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>2.0.17</version>
        </dependency>

    </dependencies>

</project>
```

---

# 20. Create `Student.java`

Create:

```text
src/main/java/com/example/model/Student.java
```

Paste:

```java
package com.example.model;

import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "student")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(name = "name")
    private String name;

    @ManyToMany
    @JoinTable(
            name = "student_course",
            joinColumns = @JoinColumn(name = "student_id"),
            inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private List<Course> courses = new ArrayList<>();

    public Student() {
    }

    public Student(String name) {
        this.name = name;
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

    public List<Course> getCourses() {
        return courses;
    }

    public void setCourses(List<Course> courses) {
        this.courses = courses;
    }

    public void addCourse(Course course) {
        courses.add(course);
    }

    public void removeCourse(Course course) {
        courses.remove(course);
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

Now this is the important part:

```java
@ManyToMany
@JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
)
private List<Course> courses = new ArrayList<>();
```

We'll break it down carefully.

---

# 21. `@ManyToMany`

```java
@ManyToMany
```

means:

> One Student can have many Courses, and the same Course can belong to many Students.

For example:

```text
Rahul → Java
Rahul → DBMS

Aman → Java
Aman → Spring
```

Java:

```java
private List<Course> courses;
```

---

# 22. Why `List<Course>`?

Because one Student can have multiple Courses.

For Rahul:

```text
courses
----------------
Java
DBMS
Spring
```

So:

```java
List<Course>
```

makes sense.

---

# 23. Now the difficult part — `@JoinTable`

We write:

```java
@JoinTable(
        name = "student_course",
```

This tells Hibernate:

> Use the `student_course` table to connect Student and Course.

Remember our database:

```text
student
     \
      \
student_course
      /
     /
course
```

That's exactly what `@JoinTable` represents.

---

# 24. Understand `joinColumns`

We have:

```java
joinColumns =
    @JoinColumn(name = "student_id")
```

This means:

> The foreign key representing the current entity, Student, is `student_id`.

So:

```text
Student
   ↓
student_id
```

In the join table:

```text
student_course

student_id
course_id
```

---

# 25. Understand `inverseJoinColumns`

We have:

```java
inverseJoinColumns =
    @JoinColumn(name = "course_id")
```

This means:

> The foreign key pointing to the other entity, Course, is `course_id`.

Therefore:

```text
student_id → Student
course_id  → Course
```

This is extremely important.

---

# 26. Visualize `@JoinTable`

Our annotation:

```java
@JoinTable(
    name = "student_course",
    joinColumns = @JoinColumn(name = "student_id"),
    inverseJoinColumns = @JoinColumn(name = "course_id")
)
```

means:

```text
                  Student
                     |
                     |
                student_id
                     |
                     ↓
              student_course
                     ↑
                     |
                 course_id
                     |
                     |
                   Course
```

---

# 27. Create `Course.java`

Create:

```text
src/main/java/com/example/model/Course.java
```

Paste:

```java
package com.example.model;

import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "course")
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(name = "name")
    private String name;

    @ManyToMany(mappedBy = "courses")
    private List<Student> students = new ArrayList<>();

    public Course() {
    }

    public Course(String name) {
        this.name = name;
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

    public List<Student> getStudents() {
        return students;
    }

    public void setStudents(List<Student> students) {
        this.students = students;
    }

    @Override
    public String toString() {
        return "Course{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

---

# 28. Why `mappedBy = "courses"`?

This is extremely important.

Student has:

```java
private List<Course> courses;
```

Therefore Course says:

```java
@ManyToMany(mappedBy = "courses")
```

The:

```text
"courses"
```

is the **Java field name** inside Student.

It is not:

```text
student_course
```

and it is not:

```text
course_id
```

It is:

```java
private List<Course> courses;
```

Therefore:

```java
mappedBy = "courses"
```

---

# 29. Owning side

In our mapping:

```java
Student
```

is the owning side because it has:

```java
@JoinTable(...)
```

Course has:

```java
mappedBy = "courses"
```

Therefore:

```text
Student
  ↓
OWNING SIDE

Course
  ↓
INVERSE SIDE
```

This means Hibernate uses Student's mapping to manage the join table.

---

# 30. Why does ownership matter?

Suppose:

```java
Student student = new Student("Rahul");
Course java = new Course("Java");
```

If we do:

```java
student.addCourse(java);
```

we modify the owning side.

Hibernate can then update:

```text
student_course
```

But simply doing:

```java
course.getStudents().add(student);
```

on the inverse side does not make Course the owner.

This is why ownership matters in bidirectional relationships.

---

# 31. Hibernate configuration

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

        <!-- MySQL Driver -->
        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <!-- Database -->
        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/hibernate_manytomany
        </property>

        <!-- Username -->
        <property name="hibernate.connection.username">
            root
        </property>

        <!-- Password -->
        <property name="hibernate.connection.password">
            YOUR_MYSQL_PASSWORD
        </property>

        <!-- Dialect -->
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

        <!-- Entities -->
        <mapping class="com.example.model.Student"/>
        <mapping class="com.example.model.Course"/>

    </session-factory>

</hibernate-configuration>
```

Replace:

```text
YOUR_MYSQL_PASSWORD
```

with your MySQL password.

---

# 32. Create Main.java

Create:

```text
src/main/java/com/example/Main.java
```

Paste:

```java
package com.example;

import com.example.model.Course;
import com.example.model.Student;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class Main {

    public static void main(String[] args) {

        // 1. Create Configuration
        Configuration configuration = new Configuration();

        // 2. Read hibernate.cfg.xml
        configuration.configure();

        // 3. Build SessionFactory
        SessionFactory sessionFactory =
                configuration.buildSessionFactory();

        // 4. Open Session
        Session session =
                sessionFactory.openSession();

        // 5. Start Transaction
        session.beginTransaction();

        // 6. Create Students
        Student rahul =
                new Student("Rahul");

        Student aman =
                new Student("Aman");

        Student priya =
                new Student("Priya");

        // 7. Create Courses
        Course java =
                new Course("Java");

        Course dbms =
                new Course("DBMS");

        Course spring =
                new Course("Spring");

        // 8. Create Many-to-Many relationships

        rahul.addCourse(java);
        rahul.addCourse(dbms);
        rahul.addCourse(spring);

        aman.addCourse(java);
        aman.addCourse(spring);

        priya.addCourse(dbms);

        // 9. Save Courses
        session.persist(java);
        session.persist(dbms);
        session.persist(spring);

        // 10. Save Students
        session.persist(rahul);
        session.persist(aman);
        session.persist(priya);

        // 11. Commit
        session.getTransaction().commit();

        // 12. Close Session
        session.close();

        // 13. Close SessionFactory
        sessionFactory.close();

        System.out.println(
                "Many-to-Many data saved successfully!"
        );
    }
}
```

---

# 33. Understand the Main method

Let's go line by line.

First:

```java
Configuration configuration =
        new Configuration();
```

Creates Hibernate configuration.

Then:

```java
configuration.configure();
```

reads:

```text
hibernate.cfg.xml
```

Then:

```java
SessionFactory sessionFactory =
        configuration.buildSessionFactory();
```

creates the SessionFactory.

Then:

```java
Session session =
        sessionFactory.openSession();
```

opens a database session.

Then:

```java
session.beginTransaction();
```

starts the transaction.

---

# 34. Create Students

```java
Student rahul =
        new Student("Rahul");
```

This creates a Java object:

```text
Student
---------
name = Rahul
```

Similarly:

```java
Student aman =
        new Student("Aman");

Student priya =
        new Student("Priya");
```

---

# 35. Create Courses

```java
Course java =
        new Course("Java");
```

creates:

```text
Course
---------
name = Java
```

Similarly:

```java
Course dbms =
        new Course("DBMS");

Course spring =
        new Course("Spring");
```

---

# 36. Now create relationships

This is the most important part:

```java
rahul.addCourse(java);
rahul.addCourse(dbms);
rahul.addCourse(spring);
```

This means:

```text
Rahul
  |
  ├── Java
  ├── DBMS
  └── Spring
```

Then:

```java
aman.addCourse(java);
aman.addCourse(spring);
```

means:

```text
Aman
  |
  ├── Java
  └── Spring
```

And:

```java
priya.addCourse(dbms);
```

means:

```text
Priya
  |
  └── DBMS
```

---

# 37. Visualize Java objects

After the relationship code:

```text
                    Java
                   ↑    ↑
                  /      \
              Rahul      Aman
               / \       /
              /   \     /
           DBMS    Spring
             ↑
             |
           Priya
```

A cleaner representation:

```text
Rahul
 ├── Java
 ├── DBMS
 └── Spring

Aman
 ├── Java
 └── Spring

Priya
 └── DBMS
```

That's Many-to-Many.

---

# 38. Save courses

We do:

```java
session.persist(java);
session.persist(dbms);
session.persist(spring);
```

This saves the Course entities.

Conceptually:

```sql
INSERT INTO course (name)
VALUES ('Java');

INSERT INTO course (name)
VALUES ('DBMS');

INSERT INTO course (name)
VALUES ('Spring');
```

---

# 39. Save students

Then:

```java
session.persist(rahul);
session.persist(aman);
session.persist(priya);
```

Hibernate saves:

```text
student
```

records.

---

# 40. What about `student_course`?

This is the beautiful part.

You never manually wrote:

```sql
INSERT INTO student_course
```

Hibernate understands:

```java
rahul.addCourse(java);
```

and because Student is the owning side, Hibernate generates relationship rows in the join table.

Conceptually:

```sql
INSERT INTO student_course
(student_id, course_id)
VALUES (1, 1);
```

Then:

```sql
INSERT INTO student_course
(student_id, course_id)
VALUES (1, 2);
```

Then:

```sql
INSERT INTO student_course
(student_id, course_id)
VALUES (1, 3);
```

and so on.

---

# 41. Final database

After running the application:

### student

```text
id    name
-------------
1     Rahul
2     Aman
3     Priya
```

### course

```text
id    name
-------------
1     Java
2     DBMS
3     Spring
```

### student_course

```text
student_id    course_id
-----------------------
1             1
1             2
1             3
2             1
2             3
3             2
```

This is exactly what we wanted.

---

# 42. The relationship in one picture

```text
                 STUDENT
                    |
                    |
             @ManyToMany
                    |
                    ↓
             student_course
              /          \
             /            \
            ↓              ↓
     student_id          course_id
            |              |
            ↓              ↓
        Student          Course
```

Actually, conceptually:

```text
Student
   |
   | many
   ↓
student_course
   ↑
   | many
   |
Course
```

Both sides can have many records.

---

# 43. Why do we need a join table?

This is the most important database question.

Suppose we had:

```text
student

id | name | course_id
```

Rahul needs:

```text
Java
DBMS
Spring
```

We would need multiple course IDs in one field:

```text
1,2,3
```

That's not good relational design.

Instead:

```text
student_course

student_id | course_id
----------------------
1          | 1
1          | 2
1          | 3
```

One relationship = one row.

That's the proper relational model.

---

# 44. `@ManyToMany` without `mappedBy`

If we only wanted one-direction navigation:

```text
Student → Courses
```

we could write:

```java
@ManyToMany
@JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
)
private List<Course> courses = new ArrayList<>();
```

And Course wouldn't need:

```java
List<Student>
```

This is a **unidirectional Many-to-Many**.

---

# 45. Bidirectional Many-to-Many

Our example is bidirectional.

Student:

```java
@ManyToMany
@JoinTable(...)
private List<Course> courses;
```

Course:

```java
@ManyToMany(mappedBy = "courses")
private List<Student> students;
```

Therefore:

```text
Student
   ↓
courses

Course
   ↓
students
```

You can navigate in both directions.

---

# 46. What does navigation mean?

From Student:

```java
rahul.getCourses();
```

You can get:

```text
Java
DBMS
Spring
```

From Course:

```java
java.getStudents();
```

you can get:

```text
Rahul
Aman
```

That's why bidirectional relationships are useful.

---

# 47. Important problem with our helper method

Our current:

```java
public void addCourse(Course course) {
    courses.add(course);
}
```

updates only Student's list.

For a fully synchronized **bidirectional in-memory relationship**, it's better to update both sides.

Change Student to:

```java
public void addCourse(Course course) {
    courses.add(course);
    course.getStudents().add(this);
}
```

And:

```java
public void removeCourse(Course course) {
    courses.remove(course);
    course.getStudents().remove(this);
}
```

Now both Java objects know about the relationship.

---

# 48. Better `Student.java`

The relevant part becomes:

```java
@ManyToMany
@JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
)
private List<Course> courses = new ArrayList<>();

public void addCourse(Course course) {
    courses.add(course);
    course.getStudents().add(this);
}

public void removeCourse(Course course) {
    courses.remove(course);
    course.getStudents().remove(this);
}
```

Now:

```java
rahul.addCourse(java);
```

does:

```text
Rahul.courses
       ↓
      Java

Java.students
       ↓
     Rahul
```

Both sides are synchronized.

---

# 49. Fetch a student and courses

After saving, we can retrieve:

```java
Student student =
        session.find(Student.class, 1);
```

Then:

```java
System.out.println(student.getName());
```

Output:

```text
Rahul
```

And:

```java
for (Course course : student.getCourses()) {
    System.out.println(course.getName());
}
```

Output:

```text
Java
DBMS
Spring
```

---

# 50. Fetch course and students

Similarly:

```java
Course course =
        session.find(Course.class, 1);
```

Then:

```java
System.out.println(course.getName());
```

Output:

```text
Java
```

And:

```java
for (Student student : course.getStudents()) {
    System.out.println(student.getName());
}
```

Output:

```text
Rahul
Aman
```

So we can navigate:

```text
Student → Courses
```

and:

```text
Course → Students
```

---

# 51. Understand ownership again

This is one of the most important interview questions.

Who owns this relationship?

```java
Student
```

because Student has:

```java
@JoinTable(...)
```

Course has:

```java
mappedBy = "courses"
```

Therefore:

```text
Student = owning side
Course  = inverse side
```

---

# 52. What does `mappedBy` mean here?

Course:

```java
@ManyToMany(mappedBy = "courses")
```

means:

> Don't create/manage another join-table mapping from Course. The Student entity's `courses` field already owns this relationship.

And:

```text
"courses"
```

refers to:

```java
private List<Course> courses;
```

inside Student.

---

# 53. `mappedBy` is Java-field based

Remember this from the previous relationships too.

If Student has:

```java
private List<Course> courses;
```

then:

```java
mappedBy = "courses"
```

Correct.

If Student had:

```java
private List<Course> enrolledCourses;
```

then we would write:

```java
mappedBy = "enrolledCourses"
```

Not:

```java
mappedBy = "student_course"
```

and not:

```java
mappedBy = "course_id"
```

---

# 54. Compare all Hibernate relationships

Now you have learned all four major relationship types.

## One-to-One

```text
Person ───── Passport
   1          1
```

Annotation:

```java
@OneToOne
```

---

## One-to-Many

```text
Department
     |
     ├── Employee
     ├── Employee
     └── Employee

1              Many
```

Annotation:

```java
@OneToMany
```

---

## Many-to-One

```text
Employee ──┐
Employee ──┼──→ Department
Employee ──┘

Many             1
```

Annotation:

```java
@ManyToOne
```

---

## Many-to-Many

```text
Student ──┐
Student ──┼──→ Courses
Student ──┘
              ↑
           Many

Many ↔ Many
```

Annotation:

```java
@ManyToMany
```

And normally:

```text
Many-to-Many
     ↓
Join Table
```

---

# 55. The four relationships in one table

| Relationship | Example                | Database idea   |
| ------------ | ---------------------- | --------------- |
| One-to-One   | Person ↔ Passport      | Foreign key     |
| One-to-Many  | Department → Employees | FK on many side |
| Many-to-One  | Employees → Department | FK on many side |
| Many-to-Many | Students ↔ Courses     | Join table      |

---

# 56. The complete Many-to-Many architecture

Remember this diagram:

```text
                  JAVA

       ┌────────────────────┐
       │      Student       │
       │                    │
       │ id                 │
       │ name               │
       │                    │
       │ List<Course>       │
       └─────────┬──────────┘
                 │
                 │ @ManyToMany
                 │
                 ↓
       ┌────────────────────┐
       │       Course       │
       │                    │
       │ id                 │
       │ name               │
       │                    │
       │ List<Student>      │
       └────────────────────┘


                  ↓
               Hibernate
                  ↓
              Join Table
                  ↓

       ┌────────────────────┐
       │   student_course   │
       │                    │
       │ student_id         │
       │ course_id          │
       └────────────────────┘
```

---

# 57. Complete database architecture

```text
┌──────────────┐
│   student    │
├──────────────┤
│ id           │
│ name         │
└──────┬───────┘
       │
       │ 1
       │
       │
       │ Many
┌──────▼─────────────┐
│   student_course   │
├────────────────────┤
│ student_id         │
│ course_id          │
└──────┬─────────────┘
       │
       │ Many
       │
       │
       │ 1
┌──────▼───────┐
│    course    │
├──────────────┤
│ id           │
│ name         │
└──────────────┘
```

The join table turns the Many-to-Many relationship into two One-to-Many-style foreign-key relationships at the database level.

---

# 58. One very important real-world warning

For a simple example, this is fine:

```java
@ManyToMany
private List<Course> courses;
```

But in real applications, Many-to-Many often becomes more complicated.

Suppose enrollment needs:

```text
student_id
course_id
enrollment_date
grade
status
```

Now the relationship itself has data.

Instead of directly using:

```text
Student ↔ Course
```

you may create an entity such as:

```text
Enrollment
```

Then:

```text
Student 1 ─── * Enrollment * ─── 1 Course
```

This is often a better real-world design.

For learning basic Hibernate `@ManyToMany`, however, our current model is perfect.

---

# 59. Final cheat sheet

The most important code is:

### Student

```java
@ManyToMany
@JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
)
private List<Course> courses = new ArrayList<>();
```

### Course

```java
@ManyToMany(mappedBy = "courses")
private List<Student> students = new ArrayList<>();
```

Remember:

```text
@ManyToMany
      ↓
Two entities can have multiple relationships
      ↓
Usually requires a JOIN TABLE
      ↓
student_course
      ↓
student_id + course_id
```

And the entire concept can be remembered as:

```text
          MANY                    MANY

       Students  ─────────────  Courses
           │                       │
           │                       │
           └──── student_course ───┘
                    │
             student_id
             course_id
```

**That's Hibernate `@ManyToMany`.**


**That is `@ManyToOne`.**
