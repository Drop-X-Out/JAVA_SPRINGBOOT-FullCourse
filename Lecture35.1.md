# JUnit Tutorial for Beginners: Writing and Running Test Cases Using Maven

We will use **Java 17, Maven, JUnit 5 (JUnit Jupiter), and the existing Maven WAR project**.

---

## Table of Contents

1. What is software testing?
2. What is a test case?
3. What is JUnit?
4. Why do we need JUnit?
5. Understanding unit testing with a real example
6. Setting up JUnit in Maven
7. Understanding the project structure
8. Creating a Calculator class
9. Writing the first JUnit test case
10. Understanding JUnit annotations and assertions
11. Running test cases using Maven
12. Understanding test results
13. Testing multiple scenarios
14. Testing exceptions
15. Understanding the Maven test lifecycle
16. Common errors and solutions
17. Best practices
18. Practice exercises
19. Interview questions

---

# 1. What Is Software Testing?

Software testing is the process of checking whether a program behaves as expected.

Suppose we write a Java method that adds two numbers.

```java
public int add(int a, int b) {
    return a + b;
}
```

If we pass:

```text
a = 10
b = 20
```

The expected result is:

```text
30
```

But how do we know that our code works correctly for different inputs?

We could run the program manually and check the output. However, manually repeating this process for hundreds of inputs is time-consuming.

This is where automated testing helps.

We write test cases that check the expected behavior, and a testing framework executes those checks for us.

For example:

```text
Input: 10, 20
Expected output: 30
Actual output:   30

Result: PASS
```

If the actual output differs from the expected output, the test fails.

---

# 2. What Is a Test Case?

A test case is a set of inputs, actions, and expected results used to verify a particular behavior of software.

Consider a calculator's addition feature.

| Test case | Input | Expected result |
|---|---|---|
| Test 1 | 2 + 3 | 5 |
| Test 2 | 10 + 20 | 30 |
| Test 3 | -5 + 5 | 0 |
| Test 4 | 0 + 0 | 0 |

Each row represents a different scenario we want to verify.

A test case generally involves three steps:

1. **Arrange:** Prepare the inputs and objects.
2. **Act:** Execute the method being tested.
3. **Assert:** Compare the actual result with the expected result.

This pattern is commonly called **Arrange–Act–Assert (AAA)**.

Example:

```java
// Arrange
int a = 10;
int b = 20;

// Act
int result = a + b;

// Assert
assertEquals(30, result);
```

JUnit provides methods such as `assertEquals()` to perform these checks automatically.

---

# 3. What Is JUnit?

JUnit is a testing framework for Java that helps developers write and execute automated tests.

With JUnit, we can test:

- Mathematical calculations.
- Business logic.
- String operations.
- Validation methods.
- Exception handling.
- Other Java methods and classes.

JUnit provides annotations and assertion methods that make testing easier.

For this tutorial, we will use **JUnit 5**, specifically its JUnit Jupiter programming model.

For example:

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

class CalculatorTest {

    @Test
    void testAddition() {

        int result = 10 + 20;

        assertEquals(30, result);
    }
}
```

Here:

- `@Test` identifies a test method.
- `assertEquals()` checks whether the actual value equals the expected value.
- JUnit reports whether the test passes or fails.

We will create and execute this test properly through Maven.

---

# 4. Why Do We Need JUnit?

Imagine that you are developing a student management application.

Your application contains methods for:

- Calculating student marks.
- Calculating percentages.
- Checking whether a student passed.
- Validating student registration.
- Calculating attendance percentages.

Suppose you change the percentage calculation method.

How can you ensure that your change has not broken existing functionality?

You could manually test every possible scenario, but that becomes difficult as the application grows.

JUnit allows you to save these checks as automated test cases.

Whenever you change your code, you can run the tests again.

## Manual testing vs automated testing

| Manual testing | Automated testing using JUnit |
|---|---|
| A person performs the checks | A testing framework executes the checks |
| Repeated checks take time | Tests can be rerun quickly |
| Results may depend on human observation | Results are reported consistently |
| Difficult to repeat many scenarios manually | Many scenarios can be automated |
| Useful for exploratory and user-interface testing | Useful for repeatable Java code checks |

JUnit does not eliminate manual testing. It complements it.

---

# 5. Understanding Unit Testing with a Real Example

Suppose we are developing a student management system.

We need to calculate a student's percentage.

The formula is:

\[
\text{Percentage} =
\frac{\text{Obtained Marks}}{\text{Maximum Marks}}
\times 100
\]

For example:

- Obtained marks: 80
- Maximum marks: 100

The percentage should be 80%.

We can implement this logic in a Java class and write tests for it.

Our workflow will be:

```text
Calculator.java
      |
      | Contains application logic
      v
