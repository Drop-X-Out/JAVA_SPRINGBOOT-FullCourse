## Date Operations in Hibernate — JDK 8 Date & Time API

In Hibernate, **JDK 8 Date & Time API** gives us three important classes:

| Java Class      | Represents  | Example               |
| --------------- | ----------- | --------------------- |
| `LocalDate`     | Date only   | `2026-08-23`          |
| `LocalTime`     | Time only   | `21:30:45`            |
| `LocalDateTime` | Date + Time | `2026-08-23T21:30:45` |

Hibernate can map these directly to SQL date/time columns using modern Hibernate versions.

### 1. `LocalDate`

Use it when you only need a **date**, such as:

* Date of birth
* Admission date
* Exam date
* Joining date

```java
private LocalDate dateOfBirth;
```

Typical database mapping:

```sql
DATE
```

Example:

```java
student.setDateOfBirth(LocalDate.of(2002, 5, 15));
```

---

### 2. `LocalTime`

Use it when you only need a **time**.

```java
private LocalTime classTime;
```

Typical database mapping:

```sql
TIME
```

Example:

```java
student.setClassTime(LocalTime.of(10, 30, 0));
```

---

### 3. `LocalDateTime`

Use it when you need **both date and time**.

```java
private LocalDateTime createdAt;
```

Typical database mapping:

```sql
DATETIME
```

Example:

```java
student.setCreatedAt(LocalDateTime.of(2026, 8, 23, 21, 30));
```

---

## Complete Hibernate Example

### Folder Structure

```text
HibernateDateDemo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │   └── com/example/
        │       ├── HibernateUtil.java
        │       ├── Student.java
        │       └── App.java
        │
        └── resources/
            └── hibernate.cfg.xml
```

### `Student.java`

```java
package com.example;

import jakarta.persistence.*;
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.LocalDateTime;

@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;

    // Stores only date
    private LocalDate dateOfBirth;

    // Stores only time
    private LocalTime classTime;

    // Stores date + time
    private LocalDateTime createdAt;

    public Student() {
    }

    public Student(String name,
                   LocalDate dateOfBirth,
                   LocalTime classTime,
                   LocalDateTime createdAt) {

        this.name = name;
        this.dateOfBirth = dateOfBirth;
        this.classTime = classTime;
        this.createdAt = createdAt;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public LocalDate getDateOfBirth() {
        return dateOfBirth;
    }

    public LocalTime getClassTime() {
        return classTime;
    }

    public LocalDateTime getCreatedAt() {
        return createdAt;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setDateOfBirth(LocalDate dateOfBirth) {
        this.dateOfBirth = dateOfBirth;
    }

    public void setClassTime(LocalTime classTime) {
        this.classTime = classTime;
    }

    public void setCreatedAt(LocalDateTime createdAt) {
        this.createdAt = createdAt;
    }
}
```

### Creating the object

```java
Student student = new Student(
        "Rahul",

        LocalDate.of(2002, 5, 15),

        LocalTime.of(10, 30, 0),

        LocalDateTime.now()
);
```

Notice:

```java
LocalDate.of(2002, 5, 15)
```

means:

```text
15 May 2002
```

while:

```java
LocalTime.of(10, 30, 0)
```

means:

```text
10:30:00
```

and:

```java
LocalDateTime.now()
```

gives the current date and time.

---

## Hibernate Configuration

For Hibernate 6.x:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "https://hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/hibernate_db
        </property>

        <property name="hibernate.connection.username">
            root
        </property>

        <property name="hibernate.connection.password">
            root
        </property>

        <property name="hibernate.dialect">
            org.hibernate.dialect.MySQLDialect
        </property>

        <property name="hibernate.hbm2ddl.auto">
            update
        </property>

        <property name="hibernate.show_sql">
            true
        </property>

        <property name="hibernate.format_sql">
            true
        </property>

        <mapping class="com.example.Student"/>

    </session-factory>

</hibernate-configuration>
```

### `HibernateUtil.java`

```java
package com.example;

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

### `App.java`

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;

import java.time.LocalDate;
import java.time.LocalTime;
import java.time.LocalDateTime;

public class App {

