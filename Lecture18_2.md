# Hibernate ORM — One-to-Many

## 1. What are we going to build?

We will create a simple **Department → Employees** relationship.

One department can have many employees:

```text
Department
   |
   | 1
   |
   |-------------------|
   |        |          |
   ↓        ↓          ↓
Employee  Employee   Employee
   M        M          M
```

For example:

```text
Department
----------------
id = 1
name = IT

Employees
----------------
101  Rahul   1
102  Aman    1
103  Priya   1
```

Here:

* **One Department**
* can have **Many Employees**

Therefore:

```text
Department 1 -------- * Employee
```

This is called a **One-to-Many relationship**.

---

# 2. First understand the database relationship

Before touching Java or Hibernate, understand the database.

We need two tables:

```text
department
---------------------
id
name


employee
---------------------
id
name
salary
department_id
```

The important column is:

```text
department_id
```

inside the `employee` table.

Why?

Because every employee needs to know **which department they belong to**.

For example:

```text
department

id     name
----------------
1      IT
2      HR
3      Finance
```

And:

```text
employee

id     name       salary     department_id
------------------------------------------------
101    Rahul      50000      1
102    Aman       60000      1
103    Priya      55000      1
104    Neha       45000      2
```

Look carefully:

```text
Rahul  → department_id = 1
Aman   → department_id = 1
Priya  → department_id = 1
```

Therefore:

```text
Department 1 (IT)
       |
       |------ Rahul
       |
       |------ Aman
       |
       |------ Priya
```

That's the entire idea behind One-to-Many.

---

# 3. Why does the foreign key go into Employee?

This is extremely important.

Suppose we tried to put employees inside the department table:

```text
department

id | name | employee1 | employee2 | employee3
```

This is a bad relational database design because the number of employees isn't fixed.

A department might have:

```text
1 employee
```

or

```text
10 employees
```

or

```text
10,000 employees
```

Instead, we create a separate employee table.

Then each employee stores:

```text
department_id
```

So the database naturally represents:

```text
Department
    ↑
    |
    | foreign key
    |
Employee
```

The **Many side owns the foreign-key column**.

Therefore:

```text
Department = One side

Employee = Many side
```

---

# 4. Create the database

We'll use MySQL.

Open MySQL Workbench or MySQL command line.

Create the database:

```sql
CREATE DATABASE hibernate_onetomany;
```

Select it:

```sql
USE hibernate_onetomany;
```

Check:

```sql
SHOW DATABASES;
```

You should see:

```text
hibernate_onetomany
```

---

# 5. Create the Department table

Run:

```sql
CREATE TABLE department (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL
);
```

Let's understand this.

### `id`

```sql
id INT
```

The department ID is an integer.

Example:

```text
1
2
3
```

### `PRIMARY KEY`

```sql
PRIMARY KEY
```

Every department must have a unique ID.

### `AUTO_INCREMENT`

```sql
AUTO_INCREMENT
```

MySQL automatically generates IDs.

So if you insert:

```sql
INSERT INTO department (name)
VALUES ('IT');
```

MySQL automatically creates:

```text
id = 1
```

---

# 6. Create Employee table

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

This is the most important table for understanding One-to-Many.

The important part is:

```sql
department_id INT
```

and:

```sql
FOREIGN KEY (department_id)
REFERENCES department(id)
```

This means:

> The `department_id` inside employee must refer to an existing `id` inside department.

---

# 7. Insert some data manually

First insert departments:

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

You should get something similar to:

```text
id    name
----------------
1     IT
2     HR
3     Finance
```

Now employees:

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

Expected:

```text
id    name     salary    department_id
---------------------------------------
1     Rahul    50000     1
2     Aman     60000     1
3     Priya    55000     1
4     Neha     45000     2
```

Now we can see:

```text
IT
 |
 |--- Rahul
 |--- Aman
 |--- Priya

HR
 |
 |--- Neha

Finance
 |
 |--- nobody
```

This is One-to-Many.

---

# 8. Let's verify using JOIN

Run:

```sql
SELECT
    d.id AS department_id,
    d.name AS department_name,
    e.id AS employee_id,
    e.name AS employee_name,
    e.salary
FROM department d
JOIN employee e
ON d.id = e.department_id;
```

