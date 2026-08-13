# 1. First: What problem does Maven solve?

Suppose you are writing a Java JDBC program.

You need MySQL Connector/J.

Without Maven, you might do this:

```text
Your Project
│
├── src/
│   └── Main.java
│
└── lib/
    └── mysql-connector-j-9.x.x.jar
```

Then you manually tell Java/IDE:

> "Use this JAR file because my program needs MySQL Connector."

This works.

But imagine your project needs:

* MySQL JDBC driver
* HikariCP
* SLF4J
* Logback
* Jackson
* JUnit
* Spring
* Spring Boot
* Hibernate

Now you have to manually download and manage many `.jar` files.

That's where **Maven** comes in.

---

# 2. So what exactly is Maven?

**Maven is a build and dependency management tool for Java.**

In simple words:

> **Maven manages the libraries your Java project needs and also manages the process of building your project.**

It can:

* download libraries
* download libraries' required libraries
* manage versions
* compile your code
* run tests
* package your application
* maintain a standard project structure
* create `.jar` / `.war` files
* run plugins for different tasks

Think of Maven as a **project manager for your Java project**.

---

# 3. What were we doing before Maven?

Suppose you learned JDBC.

You downloaded:

```text
mysql-connector-j-9.4.0.jar
```

and added it to your project.

Your program:

```java
Class.forName("com.mysql.cj.jdbc.Driver");
Connection con =
    DriverManager.getConnection(
        "jdbc:mysql://localhost:3306/test",
        "root",
        "password"
    );
```

Your Java code needs the MySQL JDBC driver.

So you manually gave Java the JAR.

### This is called manual dependency management.

```text
YOU
 │
 ├── Search for JAR
 │
 ├── Download JAR
 │
 ├── Put JAR in project
 │
 ├── Add JAR to classpath
 │
 └── Maintain its version
```

Maven automates most of this.

---

# 4. Maven vs normal external JAR

| Without Maven                           | With Maven                  |
| --------------------------------------- | --------------------------- |
| Download JAR manually                   | Maven downloads it          |
| Put JAR in project                      | Maven manages it            |
| Add JAR to classpath                    | Maven manages classpath     |
| Manually find dependencies              | Maven resolves dependencies |
| Manually manage versions                | `pom.xml` manages versions  |
| Manually find dependency's dependencies | Maven downloads them        |
| Harder to reproduce project             | Easier to reproduce project |

But remember:

> **The library itself has not disappeared.**

This is the important concept.

If you use Maven:

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.4.0</version>
</dependency>
```

Maven still ultimately gets the **JAR file**.

It is simply doing the downloading and management for you.

---

# 5. Then what is `pom.xml`?

`pom.xml` is basically the **instruction/configuration file for Maven**.

POM means:

> **Project Object Model**

For a beginner, think:

> `pom.xml` = "Maven's instruction book for my project."

For example:

```xml
<project>

    <groupId>com.example</groupId>
    <artifactId>StudentApp</artifactId>
    <version>1.0</version>

    <dependencies>

        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>9.4.0</version>
        </dependency>

    </dependencies>

</project>
```

You are basically telling Maven:

> "My project is StudentApp, and I need MySQL Connector/J version 9.4.0."

---

# 6. What are these three things?

```xml
<groupId>com.mysql</groupId>
<artifactId>mysql-connector-j</artifactId>
<version>9.4.0</version>
```

Together they identify a particular library version.

Think of it like an address.

```text
groupId
   ↓
Which organization?

artifactId
   ↓
Which library?

version
   ↓
Which version?
```

For example:

```text
com.mysql
    ↓
mysql-connector-j
    ↓
9.4.0
```

means:

> MySQL's Connector/J library, version 9.4.0.

---

# 7. Where does Maven get this JAR from?

This is where the **Maven repository** comes in.

There are three concepts :

```text
Local Repository
       ↑
       │
       ↓
Maven Central / Remote Repository
```

Let's understand them one by one.

---

# 8. Maven Central — the cloud

Imagine you write:

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.4.0</version>
</dependency>
```

Maven needs to find this library.

It can download it from a remote repository such as **Maven Central**.

Conceptually:

