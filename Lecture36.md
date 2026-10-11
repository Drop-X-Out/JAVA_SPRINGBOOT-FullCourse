# Testing Java Web Applications Using HttpUnit and Mockito

A complete Maven tutorial for beginners covering HTTP testing, Servlet testing, mocking dependencies, JUnit integration, and running test cases for a Java WAR application.

## Table of Contents

1. Understanding HttpUnit and Mockito
2. Difference between HttpUnit and Mockito
3. Project setup and prerequisites
4. Configuring Maven dependencies
5. Understanding the project structure
6. Testing a Servlet using HttpUnit
7. Understanding HttpUnit code line by line
8. Understanding Mockito
9. Creating a Student Repository
10. Creating a Student Service
11. Writing Mockito test cases
12. Running Mockito tests
13. Running HttpUnit tests
14. Understanding test reports
15. Common errors and solutions
16. Best practices
17. Practice exercises
18. Interview questions

---

# 1. Understanding HttpUnit and Mockito

When developing a Java web application, we need to test different parts of our code.

For example, suppose we are developing a Student Management System.

Our application has:

- Servlets to handle HTTP requests.
- Services to implement business logic.
- Repositories to access student data.
- HTML pages to display results.

How can we test these components?

We can use different testing tools for different purposes.

## What is HttpUnit?

HttpUnit is a Java library designed to test web applications through HTTP requests and responses.

It can help us verify:

- Whether a web page responds correctly.
- Whether the response contains expected text.
- Whether a page contains the expected HTML.
- Whether a web application behaves correctly from the client's perspective.

For example, suppose our Servlet is available at:

```text
http://localhost:8080/hello-maven/hello
```

We want to verify that it returns:

```text
Hello from Maven and Tomcat!
```

Instead of opening the browser and checking manually, we can write an automated HTTP test.

## What is Mockito?

Mockito is a Java mocking framework used to create fake objects that simulate the behavior of real dependencies during tests.

Suppose a Student Service needs a Student Repository to retrieve student records.

Normally, the repository might access a database.

However, when testing the service, we may not want to connect to a real database.

Mockito lets us create a mock repository and specify what it should return.

For example:

```java
when(studentRepository.findNameById(101))
        .thenReturn("Rahul");
```

This tells the mock repository to return `"Rahul"` when the service calls `findNameById(101)`.

We can then test the service without accessing a real database.

---

# 2. Difference Between HttpUnit and Mockito

| Feature | HttpUnit | Mockito |
|---|---|---|
| Main purpose | Test web interactions | Mock dependencies |
| Testing style | HTTP-level or black-box testing | Unit testing |
| Works with | Web applications and HTTP endpoints | Java objects and methods |
| Requires a running server for ordinary HTTP tests | Yes, when targeting a deployed application | No |
| Requires a real database | Not necessarily | No, when dependencies are mocked |
| Example | Check the response from a Servlet | Mock a repository used by a service |

**Important:** These libraries are not replacements for each other.

HttpUnit checks the behavior of a web application through HTTP. Mockito isolates individual Java components from their dependencies.

We will use JUnit 5 to execute and organize the test methods for both.

---

# 3. Project Setup and Prerequisites

This tutorial continues with the Maven WAR application from the previous lesson.

Use the following environment:

- Java 17
- Maven 3.9.x
- Apache Tomcat 10.1.x
- JUnit 5
- Mockito
- HttpUnit

## Important compatibility note about HttpUnit

The commonly published HttpUnit artifact, version `1.7`, is an older library. Its dependency metadata includes legacy Servlet API dependencies and predates modern Jakarta Servlet applications.

Our Servlet application uses `jakarta.servlet` and Tomcat 10.1.

The HTTP test below accesses the running application through an ordinary HTTP URL, so the test does not need to import the Servlet API itself. However, HttpUnit 1.7 is legacy software and may have compatibility or dependency-resolution issues in a modern Java environment.

If your project requires long-term support for modern Jakarta applications, consider a maintained HTTP-testing alternative. The tutorial still demonstrates HttpUnit because it is the requested topic.

---