CalculatorTest.java
      |
      | Checks expected behavior
      v
JUnit
      |
      | Executes assertions
      v
Maven
      |
      | Runs the test lifecycle
      v
Test report
```

Notice that `CalculatorTest.java` is separate from `Calculator.java`.

This separation helps keep production code independent from test code.

---

# 6. Setting Up JUnit in Maven

We will use our existing Maven WAR project.

Its root directory contains:

```text
pom.xml
```

Maven manages JUnit through a dependency declared in that file.

## Step 1: Add the JUnit dependency

Inside the existing `<dependencies>` element of `pom.xml`, add:

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.2</version>
    <scope>test</scope>
</dependency>
```

The `test` scope means that JUnit is needed for testing, not for running the application in production.

Maven will resolve JUnit and its required dependencies.

## Step 2: Configure Maven Surefire

Maven needs a test runner to discover and execute JUnit tests during the `test` phase.

Add this plugin inside the existing `<build><plugins>` element:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.2.5</version>
</plugin>
```

**What is Surefire?**

Maven Surefire is a Maven plugin that executes tests during the test phase. It supports JUnit and other testing frameworks.

JUnit defines and runs the tests through its engine; Surefire integrates test execution into the Maven build.

---

# 7. Complete `pom.xml`

The following is the complete configuration for our earlier `hello-maven` WAR project, now with JUnit added.

Replace the existing `pom.xml` with this version if you are following the previous tutorial.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>hello-maven</artifactId>
    <version>1.0.0</version>
    <packaging>war</packaging>

    <name>Hello Maven Web Application</name>

    <properties>
        <maven.compiler.release>17</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>

        <!-- Servlet API supplied by Tomcat at runtime -->
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>6.0.0</version>
            <scope>provided</scope>
        </dependency>

        <!-- JUnit 5 for automated tests -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>

        <finalName>hello-maven</finalName>

        <plugins>

            <!-- Compile Java source code -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
            </plugin>

            <!-- Run automated tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
            </plugin>

            <!-- Package the web application as a WAR -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.4.0</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>

        </plugins>

    </build>

</project>
```

### What changed?

We added two important things:

1. The JUnit Jupiter dependency.
2. The Maven Surefire plugin.

The project is still a WAR project. Adding JUnit does not change its packaging type.

---

# 8. Understanding the Project Structure

Our project will now have this structure:

```text
hello-maven/
│
├── pom.xml
│
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/
    │   │       └── example/
    │   │           └── Calculator.java
    │   │
    │   └── webapp/
    │       └── index.html
    │
    └── test/
        └── java/
            └── com/
                └── example/
                    └── CalculatorTest.java
```

Two directories are particularly important.

### `src/main/java`

This contains production code — the code used by the actual application.

Example:

```text
src/main/java/com/example/Calculator.java
```

### `src/test/java`

This contains automated test code.

Example:

```text
src/test/java/com/example/CalculatorTest.java
```

**Remember:** Do not put `CalculatorTest.java` inside `src/main/java`. Maven uses separate directories for application code and test code.

---

# 9. Creating the Calculator Class

First, we need a class to test.

Create this file:

`src/main/java/com/example/Calculator.java`

```java
package com.example;

public class Calculator {

    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }

    public int multiply(int a, int b) {
        return a * b;
    }

    public int divide(int a, int b) {

        if (b == 0) {
            throw new IllegalArgumentException(
                    "Divisor cannot be zero"
            );
        }

        return a / b;
    }
}
```

## Understanding the code

### Addition

```java
public int add(int a, int b) {
    return a + b;
}
```

If we call:

```java
add(10, 20);
```

The result is:

```text
30
```

### Subtraction

```java
public int subtract(int a, int b) {
    return a - b;
}
```

Calling `subtract(20, 10)` returns `10`.

### Multiplication

```java
public int multiply(int a, int b) {
    return a * b;
}
```

Calling `multiply(5, 4)` returns `20`.

### Division

```java
public int divide(int a, int b)
```

The method checks whether the divisor is zero.

```java
if (b == 0) {
    throw new IllegalArgumentException(
            "Divisor cannot be zero"
    );
}
```

If the divisor is zero, the method throws an exception instead of performing an invalid division.

We will write a JUnit test to verify this behavior.

---

# 10. Writing the First JUnit Test Case

Create the following file:

`src/test/java/com/example/CalculatorTest.java`

