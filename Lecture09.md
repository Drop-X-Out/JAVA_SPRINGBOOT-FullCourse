# Transaction Management in JDBC

## 1. What is a Transaction?

A **transaction** is a group of one or more database operations that are treated as a **single unit of work**.

The basic rule is:

> **Either all operations in a transaction are successfully completed, or none of them should be permanently applied.**

For example, consider a bank transfer:

```text
John → Jack : ₹500
```

This requires two database operations:

```text
1. Deduct ₹500 from John's account
2. Add ₹500 to Jack's account
```

These two operations are logically one operation: **Transfer ₹500 from John to Jack**.

If the first operation succeeds but the second operation fails, the database should not keep the first change.

That is exactly what transaction management solves.

---

# 2. Why Do We Need Transactions?

Consider the following situation.

Initial balances:

```text
John = ₹5,000
Jack = ₹3,000
```

John wants to transfer ₹500 to Jack.

The application performs:

```sql
UPDATE account
SET balance = balance - 500
WHERE id = 1;
```

John's balance becomes:

```text
John = ₹4,500
```

Now the application executes:

```sql
UPDATE account
SET balance = balance + 500
WHERE id = 2;
```

But suppose this operation fails because Jack's account does not exist.

Now the database may contain:

```text
John = ₹4,500
Jack = ₹3,000
```

The ₹500 has effectively disappeared.

This is an inconsistent state.

A transaction allows us to say:

```text
Deduct money
      +
Add money
      ↓
Both successful → COMMIT
Any failure      → ROLLBACK
```

---

# 3. Transaction in Simple Terms

A transaction can be thought of as:

```text
START
  ↓
Operation 1
  ↓
Operation 2
  ↓
Operation 3
  ↓
Everything successful?
  ↓
 ┌───────────────┐
 │               │
YES             NO
 │               │
 ↓               ↓
COMMIT        ROLLBACK
 │               │
 ↓               ↓
SAVE            UNDO
```

The important idea is:

```text
COMMIT   → Permanently save the transaction
ROLLBACK → Undo the transaction
```

---

# 4. JDBC and Transactions

JDBC provides transaction management through the `Connection` object.

The three most important methods are:

```java
con.setAutoCommit(false);
con.commit();
con.rollback();
```

### `setAutoCommit(false)`

Disables automatic committing.

```java
con.setAutoCommit(false);
```

Now the application controls when the transaction is committed.

---

### `commit()`

Permanently saves all changes made in the current transaction.

```java
con.commit();
```

---

### `rollback()`

Cancels the changes made in the current transaction.

```java
con.rollback();
```

---

# 5. What is Auto-Commit?

By default, a JDBC connection normally has:

```java
autoCommit = true
```

This means each SQL statement is automatically committed after successful execution.

Conceptually:

```text
SQL Operation 1
      ↓
   COMMIT

SQL Operation 2
      ↓
   COMMIT

SQL Operation 3
      ↓
   COMMIT
```

This is useful for simple independent operations.

For example:

```sql
INSERT INTO student VALUES (1, 'John', 90);
```

If this is an independent operation, automatically committing it may be perfectly fine.

---

# 6. Why Disable Auto-Commit?

Suppose multiple SQL operations must succeed together.

For example:

```text
Bank Transfer

1. Deduct ₹500 from Account A
2. Add ₹500 to Account B
```

We don't want:

```text
Operation 1 → COMMIT
Operation 2 → ERROR
```

Instead, we want:

```text
Operation 1
     ↓
Operation 2
     ↓
COMMIT
```

or:

```text
Operation 1
     ↓
Operation 2
     ↓
ERROR
     ↓
ROLLBACK
```

Therefore:

```java
con.setAutoCommit(false);
```

is used.

---

# 7. Transaction Lifecycle

A typical JDBC transaction looks like this:

```java
try {

    con.setAutoCommit(false);

    // Database operation 1
    // Database operation 2
    // Database operation 3

    con.commit();

} catch (Exception e) {

    con.rollback();

}
```

The flow is:

```text
Connection
    ↓
setAutoCommit(false)
    ↓
Transaction begins
    ↓
Execute SQL operations
    ↓
    ┌───────────────┐
    │               │
Success          Failure
    │               │
    ↓               ↓
 COMMIT          ROLLBACK
    │               │
    ↓               ↓
 SAVE             UNDO
```

---

# 8. `executeUpdate()` vs `commit()`

These two methods should not be confused.

Consider:

```java
ps.executeUpdate();
```

This means:

> Execute the SQL statement.

Whereas:

```java
con.commit();
```

means:

> Permanently save the changes belonging to the current transaction.

They have different responsibilities.

