# Lecture 05: `CallableStatement`

---

# Recap

So far we have learned three ways to execute SQL.

### Statement

```java
Statement statement = connection.createStatement();
```

Best for

* Static SQL

Example

```sql
SELECT * FROM employee;
```

---

### PreparedStatement

```java
PreparedStatement ps =
connection.prepareStatement(query);
```

Best for

* Dynamic values
* User input
* Secure queries

Example

```sql
INSERT INTO employee(name,age)
VALUES(?,?);
```

---

Now comes the third interface.

```java
CallableStatement
```

---

# What is CallableStatement?

## Definition

**CallableStatement** is a JDBC interface used to call **Stored Procedures** and **Stored Functions** present inside the database.

Instead of writing SQL directly inside Java,

Java simply asks the database

> "Run this stored procedure."

---

# What is a Stored Procedure?

A Stored Procedure is a collection of SQL statements stored permanently inside the database under a name.

Think of it like a Java method.

Java

```java
calculateSalary();
```

Database

```sql
CALL calculateSalary();
```

---

# Real Life Analogy

Imagine an ATM.

When you press

```text
Withdraw Money
```

The ATM doesn't perform all banking calculations itself.

Instead,

it sends a request to the bank server.

The server already contains a predefined process.

```
Check Balance

↓

Check PIN

↓

Deduct Amount

↓

Update Balance

↓

Print Receipt
```

Java works similarly.

Instead of sending many SQL queries,

it simply calls

```sql
CALL withdrawMoney(...)
```

The database performs all the work.

---

# Why Stored Procedures?

Without Stored Procedure

Java sends

```sql
SELECT ...

UPDATE ...

INSERT ...

DELETE ...
```

Many SQL statements.

---

With Stored Procedure

Java sends only

```sql
CALL employeeProcedure();
```

Database executes all SQL internally.

---

# Advantages

* Faster execution
* Better security
* Reusable logic
* Less Java code
* Centralized business logic
* Easier maintenance

---

# CallableStatement Architecture

```text
Java Program

↓

CallableStatement

↓

Stored Procedure

↓

Database

↓

Result
```

---

# Database Table

```sql
CREATE TABLE employee(

id INT PRIMARY KEY AUTO_INCREMENT,

name VARCHAR(100),

age INT,

department VARCHAR(100)

);
```

---

Sample Data

```text
---------------------------------------
1   Usama    24   IT
2   Ali      22   HR
3   Sara     26   Finance
---------------------------------------
```

---

# Creating Our First Stored Procedure

```sql
CREATE PROCEDURE getEmployees()

SELECT * FROM employee;
```

Some MySQL clients require wrapping the procedure with `BEGIN ... END`, especially when it contains multiple SQL statements.

Example:

```sql
DELIMITER //

CREATE PROCEDURE getEmployees()
BEGIN
    SELECT * FROM employee;
END //

DELIMITER ;
```

---

# Understanding

`CREATE PROCEDURE`

Create a procedure.

---

`getEmployees()`

Procedure Name.

---

`SELECT * FROM employee`

SQL stored permanently.

---

Now database remembers it.

Java only needs

```sql
CALL getEmployees();
```

---

# JDBC Syntax

```java
CallableStatement callableStatement =
connection.prepareCall(
"{CALL getEmployees()}"
);
```

Notice

```text
{CALL ...}
```

JDBC uses escape syntax for stored procedure calls.

---

# Complete Java Program

```java
import java.sql.*;

public class Main {

    private static final String url =
            "jdbc:mysql://localhost:3306/mydb";

    private static final String username = "root";

    private static final String password = "password";

    public static void main(String[] args) {

        try {

            Connection connection =
                    DriverManager.getConnection(
                            url,
                            username,
                            password
                    );

            CallableStatement callableStatement =
                    connection.prepareCall(
                            "{CALL getEmployees()}"
                    );

            ResultSet resultSet =
                    callableStatement.executeQuery();

            while(resultSet.next()){

                System.out.println(

                        resultSet.getInt("id")+" "

                        +

                        resultSet.getString("name")+" "

                        +

                        resultSet.getInt("age")+" "

                        +

                        resultSet.getString("department")

                );

            }

            resultSet.close();

            callableStatement.close();

            connection.close();

        }

        catch(Exception e){

            e.printStackTrace();

        }

    }

}
```

---

# Understanding Every Line

---

## Prepare Call