# 4. Configuring Maven Dependencies

Open the existing `pom.xml` in the root of your project.

Add the JUnit, Mockito, and HttpUnit dependencies.

## Complete `pom.xml`

The following file keeps the WAR packaging and adds the required testing libraries.

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

        <!-- Servlet API supplied by Tomcat -->
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>6.0.0</version>
            <scope>provided</scope>
        </dependency>

        <!-- JUnit 5 -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.13.4</version>
            <scope>test</scope>
        </dependency>

        <!-- Mockito integration with JUnit 5 -->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-junit-jupiter</artifactId>
            <version>5.14.2</version>
            <scope>test</scope>
        </dependency>

        <!-- HttpUnit for HTTP testing -->
        <dependency>
            <groupId>httpunit</groupId>
            <artifactId>httpunit</artifactId>
            <version>1.7</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>

        <finalName>hello-maven</finalName>

        <plugins>

            <!-- Compile Java code -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
            </plugin>

            <!-- Execute JUnit tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
            </plugin>

            <!-- Package application into WAR -->
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

### Understanding the dependencies

**JUnit 5**

Provides the `@Test` annotation and assertions such as `assertEquals()` and `assertTrue()`.

**Mockito JUnit Jupiter**

Provides Mockito functionality and integration with the JUnit 5 extension model.

**HttpUnit**

Provides classes such as `WebConversation`, `WebRequest`, and `WebResponse` for HTTP-oriented testing.

**Maven Surefire**

Discovers and executes the unit tests during the Maven test lifecycle.

The dependencies have `test` scope because they are required for testing rather than normal production execution.

---

# 5. Understanding the Project Structure

Our project will contain the following files:

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
    │   │           ├── HelloServlet.java
    │   │           ├── StudentRepository.java
    │   │           └── StudentService.java
    │   │
    │   └── webapp/
    │       └── index.html
    │
    └── test/
        └── java/
            └── com/
                └── example/
                    ├── HelloServletHttpUnitIT.java
                    └── StudentServiceTest.java
```

The `main` directory contains production code.

The `test` directory contains test code.

We will create two separate tests:

- `HelloServletHttpUnitIT.java` tests the deployed web application over HTTP.
- `StudentServiceTest.java` tests business logic using Mockito.

---

# 6. Testing a Servlet Using HttpUnit

First, we need a Servlet to test.

Create the following file:

`src/main/java/com/example/HelloServlet.java`

## Complete `HelloServlet.java`

```java
package com.example;

import java.io.IOException;
import java.io.PrintWriter;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();

        out.println("<!DOCTYPE html>");
        out.println("<html>");
        out.println("<head>");
        out.println("<title>Hello Page</title>");
        out.println("</head>");
        out.println("<body>");

        out.println("<h1>Hello from Maven and Tomcat!</h1>");
        out.println("<p>HttpUnit testing example</p>");

        out.println("</body>");
        out.println("</html>");
    }
}
```

The Servlet responds to a GET request at the `/hello` mapping.

When deployed as `hello-maven.war`, the full URL will normally be:

```text
http://localhost:8080/hello-maven/hello
```

The Servlet returns HTML containing the text we want to verify.

---

# 7. Writing an HttpUnit Test Case

Create this file:

`src/test/java/com/example/HelloServletHttpUnitIT.java`

## Complete code

```java
package com.example;

import com.meterware.httpunit.GetMethodWebRequest;
import com.meterware.httpunit.WebConversation;
import com.meterware.httpunit.WebRequest;
import com.meterware.httpunit.WebResponse;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

class HelloServletHttpUnitIT {

