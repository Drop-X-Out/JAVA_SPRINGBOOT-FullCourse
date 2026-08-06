# Lecture 04: `PreparedStatement`

# Recap of Previous Lecture

Previously we learned

```text
Statement

↓

executeQuery()

↓

SELECT
```

and

```text
Statement

↓

executeUpdate()

↓

INSERT
UPDATE
DELETE
```

Everything worked perfectly.

Then why did Java create another interface called **PreparedStatement**?

---

# The Problem with Statement

Suppose you want to insert a student.

You ask the user

```text
Enter Name :
```

User enters

```text
Usama
```

Age

```text
24
```

Department

```text
IT
```

Now we create SQL like this

```java
String query =
"INSERT INTO employee(name,age,department) VALUES('"
+ name + "',"
+ age + ",'"
+ department + "')";
```

If

```text
name = Usama
```

Query becomes

```sql
INSERT INTO employee(name,age,department)
VALUES('Usama',24,'IT');
```

Everything works.

---

# But What If the User Is Malicious?

Suppose user enters

```text
Usama'); DROP TABLE employee; --
```

Now Java creates

```sql
INSERT INTO employee(name,age,department)
VALUES('Usama'); DROP TABLE employee; --',24,'IT');
```

Database may interpret this as two SQL statements.

One inserts data.

The second deletes the table.

This type of attack is called **SQL Injection**.

> Modern MySQL JDBC drivers usually do **not** execute multiple SQL statements in a single `Statement` by default unless explicitly enabled. However, concatenating user input into SQL is still dangerous and can allow SQL injection in many forms (such as bypassing authentication or altering query logic).

---

# Another Example (Login System)

Suppose query

```java
String query =
"SELECT * FROM users WHERE username='"
+ username +
"' AND password='"
+ password + "'";
```

Normal input

```text
Username

Usama

Password

1234
```

Query becomes

```sql
SELECT *
FROM users
WHERE username='Usama'
AND password='1234';
```

Perfect.

---

Now attacker enters

Username

```text
' OR '1'='1
```

Password

Anything

Query becomes

```sql
SELECT *
FROM users
WHERE username=''
OR '1'='1'
AND password='abc';
```

Since `'1'='1'` is always true, the login logic may be bypassed depending on how the query is written.

This is one of the most common SQL Injection attacks.

---

# Why Does SQL Injection Happen?

Because SQL and user data are mixed together.

```java
String query =
"SELECT * FROM employee WHERE name='"
+ name + "'";
```

Java builds SQL using user input.

Database cannot distinguish

* SQL Command
* User Data

Everything becomes one SQL statement.

---

# Solution

Java introduced

```text
PreparedStatement
```

Instead of inserting values directly,

we use

```text
?
```

called a **Placeholder** or **Parameter Marker**.

---

# What is PreparedStatement?

**Definition**

`PreparedStatement` is a sub-interface of `Statement` used to execute **precompiled**, **parameterized** SQL queries safely and efficiently.

---

# Real Life Analogy

Imagine a passport application form.

The form already contains fixed fields

```text
Name : _______

Age : _______

Country : _______
```

Only the blanks change.

Similarly,

SQL structure remains fixed.

Only values change.

---

# Placeholder

Instead of

```sql
INSERT INTO employee
VALUES('Usama',24,'IT');
```

we write

```sql
INSERT INTO employee(name,age,department)
VALUES(?,?,?);
```

Question marks represent values.

---

# How Values Are Added?

```java
preparedStatement.setString(1,"Usama");
preparedStatement.setInt(2,24);
preparedStatement.setString(3,"IT");
```

Database knows

These are **data values**, not SQL commands.

---

# Complete Flow

```text
Java Program

↓

SQL with ?

↓

PreparedStatement

↓

Bind Values

↓

Database

↓

Execute
```

---

# Example 1: INSERT using PreparedStatement

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

            String query =
                    "INSERT INTO employee(name,age,department) VALUES(?,?,?)";

            PreparedStatement preparedStatement =
                    connection.prepareStatement(query);

            preparedStatement.setString(1,"Usama");
            preparedStatement.setInt(2,24);
            preparedStatement.setString(3,"IT");

            int rows =
                    preparedStatement.executeUpdate();

            System.out.println(
                    rows + " Row Inserted Successfully."
            );

            preparedStatement.close();
            connection.close();

        } catch(Exception e){

            e.printStackTrace();

        }
    }
}
```

---

# Understanding Every Line

## SQL Query

```java
String query =
"INSERT INTO employee(name,age,department)
VALUES(?,?,?)";
```

Three placeholders.

One for

Name

Age

Department

---

## Create PreparedStatement

```java
PreparedStatement preparedStatement =
connection.prepareStatement(query);
```

Database receives the SQL template.

The SQL structure is prepared before values are supplied.

---

## Set String

```java
preparedStatement.setString(1,"Usama");
```

Meaning

First placeholder

↓

Usama

Index starts from **1**, not **0**.

---

## Set Integer

```java
preparedStatement.setInt(2,24);
```

Second placeholder

↓

24

---

## Set String

```java
preparedStatement.setString(3,"IT");
```

Third placeholder

↓

IT

---

## Execute

```java
preparedStatement.executeUpdate();
```

No SQL String is passed here.

Why?

Because SQL was already supplied when the `PreparedStatement` was created.

Only the values were filled later.

---

# Example 2: SELECT using PreparedStatement

```java
import java.sql.*;