    public static void main(String[] args) {

        Student student = new Student(
                "Rahul",

                // Date only
                LocalDate.of(2002, 5, 15),

                // Time only
                LocalTime.of(10, 30, 0),

                // Current date + current time
                LocalDateTime.now()
        );

        Session session =
                HibernateUtil.getSessionFactory().openSession();

        Transaction transaction = session.beginTransaction();

        session.persist(student);

        transaction.commit();

        session.close();

        System.out.println("Student saved successfully!");
    }
}
```

### Database Result

Hibernate will create something conceptually like:

```sql
CREATE TABLE students (
    id INTEGER NOT NULL AUTO_INCREMENT,
    name VARCHAR(255),
    dateOfBirth DATE,
    classTime TIME,
    createdAt DATETIME,
    PRIMARY KEY (id)
);
```

And the stored data could look like:

```text
id    name     dateOfBirth    classTime    createdAt
---------------------------------------------------------------
1     Rahul    2002-05-15     10:30:00     2026-08-23 21:30:45
```

### Important concept

Think of the three classes like this:

```text
LocalDate
   ↓
DATE
   ↓
2026-08-23


LocalTime
   ↓
TIME
   ↓
21:30:45


LocalDateTime
   ↓
DATETIME
   ↓
2026-08-23 21:30:45
```

**No timezone is stored** in any of these three classes. If you need timezone information, you would look at `ZonedDateTime`, `OffsetDateTime`, or `Instant`.

# Object Versioning & Timestamping in Hibernate

Hibernate provides three very useful annotations for tracking **when an object was created, when it was updated, and how many times it has been modified**:

| Annotation           | Purpose                                       | Typical Field |
| -------------------- | --------------------------------------------- | ------------- |
| `@CreationTimestamp` | Stores creation date/time                     | `createdAt`   |
| `@UpdateTimestamp`   | Stores last update date/time                  | `updatedAt`   |
| `@Version`           | Tracks object version & prevents lost updates | `version`     |

These are especially useful in real-world applications for **auditing, concurrency control, and tracking changes**.

---

# 1. `@CreationTimestamp`

`@CreationTimestamp` automatically stores the **date and time when the entity is first inserted**.

```java
@CreationTimestamp
private LocalDateTime createdAt;
```

You don't need to manually write:

```java
student.setCreatedAt(LocalDateTime.now());
```

Hibernate handles it.

### Example

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;

    @CreationTimestamp
    private LocalDateTime createdAt;
}
```

When you execute:

```java
session.persist(student);
```

Hibernate automatically sets:

```text
createdAt = 2026-08-23 21:30:15
```

---

# 2. `@UpdateTimestamp`

`@UpdateTimestamp` stores the **last time the entity was updated**.

```java
@UpdateTimestamp
private LocalDateTime updatedAt;
```

For example:

### Initially

```text
id = 1
name = Rahul
createdAt = 2026-08-23 10:00:00
updatedAt = 2026-08-23 10:00:00
```

Then you change:

```java
student.setName("Rahul Kumar");
```

and commit:

```java
transaction.commit();
```

Hibernate automatically updates:

```text
updatedAt = 2026-08-23 11:15:30
```

The `createdAt` remains unchanged.

---

# 3. `@Version`

This one is slightly different.

`@Version` is used for **optimistic locking**.

It prevents the classic problem of:

> Two users modifying the same database record at the same time.

Example:

```java
@Version
private int version;
```

Initially:

```text
version = 0
```

After the first successful update:

```text
version = 1
```

After another update:

```text
version = 2
```

And so on.

---

# Why Do We Need `@Version`?

Imagine we have:

```text
Student ID = 1
Name = Rahul
Version = 5
```

Two users open the same student.

### User A

```text
Reads Rahul
Version = 5
```

### User B

```text
Reads Rahul
Version = 5
```

Now User A changes:

```text
Rahul → Rahul Kumar
```

Hibernate executes something conceptually like:

```sql
UPDATE students
SET name = 'Rahul Kumar',
    version = 6
WHERE id = 1
AND version = 5;
```

It succeeds.

Database:

```text
Name = Rahul Kumar
Version = 6
```

---

Now User B tries to save his older copy.