```java
package com.example;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

class CalculatorTest {

    @Test
    void testAddition() {

        // Arrange
        Calculator calculator = new Calculator();

        // Act
        int actualResult = calculator.add(10, 20);

        // Assert
        assertEquals(30, actualResult);
    }
}
```

This is our first automated test.

## Understanding the code line by line

### Importing `@Test`

```java
import org.junit.jupiter.api.Test;
```

This imports the JUnit annotation used to mark test methods.

### Importing `assertEquals`

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
```

This lets us call `assertEquals()` directly without writing the full class name.

### Creating the test class

```java
class CalculatorTest {
```

The class contains our test methods.

The name `CalculatorTest` follows a common naming convention that makes the purpose of the class clear.

### Using `@Test`

```java
@Test
void testAddition() {
```

The `@Test` annotation identifies the method as a JUnit test.

JUnit discovers and executes it when the test runner runs.

### Arrange

```java
Calculator calculator = new Calculator();
```

We create the object that we want to test.

### Act

```java
int actualResult = calculator.add(10, 20);
```

We call the method and store its result.

The actual result is `30`.

### Assert

```java
assertEquals(30, actualResult);
```

The first argument is the expected result, and the second is the actual result.

If the values are equal, the assertion passes.

If they are different, JUnit marks the test as failed and reports the mismatch.

---

# 11. Running Test Cases Using Maven

Now comes the most important part: executing our test cases.

## Step 1: Open a terminal

Navigate to the directory containing `pom.xml`.

```bash
cd hello-maven
```

Use the actual project path on your computer.

## Step 2: Run the tests

Execute:

```bash
mvn test
```

Maven will:

1. Read the project configuration.
2. Resolve required dependencies and plugins.
3. Compile the main Java source code.
4. Compile the test source code.
5. Execute discovered tests using Surefire.
6. Display the test results.

A successful run should end with output similar to:

```text
[INFO] Running com.example.CalculatorTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] BUILD SUCCESS
```

Exact output may differ depending on your Maven and plugin versions.

### What does `BUILD SUCCESS` mean?

It means the Maven build completed successfully.

In this context, if the test phase ran and the tests were discovered, it indicates that no test failures or errors caused the build to fail.

**Important:** `BUILD SUCCESS` does not prove that the entire application is free of bugs. It only tells us that the build and the checks that ran were successful.

---

# 12. Understanding Test Results

JUnit and Maven report several important values.

| Term | Meaning |
|---|---|
| Tests run | Number of tests executed |
| Failures | Tests whose assertions failed |
| Errors | Tests that encountered unexpected errors or exceptions |
| Skipped | Tests that were not executed |
| Build success | Maven completed the build successfully |

Suppose we see:

```text
Tests run: 4, Failures: 1, Errors: 0, Skipped: 0
```

This means:

- Four tests were executed.
- One test failed an assertion.
- No test encountered an unexpected error.
- No tests were skipped.

The Maven build will normally fail when a test failure or error occurs.

## Test report location

Maven Surefire typically generates reports inside:

```text
target/surefire-reports/
```

For our project, you may see files such as:

```text
target/
└── surefire-reports/
    ├── com.example.CalculatorTest.txt
    └── TEST-com.example.CalculatorTest.xml
```

The text report is useful for reading test results. The XML report is useful for build tools and continuous integration systems.

---

# 13. Testing Multiple Scenarios

Testing only one input is not enough.

We should check all the important operations and multiple input combinations.

Replace `CalculatorTest.java` with the following complete version.

```java
package com.example;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

class CalculatorTest {

    @Test
    void testAddition() {

        Calculator calculator = new Calculator();

        int actualResult = calculator.add(10, 20);

        assertEquals(30, actualResult);
    }

    @Test
    void testSubtraction() {

        Calculator calculator = new Calculator();

        int actualResult = calculator.subtract(20, 10);

        assertEquals(10, actualResult);
    }

    @Test
    void testMultiplication() {

        Calculator calculator = new Calculator();

        int actualResult = calculator.multiply(5, 4);

        assertEquals(20, actualResult);
    }

    @Test
    void testDivision() {

        Calculator calculator = new Calculator();

        int actualResult = calculator.divide(20, 5);

        assertEquals(4, actualResult);
    }