```text
                INTERNET
                   │
                   ▼
          Maven Central
                   │
                   │ download
                   ▼
              Your PC
```

So Maven Central is essentially a huge online repository containing Java artifacts.

You don't normally visit it and manually download the JAR.

Maven does it.

---

# 9. What is the local repository?

This is where the concept becomes really interesting.

Suppose you run your project for the first time.

Maven says:

> "I need MySQL Connector/J."

It checks your computer first.

Conceptually:

```text
Your project
     │
     ▼
Local Maven Repository
     │
     ├── Found? ── YES ──> Use it
     │
     └── NO
          │
          ▼
      Internet
          │
          ▼
   Maven Central
          │
          ▼
    Download JAR
          │
          ▼
 Local Repository
```

On Windows, the local Maven repository is normally under:

```text
C:\Users\<username>\.m2\repository
```

For example:

```text
.m2
└── repository
    └── com
        └── mysql
            └── mysql-connector-j
                └── 9.4.0
                    └── mysql-connector-j-9.4.0.jar
```

So after Maven downloads it once, it can reuse it.

---

# 10. Is `.m2` cloud?

**No.**

This is extremely important.

```text
Maven Central
     ↓
Cloud / Internet
```

but

```text
.m2/repository
     ↓
Your computer's local storage
```

So:

### Maven Central

Online.

### `.m2/repository`

On your computer.

---

# 11. What happens the first time?

Suppose you create:

```text
StudentApp
```

and add:

```xml
mysql-connector-j
```

You run:

```bash
mvn compile
```

Maven roughly does:

```text
Read pom.xml
      ↓
Find dependency
      ↓
Check local .m2 repository
      ↓
Is MySQL driver present?
      ↓
       NO
       ↓
Go to remote repository
       ↓
Download MySQL driver
       ↓
Store it in .m2
       ↓
Add it to project classpath
       ↓
Compile your code
```

---

# 12. What happens the second time?

You run:

```bash
mvn compile
```

again.

Maven checks:

```text
.m2/repository
```

and finds:

```text
mysql-connector-j-9.4.0.jar
```

So it doesn't need to download it again.

Conceptually:

```text
pom.xml
   ↓
Need MySQL
   ↓
Check .m2
   ↓
FOUND ✅
   ↓
Use existing JAR
```

---

# 13. Now the BIG difference from your previous JDBC method

Previously you might have done:

```text
Download mysql-connector.jar
       ↓
Copy into lib/
       ↓
Add JAR to project
```

With Maven:

```text
pom.xml
   ↓
Declare dependency
   ↓
Maven finds it
   ↓
Downloads it
   ↓
Stores it locally
   ↓
Makes it available to project
```

---

# 14. What if we don't add ANY dependency?

This is an excellent question.

Suppose your Maven project contains:

```java
public class Main {

    public static void main(String[] args) {

        System.out.println("Hello");

    }
}
```

You don't need an external library.

Why?

Because `System.out.println()` belongs to Java's standard library.

You don't need to add:

```xml
<dependency>
    ...
</dependency>
```

for basic Java classes.

---

# 15. Java itself already comes with libraries

When you install a JDK, you already get the Java platform libraries.

For example:

```java
String
ArrayList
HashMap
Scanner
File
Thread
Exception
System
```

These are provided by Java.

Conceptually:

```text
JDK
│
├── Java Compiler
├── JVM
└── Java Platform Libraries
```

So Maven isn't needed for every single class.

---

# 16. When do I need a Maven dependency?

When your application needs a library that isn't part of the Java platform.

For example:

### JDBC MySQL

```text
MySQL Connector/J
```

### JSON

```text
Jackson
```

### Connection Pool

```text
HikariCP
```

### Logging

```text
SLF4J
Logback
```

### Web application

```text
Spring
Spring Boot
```

Then Maven can manage those dependencies.

---

# 17. But why does Maven sometimes download MANY JARs when I only asked for ONE?

This is one of Maven's most important features.

Suppose you write:

```xml
<dependency>
    ...
</dependency>
```

for Library A.

But Library A internally needs:

```text
Library B
Library C
Library D
```

Maven understands this.