Result:

```text
department_id | department_name | employee_id | employee_name | salary
----------------------------------------------------------------------
1             | IT              | 1           | Rahul         | 50000
1             | IT              | 2           | Aman          | 60000
1             | IT              | 3           | Priya         | 55000
2             | HR              | 4           | Neha          | 45000
```

Notice:

```text
IT → Rahul
IT → Aman
IT → Priya
```

The same department appears multiple times because it has multiple employees.

That is the database meaning of:

```text
ONE Department → MANY Employees
```

---

# 9. Now create the Maven project

Now we move from database to Java.

Our project structure will eventually look like this:

```text
hibernate-onetomany
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
        │           ├── Department.java
        │           └── Employee.java
        │
        └── resources
            └── hibernate.cfg.xml
```

This structure is worth remembering.

```text
model
```

contains our entity/model classes.

```text
hibernate.cfg.xml
```

contains Hibernate configuration.

```text
Main.java
```

contains the program we execute.

---

# 10. Create Maven project

If you're using IntelliJ IDEA:

```text
File
 ↓
New
 ↓
Project
 ↓
Maven
```

Choose Java.

Give the project:

```text
hibernate-onetomany
```

You can use:

```text
GroupId:
com.example
```

and:

```text
ArtifactId:
hibernate-onetomany
```

---

# 11. pom.xml

Now open:

```text
pom.xml
```

Replace everything with:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>

    <artifactId>hibernate-onetomany</artifactId>

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

> The dependency versions above are examples for a Hibernate 6/Jakarta setup. If Maven reports a version-resolution problem, use the current compatible versions available in your Maven repository.

---

# 12. What is Maven?

Before continuing, understand this.

Without Maven, you would have to manually download:

```text
Hibernate JAR
MySQL Driver JAR
Jakarta Persistence JAR
Logging JAR
```

and then add them to your project.

Maven does this for us.

We tell Maven:

```xml
<dependency>
```

and Maven downloads the required library.

Think:

```text
pom.xml
   ↓
Maven
   ↓
Downloads libraries
   ↓
Your Java project
```

---

# 13. What is `pom.xml`?

POM means:

**Project Object Model**

It tells Maven information about your project.

For example:

```xml
<groupId>com.example</groupId>
```

identifies the organization/project group.

```xml
<artifactId>hibernate-onetomany</artifactId>
```

is the project name.

```xml
<version>1.0-SNAPSHOT</version>
```

is the project version.

And:

```xml
<dependencies>
```

contains external libraries.

---

# 14. Hibernate dependency

This:

```xml
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>6.6.26.Final</version>
</dependency>
```

means:

> Maven, please give my project Hibernate ORM.

Hibernate is the technology that converts Java objects into database records and vice versa.

---

# 15. MySQL driver

This:

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.4.0</version>
</dependency>
```

allows Java/Hibernate to communicate with MySQL.

Think:

```text
Java
  ↓
Hibernate
  ↓
JDBC Driver
  ↓
MySQL
```

---

# 16. Create model package

Inside:

```text
src/main/java/com/example
```

create:

```text
model
```

Inside model create:

```text
Department.java
Employee.java
```

---

# 17. Department.java

Paste:

```java
package com.example.model;

import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "department")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(name = "name")
    private String name;

    @OneToMany(mappedBy = "department")
    private List<Employee> employees = new ArrayList<>();

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

    public List<Employee> getEmployees() {
        return employees;
    }

    public void setEmployees(List<Employee> employees) {
        this.employees = employees;
    }

    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }

    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);
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

Don't worry if this looks complicated.

We're going to break it down.

---

# 18. What is `@Entity`?

This:

```java
@Entity
```

tells Hibernate:

> This Java class represents something that should be mapped to a database table.

So:

```java
@Entity
public class Department
```

means:

```text
Java class
     ↓
Hibernate
     ↓
Database table
```

Our:

```java
Department
```

class maps to:

```text
department
```

table.

---

# 19. What is `@Table`?

We wrote:

```java
@Table(name = "department")
```