```text
executeUpdate()
       ↓
Execute SQL operation
       ↓
commit()
       ↓
Permanently save transaction
```

---

# 9. Complete Example: Bank Transfer

We will create a simple bank database.

## Step 1: Create Database

```sql
CREATE DATABASE bankdb;
```

Select the database:

```sql
USE bankdb;
```

---

## Step 2: Create Account Table

```sql
CREATE TABLE account (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    balance DOUBLE
);
```

---

## Step 3: Insert Sample Data

```sql
INSERT INTO account VALUES
(1, 'John', 5000),
(2, 'Jack', 3000),
(3, 'Bob', 7000);
```

The table now contains:

| ID | Name | Balance |
| -: | ---- | ------: |
|  1 | John |    5000 |
|  2 | Jack |    3000 |
|  3 | Bob  |    7000 |

---

# 10. Full JDBC Transaction Code

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class BankTransfer {

    public static void main(String[] args) {

        String url = "jdbc:mysql://localhost:3306/bankdb";
        String user = "root";
        String password = "root";

        Connection con = null;

        try {

            // 1. Establish database connection
            con = DriverManager.getConnection(
                    url,
                    user,
                    password
            );

            // 2. Disable auto-commit
            con.setAutoCommit(false);

            // 3. Check sender's balance
            String balanceSQL =
                    "SELECT balance FROM account WHERE id = ?";

            PreparedStatement balancePS =
                    con.prepareStatement(balanceSQL);

            balancePS.setInt(1, 1);

            ResultSet rs = balancePS.executeQuery();

            if (!rs.next()) {
                throw new Exception("Sender account not found");
            }

            double balance = rs.getDouble("balance");

            double amount = 500;

            // 4. Check sufficient balance
            if (balance < amount) {
                throw new Exception("Insufficient balance");
            }

            // 5. Deduct money from sender
            String withdrawSQL =
                    "UPDATE account " +
                    "SET balance = balance - ? " +
                    "WHERE id = ?";

            PreparedStatement withdrawPS =
                    con.prepareStatement(withdrawSQL);

            withdrawPS.setDouble(1, amount);
            withdrawPS.setInt(2, 1);

            int withdrawRows =
                    withdrawPS.executeUpdate();

            if (withdrawRows != 1) {
                throw new Exception("Money deduction failed");
            }

            // 6. Add money to receiver
            String depositSQL =
                    "UPDATE account " +
                    "SET balance = balance + ? " +
                    "WHERE id = ?";

            PreparedStatement depositPS =
                    con.prepareStatement(depositSQL);

            depositPS.setDouble(1, amount);
            depositPS.setInt(2, 2);

            int depositRows =
                    depositPS.executeUpdate();

            if (depositRows != 1) {
                throw new Exception("Receiver account not found");
            }

            // 7. Both operations succeeded
            con.commit();

            System.out.println("Money transferred successfully!");

        } catch (Exception e) {

            System.out.println("Transaction failed!");

            try {

                if (con != null) {
                    con.rollback();
                }

            } catch (Exception rollbackException) {

                rollbackException.printStackTrace();
            }

            System.out.println(e.getMessage());

        } finally {

            try {

                if (con != null) {
                    con.close();
                }

            } catch (Exception e) {

                e.printStackTrace();
            }
        }
    }
}
```

---

# 11. Understanding the Code Step by Step

## Step 1: Create Connection

```java
con = DriverManager.getConnection(
        url,
        user,
        password
);
```

This establishes a connection between Java and the database.

---

## Step 2: Disable Auto-Commit

```java
con.setAutoCommit(false);
```

This is the beginning of manual transaction control.

Instead of:

```text
SQL → Automatically Commit
```

we now have:

```text
SQL
 ↓
Wait
 ↓
More SQL
 ↓
Wait
 ↓
commit()
```

---

# 12. Check Sender Balance

```java
String balanceSQL =
        "SELECT balance FROM account WHERE id = ?";
```

We use `PreparedStatement` because the account ID is supplied dynamically.

```java
PreparedStatement balancePS =
        con.prepareStatement(balanceSQL);

balancePS.setInt(1, 1);
```

Then:

```java
ResultSet rs = balancePS.executeQuery();
```

We retrieve the sender's balance.

---

# 13. Check Whether Account Exists

```java
if (!rs.next()) {
    throw new Exception("Sender account not found");
}
```

If no account exists, an exception is generated.

The exception eventually reaches the `catch` block.

The transaction will then be rolled back.

---

# 14. Check Sufficient Balance

```java
if (balance < amount) {
    throw new Exception("Insufficient balance");
}
```

For example:

```text
Balance = ₹300
Transfer = ₹500
```

The transfer cannot happen.

So we stop the transaction.

---

# 15. Deduct Money

```java
String withdrawSQL =
        "UPDATE account " +
        "SET balance = balance - ? " +
        "WHERE id = ?";