    @Test
    void testHelloServletResponse() throws Exception {

        // Arrange
        WebConversation conversation = new WebConversation();

        WebRequest request = new GetMethodWebRequest(
                "http://localhost:8080/hello-maven/hello"
        );

        // Act
        WebResponse response = conversation.getResponse(request);

        // Assert
        assertEquals(200, response.getResponseCode());

        assertTrue(
                response.getText().contains(
                        "Hello from Maven and Tomcat!"
                )
        );
    }
}
```

## Understanding the code

### Step 1: Create a web conversation

```java
WebConversation conversation = new WebConversation();
```

`WebConversation` manages the HTTP interaction.

It allows the test to send a request and receive a response.

### Step 2: Create a GET request

```java
WebRequest request = new GetMethodWebRequest(
        "http://localhost:8080/hello-maven/hello"
);
```

This defines an HTTP GET request to our deployed Servlet.

The URL must match your actual Tomcat port and application context path.

### Step 3: Send the request

```java
WebResponse response = conversation.getResponse(request);
```

HttpUnit sends the request and obtains the HTTP response.

**Important:** Tomcat must already be running, and the application must be deployed before this test executes.

### Step 4: Check the HTTP status

```java
assertEquals(200, response.getResponseCode());
```

HTTP status `200` indicates a successful request.

If the application is missing or the URL is wrong, you may receive a different status or encounter a connection error.

### Step 5: Check the page content

```java
assertTrue(
        response.getText().contains(
                "Hello from Maven and Tomcat!"
        )
);
```

This verifies that the HTML response contains the expected message.

If the message is missing, the assertion fails.

## What have we tested?

We have tested the deployed application's externally visible behavior.

```text
JUnit test
    |
    v
HttpUnit GET request
    |
    v
Tomcat
    |
    v
HelloServlet
    |
    v
HTTP response
    |
    v
Assertions
```

Unlike a direct Java method test, this test checks the actual HTTP interaction.

---

# 8. Understanding Mockito

Now let's learn Mockito.

Suppose our application contains a Student Service.

The service retrieves student names from a repository.

The repository might eventually connect to a database.

When testing the service, we want to verify the service's behavior without needing the database.

Mockito lets us create a mock repository.

A mock is an object whose behavior is controlled by our test.

For example:

```java
when(repository.findNameById(101))
        .thenReturn("Rahul");
```

This tells Mockito:

"When the mock repository receives a call to `findNameById(101)`, return `Rahul`."

We can then call the service and verify its result.

---

# 9. Creating the Student Repository

Create this file:

`src/main/java/com/example/StudentRepository.java`

## Complete code

```java
package com.example;

public interface StudentRepository {

    String findNameById(int studentId);
}
```

This interface represents the data-access layer.

In a real application, an implementation might query a database using JDBC, JPA, or Hibernate.

For our Mockito test, we do not need a real implementation because Mockito can mock the interface.

---

# 10. Creating the Student Service

Create this file:

`src/main/java/com/example/StudentService.java`

## Complete code

```java
package com.example;

public class StudentService {

    private final StudentRepository studentRepository;

    public StudentService(StudentRepository studentRepository) {
        this.studentRepository = studentRepository;
    }

    public String getStudentName(int studentId) {

        if (studentId <= 0) {
            throw new IllegalArgumentException(
                    "Student ID must be positive"
            );
        }

        String studentName =
                studentRepository.findNameById(studentId);

        if (studentName == null) {
            return "Student not found";
        }

        return studentName;
    }
}
```

## Understanding the code

### Dependency injection through the constructor

```java
public StudentService(StudentRepository studentRepository) {
    this.studentRepository = studentRepository;
}
```

The service receives its repository from outside rather than creating the repository internally.

This design makes the service easier to test.

During normal application execution, we can provide a real repository implementation.

During testing, we can provide a Mockito mock.

### Validating the student ID

```java
if (studentId <= 0) {
    throw new IllegalArgumentException(
            "Student ID must be positive"
    );
}
```

Invalid IDs are rejected before the repository is called.

### Calling the repository

```java
String studentName =
        studentRepository.findNameById(studentId);
```

The service requests the student name from the repository.

### Handling missing students

```java
if (studentName == null) {
    return "Student not found";
}
```

If the repository has no name for that ID, the service returns a message.

These behaviors give us several scenarios to test.

---

# 11. Writing Mockito Test Cases

Create the file:

`src/test/java/com/example/StudentServiceTest.java`

## Complete `StudentServiceTest.java`

```java
package com.example;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;