This is called **transitive dependency management**.

Conceptually:

```text
Your Application
       │
       ▼
   Library A
    /  |  \
   ▼   ▼   ▼
  B    C    D
       │
       ▼
       E
```

You asked for:

```text
A
```

Maven discovers:

```text
A → B
A → C
A → D
C → E
```

and downloads the required dependencies.

Without Maven, you might have to manually figure all of that out.

---

# 18. This is where Maven becomes REALLY useful

Imagine Spring Boot.

You write:

```xml
<dependency>
    ...
    spring-boot-starter-web
</dependency>
```

You didn't manually download:

```text
Spring MVC
Jackson
Tomcat
Spring Core
Spring Beans
Logging libraries
...
```

Maven resolves the dependency graph.

Conceptually:

```text
spring-boot-starter-web
          │
          ├── Spring Core
          ├── Spring MVC
          ├── Jackson
          ├── Embedded Tomcat
          ├── Logging
          └── other dependencies
```

You ask for one starter.

Maven figures out the rest.

---

# 19. What happens behind the scenes?

Let's go deeper.

You execute:

```bash
mvn compile
```

Maven starts.

### Step 1 — Maven reads `pom.xml`

It looks at:

```xml
<dependencies>
   ...
</dependencies>
```

and understands what your project needs.

---

### Step 2 — Maven creates a dependency graph

Suppose:

```text
Your Project
     │
     ├── MySQL Driver
     │
     └── HikariCP
              │
              └── SLF4J
```

Maven understands these relationships.

---

### Step 3 — Maven checks local repository

It looks inside:

```text
.m2/repository
```

Does the required artifact exist?

If yes:

```text
Use it
```

If no:

```text
Download it
```

---

### Step 4 — Maven contacts remote repositories

Usually Maven Central or another configured repository.

It finds the artifact based on:

```text
groupId
artifactId
version
```

---

### Step 5 — Maven downloads the files

Not just necessarily the JAR.

It may download metadata such as:

```text
.jar
.pom
.sha
metadata
```

The `.pom` of a dependency is particularly important because it can describe that dependency's own dependencies.

---

### Step 6 — Maven stores them locally

Something like:

```text
C:\Users\You\.m2\repository\
```

---

### Step 7 — Maven builds the classpath

Now Maven effectively tells Java:

> "When compiling this project, these JARs are available."

Conceptually:

```text
javac
 │
 ├── Your .java files
 │
 ├── MySQL JAR
 │
 ├── HikariCP JAR
 │
 └── SLF4J JAR
```

---

### Step 8 — Maven compiles

Your:

```text
.java
```

becomes:

```text
.class
```

---

# 20. Where does `target` come from?

When you run Maven commands, you may see:

```text
target/
```

For example:

```text
StudentApp
│
├── src
│   ├── main
│   │   └── java
│   │       └── Main.java
│   │
│   └── test
│
├── pom.xml
│
└── target
    ├── classes
    │   └── Main.class
    │
    └── ...
```

`target` is Maven's **build output directory**.

For example:

```text
src/main/java/Main.java
             │
             │ Maven compile
             ▼
target/classes/Main.class
```

---

# 21. So understand these three locations

```text
              YOUR COMPUTER
┌─────────────────────────────────────┐
│                                     │
│  Project                            │
│  ├── src/                           │
│  ├── pom.xml                        │
│  └── target/                        │
│                                     │
│  Local Maven Repository             │
│  └── .m2/repository/                │
│                                     │
└─────────────────────────────────────┘
                 │
                 │ Internet
                 ▼
          Maven Central
```

### `src`

Your source code.

### `pom.xml`

Instructions/dependencies for Maven.

### `target`

Build output.

### `.m2/repository`

Downloaded dependency cache.

### Maven Central

Online repository containing artifacts.

---

# 22. What if the internet is OFF?

Suppose you already downloaded:

```text
mysql-connector-j
```

and it exists inside:

```text
.m2/repository
```

Then Maven may be able to build the project **without internet**, because the dependency is already cached locally.

But if you need a dependency that isn't in `.m2`:

```text
Internet OFF
       ↓
Maven can't download it
       ↓
Build may fail
```