This tells Hibernate:

> Use the database table named `department`.

So:

```text
Department.java
      ↓
department table
```

---

# 20. What is `@Id`?

We have:

```java
@Id
private int id;
```

`@Id` means this field is the **primary key**.

Database:

```sql
id INT PRIMARY KEY
```

Java:

```java
@Id
private int id;
```

They represent the same concept.

---

# 21. What is `@GeneratedValue`?

We have:

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

Our database has:

```sql
AUTO_INCREMENT
```

Therefore Hibernate should allow the database to generate the ID.

So:

```text
Java:
id = 0

Hibernate
   ↓

Database generates:
id = 1
```

Then:

```java
department.getId()
```

will return:

```text
1
```

after saving.

---

# 22. Now the most important annotation

This:

```java
@OneToMany(mappedBy = "department")
private List<Employee> employees = new ArrayList<>();
```

is the heart of One-to-Many.

Let's understand it word by word.

```java
@OneToMany
```

means:

> One Department is associated with many Employees.

Then:

```java
List<Employee>
```

means:

> A Department contains multiple Employee objects.

For example:

```text
Department IT

employees:
    Rahul
    Aman
    Priya
```

In Java:

```java
List<Employee> employees;
```

---

# 23. What does `mappedBy = "department"` mean?

This is one of the most confusing parts for beginners.

We will create Employee.java now.

---

# 24. Employee.java

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

Now look at this:

```java
@ManyToOne
@JoinColumn(name = "department_id")
private Department department;
```

This is extremely important.

---

# 25. Why Employee has `@ManyToOne`

Remember:

```text
IT
 |
 |--- Rahul
 |--- Aman
 |--- Priya
```

From Department's perspective:

```text
One Department → Many Employees
```

Therefore:

```java
@OneToMany
```

But from Employee's perspective:

```text
Many Employees → One Department
```

Therefore:

```java
@ManyToOne
```

This is the same relationship viewed from two different directions.

```text
Department                 Employee

@OneToMany                  @ManyToOne

One                         Many
 |                            |
 |                            |
 ↓                            ↓
Many                         One
```

---

# 26. Understand `@JoinColumn`

We wrote:

```java
@JoinColumn(name = "department_id")
```

This tells Hibernate:

> The relationship is stored in the `department_id` column of the employee table.

Database:

```text
employee

id | name | salary | department_id
```

Java:

```java
private Department department;
```

Hibernate connects these using:

```java
@JoinColumn(name = "department_id")
```

So:

```text
Employee.department
        ↓
employee.department_id
        ↓
department.id
```

---

# 27. Now understand `mappedBy`

In Department:

```java
@OneToMany(mappedBy = "department")
private List<Employee> employees;
```

The word:

```java
"department"
```

refers to this field in Employee:

```java
private Department department;
```

It is **not** the database column name.

This is a very common beginner mistake.

We have:

```java
mappedBy = "department"
```

because Employee has:

```java
private Department department;
```

If Employee had:

```java
private Department myDepartment;
```

then we would write:

```java
@OneToMany(mappedBy = "myDepartment")
```

---

# 28. Who owns the relationship?

Another extremely important concept.

In our project:

```java
Employee
```

is the **owning side**.

Why?

Because Employee contains:

```java
@JoinColumn(name = "department_id")
```

Therefore Employee controls the foreign key.

Department has:

```java
mappedBy = "department"
```

which tells Hibernate:

> Don't create another relationship column. The Employee side already manages it.

Think:

```text
Department
@OneToMany
     |
     | mappedBy
     ↓
Employee
@ManyToOne
@JoinColumn
     |
     ↓
department_id
```

---

# 29. Why do we need both sides?

We could technically model only:

```java
Employee → Department
```

But then Java wouldn't easily allow:

```java
department.getEmployees()
```

By having both sides:

```java
Department → Employees
Employee → Department
```

we get a **bidirectional relationship**.

For example:

```java
employee.getDepartment();
```

and:

```java
department.getEmployees();
```

Both are possible.

---

# 30. Create Hibernate configuration

Now create:

```text
src/main/resources
```

Inside it create:

```text
hibernate.cfg.xml
```