import org.mockito.Mock;
import org.mockito.InjectMocks;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

import static org.mockito.Mockito.when;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.never;

@ExtendWith(MockitoExtension.class)
class StudentServiceTest {

    @Mock
    private StudentRepository studentRepository;

    @InjectMocks
    private StudentService studentService;

    @BeforeEach
    void setUp() {
        // Mockito initializes the mock and injects it into the service.
    }

    @Test
    void testGetStudentName() {

        // Arrange
        when(studentRepository.findNameById(101))
                .thenReturn("Rahul");

        // Act
        String result = studentService.getStudentName(101);

        // Assert
        assertEquals("Rahul", result);

        verify(studentRepository).findNameById(101);
    }

    @Test
    void testStudentNotFound() {

        // Arrange
        when(studentRepository.findNameById(999))
                .thenReturn(null);

        // Act
        String result = studentService.getStudentName(999);

        // Assert
        assertEquals("Student not found", result);

        verify(studentRepository).findNameById(999);
    }

    @Test
    void testInvalidStudentId() {

        // Act and Assert
        assertThrows(
                IllegalArgumentException.class,
                () -> studentService.getStudentName(0)
        );

        verify(
                studentRepository,
                never()
        ).findNameById(0);
    }
}
```

## Understanding the annotations

### `@ExtendWith(MockitoExtension.class)`

```java
@ExtendWith(MockitoExtension.class)
```

This integrates Mockito with JUnit 5.

It initializes Mockito's annotations and manages the mocks for each test.

### `@Mock`

```java
@Mock
private StudentRepository studentRepository;
```

Mockito creates a mock of the `StudentRepository` interface.

The mock is not a real database repository.

Its behavior is configured inside each test.

### `@InjectMocks`

```java
@InjectMocks
private StudentService studentService;
```

Mockito creates the service and injects the mock repository into its constructor.

This allows the service to be tested without a real repository implementation.

### `@BeforeEach`

```java
@BeforeEach
void setUp() {
}
```

JUnit executes this method before each test.

We do not need to initialize the Mockito annotations manually here because `MockitoExtension` handles the setup. The method is included to demonstrate the lifecycle hook and can be removed without changing the test behavior.

---

# 12. Understanding Mockito Methods

## 12.1 `when().thenReturn()`

```java
when(studentRepository.findNameById(101))
        .thenReturn("Rahul");
```

This defines the behavior of the mock.

When the service calls the repository with ID `101`, Mockito returns `"Rahul"`.

No database is accessed.

## 12.2 `verify()`

```java
verify(studentRepository).findNameById(101);
```

This verifies that the repository method was called with the specified argument.

It helps us check whether the service interacted with its dependency as expected.

## 12.3 `never()`

```java
verify(studentRepository, never())
        .findNameById(0);
```

This verifies that the repository method was not called with ID `0`.

That matters because our service should reject an invalid ID before accessing the repository.

## 12.4 `assertThrows()`

```java
assertThrows(
        IllegalArgumentException.class,
        () -> studentService.getStudentName(0)
);
```

This verifies that invalid input produces the expected exception.

Together, these methods test both the service's result and its interactions with its dependency.

---

# 13. Running Mockito Tests Using Maven

Mockito tests do not require Tomcat to be running.

They execute Java code directly inside the test process.

Open a terminal in the project root, where `pom.xml` is located.

Run:

```bash
mvn -Dtest=StudentServiceTest test
```

Maven will compile the application and test classes, then execute `StudentServiceTest`.

Expected summary:

```text
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