    @Test
    void testAdditionWithNegativeNumbers() {

        Calculator calculator = new Calculator();

        int actualResult = calculator.add(-5, 5);

        assertEquals(0, actualResult);
    }
}
```

Run:

```bash
mvn test
```

Expected summary:

```text
Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

All five test methods should pass if the code matches the example.

Notice that every test method checks one behavior. This makes failures easier to understand.

---

# 14. Testing Exceptions

Testing successful results is only one part of software testing.

We should also check how our application handles invalid inputs.

Our `divide()` method throws an `IllegalArgumentException` when the divisor is zero.

We can verify this behavior using JUnit's `assertThrows()`.

Add the following test to `CalculatorTest.java`:

```java
@Test
void testDivisionByZero() {

    Calculator calculator = new Calculator();

    assertThrows(
            IllegalArgumentException.class,
            () -> calculator.divide(10, 0)
    );
}
```

Also add this import:

```java
import static org.junit.jupiter.api.Assertions.assertThrows;
```

## Understanding `assertThrows()`

```java
assertThrows(
        IllegalArgumentException.class,
        () -> calculator.divide(10, 0)
);
```

It checks that the provided operation throws the expected exception type.

The lambda expression:

```java
() -> calculator.divide(10, 0)
```

represents the operation being tested.

If the expected exception is thrown, the assertion passes.

If no exception is thrown, the test fails.

If a different exception type is thrown, the assertion also fails.

Run the tests again:

```bash
mvn test
```

With the earlier five tests and this new test, the expected count is six tests.

---

# 15. Understanding the Maven Test Lifecycle

Maven has several build phases.

The test-related phases are particularly important.

| Command or phase | What happens |
|---|---|
| `mvn compile` | Compiles main source code; does not normally run tests |
| `mvn test-compile` | Compiles main and test source code |
| `mvn test` | Compiles the code and runs tests |
| `mvn package` | Runs the preceding lifecycle phases and packages the application |
| `mvn clean test` | Removes old build output and runs tests again |
| `mvn clean package` | Removes old build output, runs tests, and creates the WAR if successful |

For our project, the workflow is:

```text
mvn clean package
        |
        v
Clean previous output
        |
        v
Compile production code
        |
        v
Compile test code
        |
        v
Run JUnit tests
        |
        v
Package WAR
        |
        v
target/hello-maven.war
```

If the tests fail, Maven normally stops before completing the package phase.

## Does running JUnit deploy the WAR?

No.

These are separate tasks:

- `mvn test` runs automated tests.
- `mvn package` packages the application after the earlier lifecycle phases succeed.
- Copying the WAR into Tomcat's `webapps` directory is a deployment step.

You can use this command to test and package the application:

```bash
mvn clean package
```

After a successful build, you can deploy the resulting WAR to Tomcat.

---

# 16. Common Errors and Solutions

## Error 1: No tests are executed

You might see:

```text
Tests run: 0
```

Possible causes:

- The test class is in the wrong directory.
- The test method is missing `@Test`.
- The test class or method does not follow discoverable naming conventions.
- The JUnit dependency or Surefire configuration is incorrect.

Check the test location:

```text
src/test/java/com/example/CalculatorTest.java
```

Check that the test uses JUnit Jupiter:

```java
import org.junit.jupiter.api.Test;
```

And confirm that the Maven Surefire plugin is configured as shown earlier.

## Error 2: `package org.junit.jupiter.api does not exist`

Possible cause: Maven cannot resolve the JUnit dependency, or the dependency is missing from `pom.xml`.

Check the JUnit dependency, internet connectivity to your configured Maven repository, and the Maven build output.

Then run:

```bash
mvn clean test
```

## Error 3: Test failure

Suppose the test contains:

```java
assertEquals(30, calculator.add(10, 20));
```

But the implementation mistakenly contains:

```java
public int add(int a, int b) {
    return a - b;
}
```

The actual result will be `-10`, so the test fails.

Maven may display a message similar to:

```text
expected: <30> but was: <-10>
```

This is useful because the test has identified a problem in the implementation.

Fix the method:

```java
public int add(int a, int b) {
    return a + b;
}
```

Then rerun:

```bash
mvn test
```

## Error 4: JUnit 4 imports used with JUnit 5

JUnit 5 uses:

```java
import org.junit.jupiter.api.Test;
```

JUnit 4 commonly uses:

```java
import org.junit.Test;
```

Do not mix APIs and dependencies without configuring the required compatibility support.

For this tutorial, consistently use JUnit Jupiter.

## Error 5: `mvn test` succeeds, but the Servlet is broken

A unit test for `Calculator` does not test whether Tomcat can deploy the WAR or whether a browser can access the Servlet.

Those require additional integration or deployment tests.

For example, verifying that `/hello-maven/hello` returns an HTTP 200 response requires an HTTP-level test or a suitable integration-testing setup.

---

# 17. Best Practices for Writing JUnit Tests

1. **Test one behavior at a time.** Small tests are easier to debug.
2. **Use meaningful test names.** For example, `testDivisionByZero()` is clearer than `test1()`.
3. **Test edge cases.** Include zero, negative values, empty strings, and invalid inputs where relevant.
4. **Avoid depending on test execution order.** Each test should be able to run independently.
5. **Keep test code in `src/test/java`.** Do not mix test code with production code.
6. **Run tests before packaging.** Use `mvn clean package` to execute the lifecycle and package only if it succeeds.
7. **Test expected exceptions.** Use `assertThrows()` to verify invalid-input handling.
8. **Do not test only the happy path.** Check both expected and unexpected input conditions.

---

# 18. Practice Exercises

Complete these exercises using the same Maven project.

### Exercise 1: Add a square method

Add the following method to `Calculator.java`:

```java
public int square(int number) {
    return number * number;
}
```

Write a JUnit test that checks whether `square(5)` returns `25`.

### Exercise 2: Test negative multiplication

Write a test verifying that:

```text
multiply(-3, 4) = -12
```

### Exercise 3: Test division by zero

Use `assertThrows()` to verify that dividing by zero throws `IllegalArgumentException`.

### Exercise 4: Intentionally introduce a bug

Change the addition implementation from `a + b` to `a - b`.

Run:

```bash
mvn test
```

Observe the failure report.

Restore the correct implementation and rerun the tests.

### Exercise 5: Generate the WAR after testing

Run:

```bash
mvn clean package
```

Confirm that:

```text
target/hello-maven.war
```

is generated after all tests pass.

### Exercise 6: Create a student percentage calculator

Create a class named `StudentResult` with a method that calculates the percentage from obtained marks and maximum marks.

Write tests for:

- 80 out of 100.
- 45 out of 50.
- Zero obtained marks.
- Invalid maximum marks.

Decide what exception or validation behavior should apply when the maximum marks are zero.

---

# 19. Interview Questions

**Q1. What is JUnit?**

JUnit is a Java testing framework used to write and execute automated tests.

**Q2. What is unit testing?**

Unit testing verifies individual units of code, such as methods or classes, in isolation where practical.

**Q3. What is the purpose of `@Test`?**

It marks a method as a test method for the JUnit framework.

**Q4. What is an assertion?**

An assertion checks whether an actual result satisfies an expected condition.

**Q5. What does `assertEquals()` do?**

It verifies that the expected and actual values are equal according to the assertion's equality rules.

**Q6. What is Maven Surefire?**

It is a Maven plugin that executes tests during the test phase and produces test reports.

**Q7. What is the difference between `mvn test` and `mvn package`?**

`mvn test` runs the test lifecycle. `mvn package` runs the preceding lifecycle phases, including tests, before packaging the application.

**Q8. What is the purpose of `src/test/java`?**

It stores test source code separately from the main application code.

**Q9. What is the difference between a failure and an error?**

A failure typically means an assertion did not pass. An error generally indicates an unexpected problem during test execution.

**Q10. Why do we use `assertThrows()`?**

It verifies that a particular operation throws an expected exception type.

**Q11. Does JUnit test the application automatically without test methods?**

No. Developers need to write appropriate test methods, and the test runner must discover and execute them.

**Q12. Does a successful unit test guarantee that the whole application works?**

No. Unit tests cover the behavior they check. Integration tests, deployment tests, and other forms of testing may also be necessary.

---

# Final Summary

You have learned how to use JUnit with Maven to test Java code automatically.

The complete workflow is:

```text
Write Java class
       |
       v
Write JUnit test class
       |
       v
Configure JUnit in pom.xml
       |
       v
Run mvn test
       |
       v
Read test results
       |
       v
Fix any failures
       |
       v
Run mvn clean package
       |
       v
Generate the WAR file
       |
       v
Deploy to Tomcat
```

Remember the responsibility of each component:

- **JUnit:** Defines and executes tests through its testing framework.
- **Maven Surefire:** Integrates test execution into the Maven build.
- **Maven:** Manages the build lifecycle, dependencies, and packaging.
- **WAR:** Packages the web application for deployment.
- **Tomcat:** Runs the deployed web application.

## Official References

- JUnit 5 User Guide: https://junit.org/junit5/docs/current/user-guide/
- Maven Surefire Plugin: https://maven.apache.org/surefire/maven-surefire-plugin/
- Maven Build Lifecycle: https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
- Maven POM Reference: https://maven.apache.org/pom.html