---

# 23. What if I delete `.m2`?

Suppose you delete:

```text
C:\Users\You\.m2
```

Your project itself isn't necessarily deleted.

Your:

```text
src/
pom.xml
```

are still there.

But Maven has lost its local dependency cache.

Next time you run the build:

```text
pom.xml
   ↓
Check .m2
   ↓
Dependency missing
   ↓
Internet
   ↓
Download again
```

---

# 24. Maven is NOT the same as Maven Central

Think:

```text
Maven
   =
Tool
```

while:

```text
Maven Central
   =
Online repository
```

And:

```text
.m2/repository
   =
Local repository/cache
```

So:

```text
Maven
  │
  ├── reads pom.xml
  │
  ├── checks local repository
  │
  ├── contacts remote repository
  │
  ├── downloads dependencies
  │
  └── builds project
```

---

# 25. Where should you use Maven and where not?

### Small beginner Java program

You don't necessarily need Maven.

```text
Hello World
Calculator
Array programs
OOP practice
```

Normal Java is enough.

---

### Learning JDBC manually

Initially, using a JAR manually is actually useful pedagogically.

Learn:

```text
What is a JAR?
What is a driver?
What is classpath?
What is an external library?
```

For example:

```text
mysql-connector-j.jar
```

---

### Real Java project

Maven becomes much more useful.

Especially when you have:

```text
JDBC
+
HikariCP
+
Logging
+
JUnit
+
JSON
```

---

### Spring Boot

Maven/Gradle becomes almost essential from a practical teaching perspective.

Because Spring Boot projects have many dependencies.

Instead of:

```text
Download 30 JARs
Find compatible versions
Add them
Check conflicts
Repeat...
```

you can declare dependencies in:

```text
pom.xml
```

and Maven handles the dependency graph.

---

# 26. One very important misconception

> "Maven downloads libraries from the internet every time I run my program."

**No.**

Usually:

```text
First time

pom.xml
   ↓
.m2?
   ↓
NO
   ↓
Internet
   ↓
Download
   ↓
.m2
```

Then:

```text
Next time

pom.xml
   ↓
.m2?
   ↓
YES
   ↓
Use cached dependency
```

---

# 27. And Maven does NOT run your Java program

This distinction is also important.

Maven is a **build tool**.

It can invoke Java-related tools, plugins, tests, packaging, etc., but Maven itself isn't your application's runtime.

Think:

```text
Maven
   ↓
Builds your application
   ↓
Creates application artifact
   ↓
Java/JVM
   ↓
Runs your application
```

---

# 28. The simplest mental model

I would teach Maven using this analogy:

### Without Maven

You go shopping yourself.

```text
"I need MySQL."

Search Google
     ↓
Download JAR
     ↓
Put it in lib
     ↓
Configure classpath
```

### With Maven

You give a shopping list to Maven.

```text
pom.xml

I need:
MySQL Connector 9.4.0
HikariCP ...
JUnit ...
```

Maven says:

> "Okay, I'll find them, download them, find what they depend on, store them, and make them available to the project."

That is the core idea.

---

# 29. Final picture

```text
                    YOUR PROJECT
                         │
                         │
                    pom.xml
                         │
                         ▼
                      MAVEN
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
       Local Repository       Remote Repository
          (.m2)              (Maven Central)
              │                     │
              │                     │
        Already exists?       Download if needed
              │                     │
              └──────────┬──────────┘
                         │
                         ▼
                  Dependency JARs
                         │
                         ▼
                    Build Project
                         │
                         ▼
                       target/
                         │
                         ▼
                   .class / .jar
```

### In one sentence:

> **Maven doesn't replace libraries like JDBC drivers; Maven is the tool that finds, downloads, manages, connects, and builds with those libraries for your Java project.**

```text
Normal Java
    ↓
External JAR manually
    ↓
Understand classpath
    ↓
Maven
    ↓
pom.xml
    ↓
Dependency
    ↓
Maven Central
    ↓
.m2 local repository
    ↓
Transitive dependencies
    ↓
Build lifecycle
    ↓
Spring Boot
```

# Lifecycle

A **lifecycle** is a sequence of phases that Maven executes to build your project.