BUILD SUCCESS
```

The exact console output depends on your Maven and plugin versions.

## What did we test?

| Test | Expected behavior |
|---|---|
| `testGetStudentName()` | Returns `Rahul` for student ID 101 |
| `testStudentNotFound()` | Returns `Student not found` for a missing student |
| `testInvalidStudentId()` | Throws an exception and does not call the repository |

These tests are fast because they do not depend on a running server or real database.

---

# 14. Running HttpUnit Tests

The HttpUnit test requires a running web application.

Follow these steps.

## Step 1: Build the WAR

Run:

```bash
mvn clean package
```

This compiles the application, runs the default unit tests, and packages the WAR if those tests pass.

The generated file should be:

```text
target/hello-maven.war
```

Our HttpUnit test class ends in `IT`, so Maven Surefire's default test naming patterns will not normally execute it as part of this command.

This is intentional because the HTTP test requires an externally running server.

## Step 2: Deploy the WAR

Copy:

```text
target/hello-maven.war
```

Into Tomcat's `webapps` directory.

Start Tomcat and wait for the application to deploy.

## Step 3: Verify the endpoint

Open this URL in a browser:

```text
http://localhost:8080/hello-maven/hello
```

Confirm that the page displays:

```text
Hello from Maven and Tomcat!
```

## Step 4: Run the HttpUnit test

With Tomcat running and the application deployed, execute:

```bash
mvn -Dtest=HelloServletHttpUnitIT test
```

This explicitly selects the HttpUnit test.

If the test dependencies resolve successfully and the deployed application returns the expected response, the test should pass.

**Note:** If the old HttpUnit dependency fails to compile or execute in your environment, investigate the compatibility issue described in Section 3 rather than assuming that your Servlet code is incorrect.

---

# 15. Understanding Test Reports

Maven Surefire normally stores test reports under:

```text
target/surefire-reports/
```

For example:

```text
target/
└── surefire-reports/
    ├── com.example.StudentServiceTest.txt
    └── TEST-com.example.StudentServiceTest.xml
```

A successful report may contain:

```text
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0
```

A failed assertion might produce:

```text
expected: <Rahul> but was: <Student not found>
```

This tells you that the actual result differs from the expected result.

Check the report and stack trace to identify the problem.

The HTTP test also generates a Surefire report when executed through Surefire.

---

# 16. Common Errors and Solutions

## Error 1: Connection refused

Example:

```text
java.net.ConnectException: Connection refused
```

**Cause:** The HTTP test cannot connect to Tomcat.

Check:

- Whether Tomcat is running.
- Whether port `8080` is correct.
- Whether the application is deployed.
- Whether another port is configured in Tomcat.

## Error 2: HTTP 404

**Cause:** The requested URL does not map to the deployed application or Servlet.

Check:

```text
http://localhost:8080/hello-maven/hello
```

Ensure that:

- The WAR is named `hello-maven.war`.
- The Servlet uses `@WebServlet("/hello")`.
- Tomcat successfully deployed the application.

## Error 3: Mockito annotations are not initialized

If Mockito annotations are not initialized, confirm that the test uses:

```java
@ExtendWith(MockitoExtension.class)
```

and includes the `mockito-junit-jupiter` dependency.

## Error 4: Mockito test unexpectedly accesses a database

Check that the service receives the mocked repository.

The service should use constructor injection rather than creating its own repository internally.

For example:

```java
StudentService service =
        new StudentService(mockRepository);