public class Main {

    public static void main(String[] args) {

        try {

            Connection connection =
                    DriverManager.getConnection(
                            "jdbc:mysql://localhost:3306/mydb",
                            "root",
                            "password"
                    );

            String query =
                    "SELECT * FROM employee WHERE id=?";

            PreparedStatement preparedStatement =
                    connection.prepareStatement(query);

            preparedStatement.setInt(1,2);

            ResultSet resultSet =
                    preparedStatement.executeQuery();

            while(resultSet.next()){

                System.out.println(
                        resultSet.getInt("id")+" "+
                        resultSet.getString("name")
                );

            }

            resultSet.close();
            preparedStatement.close();
            connection.close();

        } catch(Exception e){

            e.printStackTrace();

        }
    }
}
```

---

# Example 3: UPDATE

```java
String query =
"UPDATE employee SET department=? WHERE id=?";

PreparedStatement ps =
connection.prepareStatement(query);

ps.setString(1,"Development");
ps.setInt(2,2);

int rows =
ps.executeUpdate();
```

---

# Example 4: DELETE

```java
String query =
"DELETE FROM employee WHERE id=?";

PreparedStatement ps =
connection.prepareStatement(query);

ps.setInt(1,5);

int rows =
ps.executeUpdate();
```

---

# Common `setXXX()` Methods

| Method           | Used For                     |
| ---------------- | ---------------------------- |
| `setInt()`       | int                          |
| `setString()`    | String                       |
| `setDouble()`    | double                       |
| `setBoolean()`   | boolean                      |
| `setFloat()`     | float                        |
| `setLong()`      | long                         |
| `setDate()`      | SQL Date                     |
| `setTimestamp()` | Date and Time                |
| `setObject()`    | Any object (supported types) |

---

# Why Does Index Start from 1?

Suppose

```sql
INSERT INTO employee
VALUES(?,?,?)
```

Java numbers placeholders as

```text
1

2

3
```

There is no placeholder number 0.

This is defined by the JDBC specification.

---

# Advantages of PreparedStatement

* Prevents SQL Injection.
* Easier to read and maintain.
* SQL structure remains separate from user data.
* Supports different data types with `setXXX()` methods.
* Can be reused with different values.
* Often performs better for repeated execution because the SQL statement can be prepared once and executed multiple times with different parameters.

---

# Statement vs PreparedStatement

| Feature       | Statement                      | PreparedStatement              |
| ------------- | ------------------------------ | ------------------------------ |
| SQL Query     | Built by concatenating Strings | Uses placeholders (`?`)        |
| SQL Injection | Vulnerable                     | Protected by parameter binding |
| Performance   | Lower for repeated execution   | Better for repeated execution  |
| Readability   | Less                           | More                           |
| User Input    | Unsafe                         | Safe                           |
| Reusable      | No                             | Yes                            |

---

# Internal Working

```text
Java Program

↓

SQL with ?

↓

PreparedStatement Created

↓

Values Bound

↓

SQL Sent to Database

↓

Database Executes

↓

Result Returned
```

---

# Interview Questions

### Why is PreparedStatement more secure?

Because SQL code and user data are handled separately through parameter binding, preventing user input from being treated as SQL commands.

---

### Why doesn't `executeUpdate()` take a query in PreparedStatement?

Because the SQL statement was already provided to `prepareStatement()` when the object was created.

---

### Can we use PreparedStatement for SELECT?

Yes.

Use

```java
executeQuery()
```

---

### Can we use PreparedStatement for INSERT?

Yes.

Use

```java
executeUpdate()
```

---

### Can PreparedStatement execute the same query multiple times?

Yes.

You can change parameter values using `setXXX()` methods and execute the same prepared SQL statement repeatedly.

---
* Batch operations using `PreparedStatement`
* Real-world examples (registration form, login system, employee management)