His object still has:

```text
Version = 5
```

Hibernate effectively tries:

```sql
UPDATE students
SET name = 'Rahul Singh',
    version = 6
WHERE id = 1
AND version = 5;
```

But the database now has:

```text
version = 6
```

So:

```text
WHERE version = 5
```

matches **zero rows**.

Hibernate knows that somebody else modified the object.

This results in an optimistic-locking exception, rather than silently overwriting User A's changes.

---

# Complete Example

Let's combine all three.

### `Student.java`

```java
package com.example;

import jakarta.persistence.*;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import java.time.LocalDateTime;

@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;

    private String email;

    // Automatically set when the record is created
    @CreationTimestamp
    private LocalDateTime createdAt;

    // Automatically updated whenever the entity is updated
    @UpdateTimestamp
    private LocalDateTime updatedAt;

    // Automatically managed by Hibernate
    // Used for optimistic locking
    @Version
    private int version;


    public Student() {
    }


    public Student(String name, String email) {
        this.name = name;
        this.email = email;
    }


    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    public LocalDateTime getCreatedAt() {
        return createdAt;
    }

    public LocalDateTime getUpdatedAt() {
        return updatedAt;
    }

    public int getVersion() {
        return version;
    }


    public void setName(String name) {
        this.name = name;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

---

# Saving the Entity

```java
Student student =
        new Student("Rahul", "rahul@gmail.com");

Session session =
        HibernateUtil.getSessionFactory()
                .openSession();

Transaction transaction =
        session.beginTransaction();

session.persist(student);

transaction.commit();

session.close();
```

Hibernate automatically handles:

```text
createdAt
updatedAt
version
```

You don't need to set them manually.

---

# What Happens in the Database?

Suppose we insert:

```java
Student student =
        new Student("Rahul", "rahul@gmail.com");
```

Database:

| id | name  | email                                     | created_at       | updated_at       | version |
| -: | ----- | ----------------------------------------- | ---------------- | ---------------- | ------: |
|  1 | Rahul | [rahul@gmail.com](mailto:rahul@gmail.com) | 2026-08-23 21:30 | 2026-08-23 21:30 |       0 |

Now:

```java
student.setName("Rahul Kumar");
```

Hibernate updates the record.

| id | name        | email                                     | created_at       | updated_at       | version |
| -: | ----------- | ----------------------------------------- | ---------------- | ---------------- | ------: |
|  1 | Rahul Kumar | [rahul@gmail.com](mailto:rahul@gmail.com) | 2026-08-23 21:30 | 2026-08-23 21:45 |       1 |

Notice:

```text
createdAt → unchanged
updatedAt → changed
version   → increased
```

---

# Important Difference

This is the easiest way to remember them:

```text
                ENTITY
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
   CREATED       UPDATED     VERSION
       │           │           │
       ↓           ↓           ↓
@CreationTimestamp @UpdateTimestamp @Version
       │           │           │
       ↓           ↓           ↓
  When created  Last modified  Update count
                              + concurrency
```

### `@CreationTimestamp`

> **When was this object created?**

```java
@CreationTimestamp
private LocalDateTime createdAt;
```

### `@UpdateTimestamp`

> **When was this object last modified?**

```java
@UpdateTimestamp
private LocalDateTime updatedAt;
```

### `@Version`

> **Which version of this object am I working with?**

```java
@Version
private int version;
```

---

# Very Important: `@Version` Is NOT Simply an Update Counter

A common beginner misconception is:

> "`@Version` tells me how many times the object was updated."

It **can look like an update counter**, but its primary purpose is **optimistic concurrency control**.

For example:

```text
version = 0
       ↓
version = 1
       ↓
version = 2
       ↓
version = 3
```

The important purpose is that Hibernate can determine:

> "Has somebody else modified this entity since I read it?"

---

# SQL Generated by Hibernate

With:

```java
@Version
private int version;
```

Hibernate can generate an update conceptually like:

```sql
UPDATE students
SET
    name = ?,
    email = ?,
    updated_at = ?,
    version = ?
WHERE
    id = ?
    AND version = ?
```

The crucial part is:

```sql
AND version = ?
```

This is what enables optimistic locking.

---

# Complete Flow

```text
INSERT
   ↓