```

Mockito's `@InjectMocks` can automate this for the example shown earlier.

## Error 5: HttpUnit dependency or compilation errors

HttpUnit 1.7 is an older artifact and may bring in legacy libraries.

Check Maven's error output and dependency tree:

```bash
mvn dependency:tree
```

Confirm that the test imports the correct HttpUnit package names and that dependencies resolve.

Do not randomly add different Servlet API versions to solve a test dependency issue; that can introduce additional conflicts.

## Error 6: `mvn clean package` fails before creating the WAR

The build may fail because a test fails or a dependency cannot be resolved.

Read the first meaningful error in the Maven output.

For troubleshooting the production build independently, you can run:

```bash
mvn -DskipTests package
```

This skips test execution but still compiles the test sources. It should be used only as a diagnostic or temporary workaround, not as proof that the tests pass.

---

# 17. Best Practices

1. Use Mockito for isolated unit tests of service classes.
2. Use HTTP-level tests to verify externally visible web behavior.
3. Keep unit tests independent of Tomcat and the database whenever possible.
4. Use clear names such as `testStudentNotFound()`.
5. Test both successful and unsuccessful scenarios.
6. Verify interactions only when those interactions are important to the behavior.
7. Do not let an HTTP integration test silently depend on an undocumented local server.
8. Keep test dependencies in Maven's `test` scope.
9. Run unit tests before packaging the WAR.
10. For new projects, evaluate actively maintained HTTP-testing tools when choosing an alternative to legacy HttpUnit.

---

# 18. Practice Exercises

### Exercise 1: Test another student

Modify the Mockito test to return `"Aman"` for student ID `102`.

Verify that `getStudentName(102)` returns `"Aman"`.

### Exercise 2: Test an invalid negative ID

Write a test that calls:

```java
studentService.getStudentName(-5);
```

Verify that an `IllegalArgumentException` is thrown.

Also verify that the repository is not called.

### Exercise 3: Change the Servlet response

Change the heading returned by `HelloServlet`.

Run the HttpUnit test and observe whether it passes or fails.

### Exercise 4: Test a missing Servlet

Change the HTTP test URL to a path that does not exist.

Observe the response and explain why the status assertion fails.

### Exercise 5: Add a new service method

Create a method that returns a student's name in uppercase.

Use Mockito to supply the original name, and use JUnit assertions to verify the uppercase result.

### Exercise 6: Compare the two testing approaches

Write down which parts of the application each tool verifies.

Explain why a passing Mockito test does not automatically prove that Tomcat can deploy the WAR.

---

# 19. Interview Questions

**Q1. What is HttpUnit?**

HttpUnit is a Java library designed to test web applications by simulating web interactions and examining responses.

**Q2. What is Mockito?**

Mockito is a framework for creating mock objects and controlling their behavior during tests.

**Q3. What is a mock object?**

A mock is a test double whose behavior and interactions can be configured and verified.

**Q4. What is the difference between HttpUnit and Mockito?**

HttpUnit tests web interactions, while Mockito isolates Java components by mocking dependencies.

**Q5. Why do we use `when().thenReturn()`?**

It configures the result a mock should return for a specified method call.

**Q6. What does `verify()` do in Mockito?**

It checks whether a particular interaction occurred.

**Q7. Does Mockito require a real database?**

No. Mockito can simulate a repository's behavior without a database.

**Q8. Does HttpUnit require Tomcat?**

Not every HttpUnit testing mode requires a real server, but the HTTP test in this tutorial targets a deployed application and therefore requires a reachable server.

**Q9. What is the purpose of `@InjectMocks`?**

It asks Mockito to create the test subject and inject suitable mock dependencies.

**Q10. Why use `@ExtendWith(MockitoExtension.class)`?**

It integrates Mockito with JUnit 5 and initializes its annotations.

**Q11. Can Mockito test a Servlet response directly?**

Mockito can mock Servlet-related dependencies and test Servlet logic in isolation, but it does not automatically simulate a complete deployed web application.

**Q12. Can HttpUnit replace unit testing?**

No. HTTP-level tests and unit tests cover different concerns and complement each other.

**Q13. Why do we separate `src/main/java` and `src/test/java`?**

This separates production code from test code and follows Maven's standard project structure.

**Q14. What is the difference between a unit test and an integration test?**

A unit test focuses on a small component in isolation. An integration test checks interactions between multiple components or systems.

---

# Final Summary

We have learned how to use HttpUnit and Mockito with Maven and JUnit.

The overall testing workflow is:

```text
                Java Web Application
                         |
             +-----------+-----------+
             |                       |
             v                       v
       Business Logic           Deployed Servlet
             |                       |
             v                       v
          Mockito                 HttpUnit
             |                       |
             v                       v
       Mock dependencies        HTTP request
             |                       |
             v                       v
       Verify method result     Verify HTTP response
             |                       |
             +-----------+-----------+
                         |
                         v
                  Automated Tests
                         |
                         v
                 Maven Test Reports
```

Remember:

- **JUnit** organizes and executes test methods.
- **Mockito** simulates dependencies for isolated testing.
- **HttpUnit** tests HTTP interactions with a web application.
- **Maven Surefire** runs unit tests during the Maven test lifecycle.
- **Tomcat** runs the deployed WAR.