```java
CallableStatement callableStatement =
connection.prepareCall(
"{CALL getEmployees()}"
);
```

Connection creates a CallableStatement.

Unlike

```java
createStatement()
```

or

```java
prepareStatement()
```

we use

```java
prepareCall()
```

because we are calling a stored procedure.

---

## Why Curly Braces?

```java
"{CALL getEmployees()}"
```

This is JDBC's standard escape syntax.

It tells the JDBC driver

> This is a stored procedure call.

---

## Execute

```java
ResultSet resultSet =
callableStatement.executeQuery();
```

Procedure contains

```sql
SELECT
```

Therefore

ResultSet is returned.

---

# Procedure with IN Parameter

Suppose

We want only one employee.

---

MySQL

```sql
CREATE PROCEDURE getEmployeeById(IN empId INT)
BEGIN

SELECT *

FROM employee

WHERE id=empId;

END;
```

---

Meaning

Procedure expects one integer.

---

Java

```java
CallableStatement cs =
connection.prepareCall(
"{CALL getEmployeeById(?)}"
);

cs.setInt(1,2);

ResultSet rs =
cs.executeQuery();
```

---

Explanation

```text
?

↓

Placeholder

↓

Value

↓

2
```

Exactly like PreparedStatement.

---

# Procedure with OUT Parameter

Suppose database should return

Total Employees.

---

MySQL

```sql
CREATE PROCEDURE totalEmployee(
OUT total INT
)
BEGIN

SELECT COUNT(*)

INTO total

FROM employee;

END;
```

---

Java

```java
CallableStatement cs =
connection.prepareCall(
"{CALL totalEmployee(?)}"
);

cs.registerOutParameter(
1,
Types.INTEGER
);

cs.execute();

int total =
cs.getInt(1);

System.out.println(total);
```

---

Explanation

### registerOutParameter()

Java tells JDBC

> "The first parameter will return an integer."

---

After execution

Read it

```java
cs.getInt(1);
```

---

# Procedure with INOUT Parameter

MySQL

```sql
CREATE PROCEDURE increaseAge(
INOUT age INT
)
BEGIN

SET age = age + 1;

END;
```

---

Java

```java
CallableStatement cs =
connection.prepareCall(
"{CALL increaseAge(?)}"
);

cs.setInt(1,24);

cs.registerOutParameter(
1,
Types.INTEGER
);

cs.execute();

System.out.println(
cs.getInt(1)
);
```

Output

```text
25
```

---

Meaning

Parameter acts as

Input

and

Output.

---

# executeQuery() vs executeUpdate() vs execute()

CallableStatement supports all three.

### executeQuery()

When procedure returns rows.

Example

```sql
SELECT
```

---

### executeUpdate()

When procedure performs only INSERT, UPDATE, or DELETE and returns an update count.

---

### execute()

When you're not sure whether the procedure will return a `ResultSet`, an update count, multiple results, or OUT parameters. It returns a `boolean` indicating whether the first result is a `ResultSet`.

---

# Difference

| Interface         | Used For                               |
| ----------------- | -------------------------------------- |
| Statement         | Static SQL                             |
| PreparedStatement | Parameterized SQL                      |
| CallableStatement | Stored Procedures and Stored Functions |

---

# Internal Flow

```text
Java Program

↓

Connection

↓

prepareCall()

↓

CallableStatement

↓

Stored Procedure

↓

Database

↓

ResultSet / OUT Parameter

↓

Java Program
```

---

# Real-World Examples

Stored Procedures are commonly used for:

* Bank transactions
* Salary calculation
* Student result generation
* Inventory management
* Attendance systems
* Payroll processing
* Online shopping order processing

---

# Interview Questions

### Why use CallableStatement?

To execute stored procedures and stored functions from Java.

---

### Which method creates CallableStatement?

```java
connection.prepareCall();
```

---

### Which method is used for OUT parameters?

```java
registerOutParameter();
```

---

### Difference between PreparedStatement and CallableStatement?

PreparedStatement executes SQL written in Java.

CallableStatement executes stored procedures already stored inside the database.

---

### Can CallableStatement use placeholders?

Yes.

Exactly like PreparedStatement.

```java
{CALL getEmployee(?)}
```

---

### Can CallableStatement return a ResultSet?

Yes, if the stored procedure executes a `SELECT` statement.

---
* Performance comparison between normal execution and batch execution
* Real-world examples (bulk student import, employee upload, CSV insertion)