Student created
   ↓
@CreationTimestamp
   ↓
createdAt = current timestamp

   ↓

@Version
   ↓
version = 0

   ↓

UPDATE Student
   ↓
@UpdateTimestamp
   ↓
updatedAt = current timestamp

   ↓

@Version
   ↓
version = version + 1
```

So a typical production entity might look like:

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    @CreationTimestamp
    private LocalDateTime createdAt;

    @UpdateTimestamp
    private LocalDateTime updatedAt;

    @Version
    private Long version;
}
```

### One-line memory trick

**`@CreationTimestamp` = Born when?**

**`@UpdateTimestamp` = Changed when?**

**`@Version` = Which version am I editing?**

# Working with LOBs in Hibernate

**LOB** means **Large Object**.

When an application needs to store very large data in a database—such as a long document, PDF, image, audio file, or video—we use LOB types.

In Hibernate/JPA, the two important LOB categories are:

| Annotation / Type | Meaning | Typical Data |
| ----------------- | ------- | ------------ |
| `@Lob` + `String` | CLOB    | Large text   |
| `@Lob` + `byte[]` | BLOB    | Binary data  |

---

## 1. What is a LOB?

LOB = **Large Object**

There are mainly two types:

### CLOB

**Character Large Object**

Used for large amounts of text.

Examples:

```text
Book content
Resume
Article
Terms & conditions
JSON/XML documents
Large descriptions
```

Java:

```java
@Lob
private String content;
```

---

### BLOB

**Binary Large Object**

Used for binary data.

Examples:

```text
Images
PDF files
Audio
Video
Documents
```

Java:

```java
@Lob
private byte[] profilePhoto;
```

---

# 2. Using `@Lob` with String

Suppose we want to store a student's **large biography**.

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @Lob
    private String biography;
}
```

Hibernate understands that:

```java
String
   +
@Lob
```

means:

> Store this as a large character object.

Depending on the database and Hibernate configuration, this is mapped to a suitable CLOB/large-text column.

---

# 3. Using `@Lob` with `byte[]`

Suppose we want to store a student's profile image.

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @Lob
    private byte[] profilePhoto;
}
```

Here:

```java
byte[]
```

contains binary data.

For example:

```text
image.jpg
    ↓
binary data
    ↓
byte[]
    ↓
@Lob
    ↓
BLOB
```

---

# 4. Complete Example

Let's create an entity containing both a large text field and a binary file.

### `Student.java`

```java
package com.example;

import jakarta.persistence.*;

@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // Large text data
    @Lob
    private String biography;

    // Binary data such as image/PDF
    @Lob
    private byte[] profilePhoto;

    public Student() {
    }

    public Student(String name,
                   String biography,
                   byte[] profilePhoto) {

        this.name = name;
        this.biography = biography;
        this.profilePhoto = profilePhoto;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getBiography() {
        return biography;
    }

    public byte[] getProfilePhoto() {
        return profilePhoto;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setBiography(String biography) {
        this.biography = biography;
    }

    public void setProfilePhoto(byte[] profilePhoto) {
        this.profilePhoto = profilePhoto;
    }
}
```

---

# 5. Saving a Text LOB

```java
String biography = """
        Rahul is a Computer Science student.
        He is learning Java, Hibernate and Spring Boot.
        He is interested in backend development.
        """;

Student student =
        new Student(
                "Rahul",
                biography,
                null
        );

Session session =
        HibernateUtil
                .getSessionFactory()
                .openSession();

Transaction transaction =
        session.beginTransaction();

session.persist(student);

transaction.commit();

session.close();
```

The large text is stored in the database.

---

# 6. Saving an Image

Suppose we have:

```text
profile.jpg
```

We can read it into a `byte[]`.

```java
Path path = Paths.get("profile.jpg");

byte[] imageData = Files.readAllBytes(path);
```

Then:

```java
Student student =
        new Student(
                "Rahul",
                "Computer Science Student",
                imageData
        );
```

Persist it:

```java
Session session =
        HibernateUtil
                .getSessionFactory()
                .openSession();

Transaction transaction =
        session.beginTransaction();

session.persist(student);

transaction.commit();

session.close();
```