```

Values:

```java
withdrawPS.setDouble(1, amount);
withdrawPS.setInt(2, 1);
```

Then:

```java
withdrawPS.executeUpdate();
```

Suppose:

```text
John = ₹5000
```

After the SQL operation:

```text
John = ₹4500
```

But remember:

```java
con.setAutoCommit(false);
```

So we have not explicitly committed the transaction yet.

---

# 16. Add Money to Receiver

Now:

```java
String depositSQL =
        "UPDATE account " +
        "SET balance = balance + ? " +
        "WHERE id = ?";
```

For Jack:

```java
depositPS.setDouble(1, amount);
depositPS.setInt(2, 2);
```

Then:

```java
depositPS.executeUpdate();
```

Now:

```text
John = ₹4500
Jack = ₹3500
```

---

# 17. Commit

If everything worked:

```java
con.commit();
```

This means:

> The complete transaction was successful. Permanently save these changes.

Final result:

```text
John = ₹4500
Jack = ₹3500
```

---

# 18. What Happens If Something Fails?

Suppose the receiver account doesn't exist.

```text
John = ₹5000
Jack = ₹3000
```

The program performs:

```text
Deduct ₹500 from John
        ↓
John = ₹4500
        ↓
Try to add ₹500 to receiver
        ↓
Receiver doesn't exist
        ↓
Exception
```

The exception takes execution to:

```java
catch (Exception e)
```

Inside it:

```java
con.rollback();
```

The database goes back to the state before the transaction.

```text
John = ₹5000
Jack = ₹3000
```

The deduction is undone.

---

# 19. Why `rollback()` Is Important

Without rollback, the first operation could remain applied even though the second operation failed.

### Without Transaction Management

```text
Deduct ₹500
    ↓
COMMIT
    ↓
Add ₹500
    ↓
ERROR
```

Result:

```text
John loses ₹500
Jack doesn't receive ₹500
```

### With Transaction Management

```text
Deduct ₹500
    ↓
Add ₹500
    ↓
ERROR
    ↓
ROLLBACK
```

Result:

```text
John gets his ₹500 back
```

That is the fundamental purpose of transactions.

---

# 20. Why Not Just Use JDBC Batch?

A common question is:

> If there are multiple SQL operations, why not use JDBC Batch instead of transactions?

Because **Batch and Transaction solve different problems**.

## Batch

Batch processing is primarily about grouping multiple SQL operations for execution.

Example:

```java
PreparedStatement ps = con.prepareStatement(
        "UPDATE student SET marks = ? WHERE id = ?"
);

ps.setInt(1, 90);
ps.setInt(2, 1);
ps.addBatch();

ps.setInt(1, 80);
ps.setInt(2, 2);
ps.addBatch();

ps.setInt(1, 70);
ps.setInt(2, 3);
ps.addBatch();

ps.executeBatch();
```

The purpose is:

```text
Many SQL operations
        ↓
Group them
        ↓
Execute as a batch
```

---

# 21. Transaction vs Batch

| Transaction                             | Batch                                                      |
| --------------------------------------- | ---------------------------------------------------------- |
| Provides atomicity                      | Groups multiple operations                                 |
| Controls commit/rollback                | Helps execute many operations efficiently                  |
| Uses `commit()`                         | Uses `executeBatch()`                                      |
| Uses `rollback()`                       | Uses `addBatch()`                                          |
| Concerned with correctness              | Concerned primarily with execution efficiency              |
| Answers: "Should all changes be saved?" | Answers: "Can these operations be sent/executed together?" |

The simplest distinction is:

```text
BATCH
"What is an efficient way to execute many operations?"

TRANSACTION
"What should happen if one of these operations fails?"
```

---

# 22. Can Batch and Transaction Be Used Together?

Yes.

In fact, they can be used together.

For example:

```java
con.setAutoCommit(false);

PreparedStatement ps = con.prepareStatement(
        "UPDATE student SET marks = ? WHERE id = ?"
);

ps.setInt(1, 90);
ps.setInt(2, 1);
ps.addBatch();

ps.setInt(1, 80);
ps.setInt(2, 2);
ps.addBatch();

ps.setInt(1, 70);
ps.setInt(2, 3);
ps.addBatch();

ps.executeBatch();

con.commit();
```

Here:

```text
Batch
  ↓
Groups/executes multiple SQL operations

Transaction
  ↓