Paste:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "https://hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <!-- Database connection -->
        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/hibernate_onetomany
        </property>

        <property name="hibernate.connection.username">
            root
        </property>

        <property name="hibernate.connection.password">
            YOUR_MYSQL_PASSWORD
        </property>

        <!-- Hibernate settings -->
        <property name="hibernate.dialect">
            org.hibernate.dialect.MySQLDialect
        </property>

        <property name="hibernate.show_sql">
            true
        </property>

        <property name="hibernate.format_sql">
            true
        </property>

        <!-- Entity classes -->
        <mapping class="com.example.model.Department"/>
        <mapping class="com.example.model.Employee"/>

    </session-factory>

</hibernate-configuration>
```

Change:

```text
YOUR_MYSQL_PASSWORD
```

to your actual MySQL password.

---

# 31. Understand Hibernate configuration

This:

```xml
<session-factory>
```

contains Hibernate's configuration.

Think:

```text
hibernate.cfg.xml
        ↓
SessionFactory
        ↓
Hibernate
        ↓
Database
```

---

# 32. Database driver

```xml
<property name="hibernate.connection.driver_class">
    com.mysql.cj.jdbc.Driver
</property>
```

This tells Hibernate:

> We are using the MySQL JDBC driver.

---

# 33. Database URL

```xml
<property name="hibernate.connection.url">
    jdbc:mysql://localhost:3306/hibernate_onetomany
</property>
```

Break it down:

```text
jdbc
 ↓
mysql
 ↓
localhost
 ↓
3306
 ↓
hibernate_onetomany
```

`localhost` means your own computer.

`3306` is MySQL's commonly used port.

`hibernate_onetomany` is our database.

---

# 34. Username and password

```xml
<property name="hibernate.connection.username">
    root
</property>
```

MySQL username.

And:

```xml
<property name="hibernate.connection.password">
    YOUR_MYSQL_PASSWORD
</property>
```

MySQL password.

---

# 35. Show SQL

We have:

```xml
<property name="hibernate.show_sql">
    true
</property>
```

This is extremely useful while learning.

Hibernate might execute:

```sql
select
    d1_0.id,
    d1_0.name
from
    department d1_0
```

With `show_sql=true`, you can see Hibernate's SQL in the console.

That lets you understand:

```text
Java operation
     ↓
Hibernate
     ↓
SQL
     ↓
MySQL
```

---

# 36. Entity mapping

These lines:

```xml
<mapping class="com.example.model.Department"/>
<mapping class="com.example.model.Employee"/>
```

tell Hibernate:

> These two Java classes are Hibernate entities.

---

# 37. Now create Main.java

Create:

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

        // 1. Create Configuration object
        Configuration configuration = new Configuration();

        // 2. Read hibernate.cfg.xml
        configuration.configure();

        // 3. Build SessionFactory
        SessionFactory sessionFactory =
                configuration.buildSessionFactory();

        // 4. Open a session
        Session session = sessionFactory.openSession();

        // 5. Start transaction
        session.beginTransaction();

        // 6. Create Department
        Department department = new Department("IT");

        // 7. Create Employees
        Employee employee1 =
                new Employee("Rahul", 50000);

        Employee employee2 =
                new Employee("Aman", 60000);

        Employee employee3 =
                new Employee("Priya", 55000);

        // 8. Establish relationship
        department.addEmployee(employee1);
        department.addEmployee(employee2);
        department.addEmployee(employee3);

        // 9. Save Department
        session.persist(department);

        // 10. Save Employees
        session.persist(employee1);
        session.persist(employee2);
        session.persist(employee3);

        // 11. Commit transaction
        session.getTransaction().commit();

        // 12. Close session
        session.close();

        // 13. Close SessionFactory
        sessionFactory.close();

        System.out.println("Data saved successfully!");
    }
}
```

---

# 38. Understand Main.java step by step

Let's slow down.

## Step 1

```java
Configuration configuration = new Configuration();
```

We create a Hibernate Configuration object.

Think:

```text
Configuration
     |
     ↓
Hibernate settings
```

---

# 39. Step 2

```java
configuration.configure();
```

This tells Hibernate:

> Read `hibernate.cfg.xml`.

Hibernate searches for:

```text
hibernate.cfg.xml
```

inside:

```text
src/main/resources
```

---

# 40. Step 3

```java
SessionFactory sessionFactory =
        configuration.buildSessionFactory();
```

This creates a:

```text
SessionFactory
```

A SessionFactory is a heavyweight Hibernate object used to create sessions.

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

Normally you create one SessionFactory for your application.

---

# 41. Step 4

```java
Session session = sessionFactory.openSession();
```

A Session represents a working interaction with the database.

Think:

```text
Session
   =
conversation with database
```

---

# 42. Step 5

```java
session.beginTransaction();
```

We begin a database transaction.

Think:

```text
BEGIN
   ↓
Database operations
   ↓
COMMIT
```

---

# 43. Create Department

```java
Department department = new Department("IT");
```

We create:

```text
Department object
```

in Java memory.

At this moment it is **not yet in the database**.

We have:

```text
Java memory

department
   |
   ↓
Department{id=0, name="IT"}
```

---

# 44. Create Employees

```java
Employee employee1 =
        new Employee("Rahul", 50000);
```

creates:

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

# 45. The most important part: establish relationship

We wrote:

```java
department.addEmployee(employee1);
```

Remember our method:

```java
public void addEmployee(Employee employee) {
    employees.add(employee);
    employee.setDepartment(this);
}
```

This does **two things**.

First:

```java
employees.add(employee);
```

means:

```text
Department
    |
    ↓
employees List
    |
    ↓
Rahul
```

Second:

```java
employee.setDepartment(this);
```

means:

```text
Rahul
  |
  ↓
department
  |
  ↓
IT
```

So both sides become connected.

---

# 46. Visualize the Java objects

After:

```java
department.addEmployee(employee1);
department.addEmployee(employee2);
department.addEmployee(employee3);
```

we have:

```text
                 Department
                /          \
               /            \
             IT              employees
                              |
                ┌─────────────┼─────────────┐
                ↓             ↓             ↓
             Rahul           Aman          Priya
                |             |             |
                └─────────────┴─────────────┘
                              |
                              ↓
                             IT
```

Conceptually:

```text
Department
   |
   | employees
   |
   ├── Employee Rahul
   |
   ├── Employee Aman
   |
   └── Employee Priya
```

And each Employee knows:

```text
Rahul.department → IT
Aman.department  → IT
Priya.department → IT
```

---

# 47. `session.persist()`

We then write:

```java
session.persist(department);
```

This tells Hibernate:

> Make this object persistent.

Then:

```java
session.persist(employee1);
session.persist(employee2);
session.persist(employee3);
```

does the same for employees.

---

# 48. What happens in the database?

Hibernate eventually performs SQL similar to:

```sql
INSERT INTO department
(name)
VALUES
('IT');
```

Suppose MySQL generates:

```text
id = 4
```

Then employees can be inserted:

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

So database becomes:

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

# 49. Commit

Finally:

```java
session.getTransaction().commit();
```

means:

> Permanently commit the transaction.

Conceptually:

```text
beginTransaction()
       ↓
INSERT/UPDATE/DELETE
       ↓
commit()
       ↓
changes become committed
```

---

# 50. Close session

```java
session.close();
```

We no longer need that database session.

Then:

```java
sessionFactory.close();
```

closes the SessionFactory.

---

# 51. Complete project structure

At this point your project should look like:

```text
hibernate-onetomany
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
        │           └── model
        │               ├── Department.java
        │               └── Employee.java
        │
        └── resources
            └── hibernate.cfg.xml
```

---

# 52. Run the application

Before running:

### MySQL must be running.

Make sure:

```text
MySQL Server
     ↓
Running
```

Your database must exist:

```text
hibernate_onetomany
```

And your password in:

```text
hibernate.cfg.xml
```

must be correct.

Then run:

```text
Main.java
```

---

# 53. Expected output

You should see Hibernate startup information and SQL.

Something similar to:

```text
Hibernate:
    insert
    into
        department
        (name)
    values
        (?)
```

Then:

```text
Hibernate:
    insert
    into
        employee
        (department_id, name, salary)
    values
        (?, ?, ?)
```

And finally:

```text
Data saved successfully!
```

---

# 54. Check your database

Run:

```sql
SELECT * FROM department;
```

Then:

```sql
SELECT * FROM employee;
```

You should see the new department and employees.

You can also run:

```sql
SELECT
    d.name AS department,
    e.name AS employee,
    e.salary
FROM department d
JOIN employee e
ON d.id = e.department_id;
```

You'll get something like:

```text
department    employee    salary
---------------------------------
IT            Rahul       50000
IT            Aman        60000
IT            Priya       55000
```

---

# 55. The entire concept in one picture

This is the picture I want you to remember:

```text
             JAVA
              |
              |
       ┌──────▼──────┐
       │ Department  │
       │             │
       │ id          │
       │ name        │
       │             │
       │ employees[] │
       └──────┬──────┘
              |
              | @OneToMany
              |
              ↓
       ┌──────────────┐
       │   Employee   │
       │              │
       │ id           │
       │ name         │
       │ salary       │
       │ department   │
       └──────┬───────┘
              |
              | @ManyToOne
              |
              ↓
        department_id
              |
              |
           DATABASE
              |
       ┌──────▼─────────┐
       │   department   │
       ├────────────────┤
       │ id             │
       │ name           │
       └────────────────┘

       ┌────────────────┐
       │    employee    │
       ├────────────────┤
       │ id             │
       │ name           │
       │ salary         │
       │ department_id  │ ← FOREIGN KEY
       └────────────────┘
```

---

# 56. The relationship annotations together

Memorize this pair:

### Department

```java
@OneToMany(mappedBy = "department")
private List<Employee> employees;
```

### Employee

```java
@ManyToOne
@JoinColumn(name = "department_id")
private Department department;
```

Together:

```text
Department
    |
    | @OneToMany
    |
    ↓
Employee
    |
    | @ManyToOne
    |
    ↓
Department
```

Database:

```text
department.id
      ↑
      |
      |
employee.department_id
```

---

# 57. One thing beginners frequently misunderstand

This:

```java
@OneToMany(mappedBy = "department")
```

does **NOT** mean:

```text
database column = department
```

`mappedBy` refers to the **Java field name**.

Employee has:

```java
private Department department;
```

Therefore:

```java
mappedBy = "department"
```

But the actual database column is:

```java
@JoinColumn(name = "department_id")
```

Therefore:

```text
Java field:
department

Database column:
department_id
```

They are two different things.

---

# 58. Database vs Java — compare them

| Database                        | Java               |
| ------------------------------- | ------------------ |
| `department` table              | `Department` class |
| `employee` table                | `Employee` class   |
| `id`                            | `@Id id`           |
| `department_id`                 | `@JoinColumn`      |
| Foreign Key                     | `@ManyToOne`       |
| One department → many employees | `@OneToMany`       |
| Row                             | Object             |
| Table                           | Entity class       |

This is the bridge between SQL and Hibernate.

---

# 59. What Hibernate is actually doing

Without Hibernate, you might write SQL manually:

```sql
INSERT INTO employee
(name, salary, department_id)
VALUES ('Rahul', 50000, 1);
```

With Hibernate, you write:

```java
Employee employee =
        new Employee("Rahul", 50000);

employee.setDepartment(department);

session.persist(employee);
```

Hibernate translates the object operation into SQL.

So the fundamental idea is:

```text
JAVA OBJECT
     ↓
Hibernate ORM
     ↓
SQL
     ↓
DATABASE
```

And going the other direction:

```text
DATABASE
     ↓
SQL result
     ↓
Hibernate ORM
     ↓
JAVA OBJECT
```

That's why it's called:

**Object-Relational Mapping**

```text
Object
   ↕
Relational database
```

---

# 60. One-to-Many in one sentence

If you remember only one thing:

> **One-to-Many means one object is associated with a collection of many objects, while the database stores the relationship using a foreign key on the many side.**

In our example:

```text
One Department
       ↓
Many Employees
```

and:

```text
employee.department_id
        ↓
department.id
```

That's the core of Hibernate One-to-Many.