The image bytes are stored as a BLOB.

---

# 7. Reading the BLOB

Retrieve the student:

```java
Session session =
        HibernateUtil
                .getSessionFactory()
                .openSession();

Student student =
        session.get(Student.class, 1L);

byte[] imageData =
        student.getProfilePhoto();

session.close();
```

Now `imageData` contains the original binary data.

We can write it back to a file:

```java
Files.write(
        Paths.get("downloaded-profile.jpg"),
        imageData
);
```

---

# 8. Database Structure

Conceptually, our table might look like:

```sql
CREATE TABLE students (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255),
    biography LONGTEXT,
    profile_photo LONGBLOB
);
```

The exact SQL type depends on the database.

For MySQL, common large-object types include:

### Text

```text
TINYTEXT
TEXT
MEDIUMTEXT
LONGTEXT
```

### Binary

```text
TINYBLOB
BLOB
MEDIUMBLOB
LONGBLOB
```

Hibernate determines the appropriate mapping based on the Java type, `@Lob`, dialect, and schema-generation configuration.

---

# 9. `@Lob` Does Not Mean "File"

This is important.

`@Lob` means:

> This property contains large data that should be persisted using a LOB-capable database type.

For example:

```java
@Lob
private String content;
```

does **not** mean Hibernate creates a `.txt` file.

It stores the content in the database.

Similarly:

```java
@Lob
private byte[] image;
```

doesn't create an image file on your server.

It stores the binary data in the database.

---

# 10. `@Lob` vs Normal String

Without `@Lob`:

```java
private String description;
```

Usually maps to something like:

```text
VARCHAR
```

With:

```java
@Lob
private String description;
```

Hibernate treats it as a large character object.

Conceptually:

```text
Normal String
      ↓
VARCHAR

@Lob String
      ↓
CLOB / large text
```

---

# 11. `@Lob` vs `byte[]`

This is another important distinction.

```java
@Lob
private String document;
```

means:

```text
Large TEXT
   ↓
CLOB
```

Whereas:

```java
@Lob
private byte[] document;
```

means:

```text
Large BINARY DATA
   ↓
BLOB
```

### Easy trick

**String → CLOB**

**byte[] → BLOB**

---

# 12. What About `Blob` and `Clob`?

JPA also provides:

```java
java.sql.Blob
java.sql.Clob
```

For example:

```java
@Lob
private Blob document;
```

or:

```java
@Lob
private Clob content;
```

But for beginners, prefer:

```java
@Lob
private String content;
```

and:

```java
@Lob
private byte[] document;
```

because they are much simpler to work with.

---

# 13. Real-World Design Consideration

Although Hibernate allows you to store images, PDFs and other files directly in the database, **that doesn't always mean you should**.

For large files, applications commonly use:

```text
Application
     │
     ├── Database
     │     └── File metadata
     │
     └── Object/File Storage
           └── Actual PDF/Image/Video
```

For example:

```text
Database
---------
id = 101
fileName = resume.pdf
fileUrl = /files/resume/101
fileSize = 2.4 MB
```

while the actual PDF is stored in object storage or a file system.

For smaller files or when transactional storage is important, database LOBs can still be appropriate.

---

# 14. Quick Comparison

| Java     | Annotation | Database concept | Example       |
| -------- | ---------- | ---------------- | ------------- |
| `String` | —          | `VARCHAR`        | Name          |
| `String` | `@Lob`     | CLOB/LONGTEXT    | Large article |
| `byte[]` | `@Lob`     | BLOB/LONGBLOB    | Image/PDF     |
| `Clob`   | `@Lob`     | CLOB             | Large text    |
| `Blob`   | `@Lob`     | BLOB             | Binary file   |

### Remember this

```text
              @Lob
                │
        ┌───────┴───────┐
        ↓               ↓
      String           byte[]
        ↓               ↓
      CLOB             BLOB
        ↓               ↓
 Large text        Binary data
```

So, for a Hibernate beginner, the two most important patterns are:

```java
@Lob
private String content;
```

**Large text**

and

```java
@Lob
private byte[] file;
```

**Large binary data**.