```text
validate
    ↓
compile
    ↓
test
    ↓
package
    ↓
verify
    ↓
install
    ↓
deploy
```

---

## clean

```text
clean
```

Deletes everything generated by previous builds.

For example:

Before:

```text
Helloji
├── pom.xml
├── src
└── target
    ├── classes
    ├── test-classes
    └── Helloji-1.0-SNAPSHOT.jar
```

After running **clean**:

```text
Helloji
├── pom.xml
└── src
```

The `target` folder is deleted.

---

## validate

```text
validate
```

Checks whether the project is configured correctly.

Maven checks:

* Is `pom.xml` valid?
* Is the project structure correct?
* Are there any obvious configuration errors?

It **doesn't compile your code**.

---

## compile

```text
compile
```

Compiles your `.java` files into `.class` files.

```text
Main.java
      ↓
compile
      ↓
Main.class
```

The generated files are stored here:

```text
target/classes
```

---

## test

```text
test
```

Runs unit tests.

For example:

```text
src
├── main
│   └── java
└── test
    └── java
        └── MainTest.java
```

Maven executes everything inside:

```text
src/test/java
```

If you don't have tests, this phase does almost nothing.

---

## package

```text
package
```

Packages the application.

```text
.class files
       ↓
package
       ↓
.jar file
```

Example:

```text
target/Helloji-1.0-SNAPSHOT.jar
```

---

## verify

```text
verify
```

Performs additional checks after packaging.

Examples:

* Is the JAR valid?
* Did all tests pass?

---

## install

```text
install
```

Copies the packaged JAR into your **local Maven repository**.

```text
Helloji-1.0-SNAPSHOT.jar
                ↓
C:\Users\YourName\.m2\repository
```

The `.m2` folder is Maven's local library storage.

---

## deploy

```text
deploy
```

Uploads your project to a **remote Maven repository**.

```text
Your computer
        ↓
Remote repository
```

Examples:

* Nexus
* Artifactory

Beginners almost never use this.

---

## site

```text
site
```

Generates project documentation.

```text
Project
     ↓
site
     ↓
HTML documentation
```

Most beginners don't use this either.

---

# Plugins

Plugins are the tools that perform the actual work.

Think of the lifecycle as the **manager** and plugins as the **workers**.

---

## maven-clean-plugin

```text
maven-clean-plugin
```

Used by:

```text
clean
```

Its job:

```text
Delete the target folder.
```

---

## maven-compiler-plugin

```text
maven-compiler-plugin
```

Used by:

```text
compile
```

Its job:

```text
.java
   ↓
.class
```

---

## maven-deploy-plugin

```text
maven-deploy-plugin
```

Used by:

```text
deploy
```

Its job:

```text
Upload artifacts to a remote repository.
```

---

## maven-install-plugin

```text
maven-install-plugin
```

Used by:

```text
install
```

Its job:

```text
Copy the JAR to the .m2 repository.
```

---

## maven-jar-plugin

```text
maven-jar-plugin
```

Used by:

```text
package
```

Its job:

```text
.class files
       ↓
.jar file
```

---

## maven-resources-plugin

```text
maven-resources-plugin
```

Copies resources into the build folder.

Examples:

```text
src
└── main
    ├── java
    └── resources
        ├── application.properties
        └── db.properties
```

These files are copied into:

```text
target/classes
```

---

## maven-site-plugin

```text
maven-site-plugin
```

Generates project documentation.

---

## maven-surefire-plugin

```text
maven-surefire-plugin
```

Runs your tests.

```text
MainTest.java
       ↓
surefire
       ↓
Test results
```

---

# Dependencies

```text
Dependencies
```

This section shows all external libraries used by your project.

Right now, it's empty.

If you add the MySQL dependency:

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.4.0</version>
</dependency>
```

You'll see:

```text
Dependencies
    ↓
mysql-connector-j
    ↓
mysql-connector-j-9.4.0.jar
```

---

# You'll usually use only these three:

```text
compile ✅

package ✅

Dependencies ✅
```

You will rarely click:

```text
validate ❌

verify ❌

install ❌

deploy ❌

site ❌
```