Controls whether the overall changes are committed
```

So:

> **Batch does not replace a transaction.**

---

# 23. Why Batch Alone Is Not Enough for a Bank Transfer

Consider:

```text
John → Jack ₹500
```

We have two different operations:

```sql
UPDATE account
SET balance = balance - 500
WHERE id = 1;
```

and:

```sql
UPDATE account
SET balance = balance + 500
WHERE id = 2;
```

Even if these operations are executed as a batch, the application still needs transaction control if they must behave as one atomic operation.

The requirement is:

```text
Deduct money
      +
Deposit money
      ↓
Both successful
      ↓
COMMIT
```

If something goes wrong:

```text
Any operation fails
      ↓
ROLLBACK
      ↓
Undo previous changes
```

Batch does not change the fundamental requirement for transaction management.

---

# 24. Batch and Transaction Can Be Visualized Like This

```text
                 DATABASE
                    │
             JDBC Connection
                    │
             ┌──────┴──────┐
             │ Transaction │
             └──────┬──────┘
                    │
             ┌──────┴──────┐
             │             │
          Batch          Single SQL
             │
       ┌─────┼─────┐
       ↓     ↓     ↓
      SQL   SQL   SQL
             │
             ↓
          COMMIT
```

A batch can exist **inside a transaction**.

They are not alternatives.

---

# 25. ACID Properties

Transactions are commonly described using the **ACID** properties.

## A — Atomicity

All operations succeed or none of them are applied.

Example:

```text
Deduct ₹500
+
Deposit ₹500
```

Both should happen, or both should be undone.

---

## C — Consistency

A transaction should take the database from one valid state to another valid state.

For example, a bank transfer should not cause money to mysteriously disappear.

---

## I — Isolation

Transactions running concurrently should not incorrectly interfere with one another.

Multiple users may be accessing the database at the same time, and the database provides mechanisms to control how their transactions interact.

---

## D — Durability

Once a transaction has been successfully committed:

```java
con.commit();
```

the committed changes should remain stored even if the application subsequently crashes.

---

# 26. ACID in One Example

For the bank transfer:

```text
John = ₹5000
Jack = ₹3000
```

Transfer:

```text
₹500
```

### Atomicity

```text
Deduct + Deposit
```

Both or none.

### Consistency

Total money remains:

```text
₹8000
```

### Isolation

Other transactions should not incorrectly interfere with this transfer.

### Durability

After:

```java
con.commit();
```

the successful transfer remains saved.

---

# 27. Important JDBC Transaction Pattern

A basic transaction generally follows this structure:

```java
try {

    con.setAutoCommit(false);

    // Operation 1

    // Operation 2

    // Operation 3

    con.commit();

} catch (Exception e) {

    con.rollback();

}
```

A safer version is:

```java
try {

    con.setAutoCommit(false);

    // Database operations

    con.commit();

} catch (Exception e) {

    try {

        if (con != null) {
            con.rollback();
        }

    } catch (Exception rollbackException) {

        rollbackException.printStackTrace();
    }

} finally {

    try {

        if (con != null) {
            con.close();
        }

    } catch (Exception e) {

        e.printStackTrace();
    }
}
```

---

# 28. Important Points to Remember

### 1. A transaction is a logical unit of work

```text
Multiple related operations
          ↓
       Transaction
```

### 2. Disable auto-commit when manual transaction control is required

```java
con.setAutoCommit(false);
```

### 3. Use commit when everything succeeds

```java
con.commit();
```

### 4. Use rollback when something fails

```java
con.rollback();
```

### 5. Transactions belong to the `Connection`

```java
con.commit();
con.rollback();
```

not to individual `PreparedStatement` objects.

### 6. Batch does not replace transactions

```text
Batch       → Group/execute many operations efficiently
Transaction → Control commit/rollback and atomicity
```

### 7. Batch and transactions can be used together

```text
Transaction
     ↓
   Batch
     ↓
Many SQL operations
     ↓
  COMMIT
```

---

# 29. Final Mental Model

Remember the following flow:

```text
                 JDBC CONNECTION
                        │
                        ↓
              setAutoCommit(false)
                        │
                        ↓
                  TRANSACTION
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       SQL 1          SQL 2          SQL 3
          │             │             │
          └─────────────┼─────────────┘
                        ↓
                 Everything OK?
                   /         \
                 YES          NO
                  ↓            ↓
               COMMIT       ROLLBACK
                  ↓            ↓
                SAVE          UNDO
```

And if there are many operations:

```text
                 TRANSACTION
                      │
                      ↓
                    BATCH
                      │
             ┌────────┼────────┐
             ↓        ↓        ↓
            SQL      SQL      SQL
             └────────┼────────┘
                      ↓
                   COMMIT
```

The core idea is:

> **A batch is about executing multiple SQL operations together. A transaction is about treating related database operations as one logical unit, where successful work is committed and failed work can be rolled back.**
