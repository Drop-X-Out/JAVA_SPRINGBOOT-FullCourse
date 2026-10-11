# 1. First: What is Thymeleaf?

Before writing any Thymeleaf code, give students this simple idea:

> **Thymeleaf is a server-side Java template engine that allows us to create dynamic HTML pages using normal HTML.**

Break that sentence down.

### What does "template engine" mean?

Suppose we have this HTML:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Student</title>
</head>
<body>

    <h1>Hello John</h1>

</body>
</html>
```

This page is **static**.

The browser receives:

```text
<h1>Hello John</h1>
```

and displays:

```text
Hello John
```

But suppose the name comes from Java:

```java
String name = "John";
```

We want Java to send the name to HTML.

Now the requirement becomes:

```text
Java
  ↓
Data
  ↓
HTML
  ↓
Browser
```

That's where a template engine becomes useful.

---

# 2. First understand the problem without Thymeleaf

Before introducing Thymeleaf, let's build a complete application using a Servlet.

This is important because students should first experience the problem.

We'll create:

```text
Browser
   ↓
HTTP Request
   ↓
Servlet
   ↓
Java creates HTML
   ↓
HTTP Response
   ↓
Browser
```

---

# 3. Create a Maven project

Project structure:

```text
servlet-html-demo
│
├── pom.xml
│
└── src
    └── main
        └── java
            └── com.example
                └── HelloServlet.java
```

We need a servlet container such as **Tomcat**.

---

# 4. Complete `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>servlet-html-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>

        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>6.0.0</version>
            <scope>provided</scope>
        </dependency>

    </dependencies>

    <build>

        <finalName>servlet-html-demo</finalName>

        <plugins>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.4.0</version>
            </plugin>

        </plugins>

    </build>

</project>
```

### Why `war`?

Because we're deploying a web application to Tomcat.

```xml
<packaging>war</packaging>
```

means Maven will build a:

```text
.war
```

file.

---

# 5. Create the Servlet

Create:

```text
src/main/java/com/example/HelloServlet.java
```

Complete code:

```java
package com.example;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        response.setContentType("text/html");

        PrintWriter out = response.getWriter();

        out.println("<!DOCTYPE html>");
        out.println("<html>");
        out.println("<head>");
        out.println("<title>Hello Page</title>");
        out.println("</head>");

        out.println("<body>");

        out.println("<h1>Hello John</h1>");
        out.println("<p>Welcome to Servlet.</p>");

        out.println("</body>");
        out.println("</html>");
    }
}
```

Now open:

```text
http://localhost:8080/servlet-html-demo/hello
```

The Servlet generates the HTML.

---

# 6. Understand what is actually happening

Students often think:

> "Servlet creates a webpage."

Technically, the Servlet doesn't create a webpage in the browser.

It creates an **HTTP response containing HTML**.

The flow is:

```text
Browser
   |
   | GET /hello
   ↓
Tomcat
   |
   ↓
HelloServlet
   |
   | creates HTML
   ↓
HTTP Response
   |
   | Content-Type: text/html
   ↓
Browser
```

The browser receives something like:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Hello Page</title>
</head>

<body>

    <h1>Hello John</h1>

    <p>Welcome to Servlet.</p>

</body>
</html>
```

and renders it.

---

# 7. Now introduce dynamic data

Suppose we have:

```java
String name = "John";
int age = 25;
```

and we want:

```text
Hello John
Your age is 25
```

Complete Servlet:

```java
package com.example;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/student")
public class StudentServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        response.setContentType("text/html");

        PrintWriter out = response.getWriter();

        String name = "John";
        int age = 25;

        out.println("<!DOCTYPE html>");
        out.println("<html>");

        out.println("<head>");
        out.println("<title>Student</title>");
        out.println("</head>");

        out.println("<body>");

        out.println("<h1>Hello " + name + "</h1>");
        out.println("<p>Your age is " + age + "</p>");

        out.println("</body>");

        out.println("</html>");
    }
}
```

This works.

But now ask students:

> **Would you like to create a 500-line HTML page using `out.println()`?**

Obviously, no.

---

# 8. The real problem with Servlet-generated HTML

Imagine an Amazon-like page.

You need:

```html
<header>
<nav>
<section>
<div>
<div>
<img>
<button>
<footer>
```

And Java is generating all of it:

```java
out.println("<header>");
out.println("<nav>");
out.println("<div>");
out.println("<h1>");
out.println("...");
out.println("</h1>");
out.println("</div>");
out.println("</nav>");
out.println("</header>");
```

This becomes extremely difficult to maintain.

The Servlet should ideally focus on:

```text
Request
Business logic
Data
Response
```

not:

```text
Creating hundreds of lines of HTML
```

This leads us to **JSP**.

---

# 9. Now introduce JSP

Tell students:

> JSP was introduced to make dynamic HTML easier to write than generating HTML from Java code.

Instead of:

```java
out.println("<h1>Hello " + name + "</h1>");
```

we can create an HTML file.

For example:

```text
src
└── main
    ├── java
    │   └── com.example
    │       └── StudentServlet.java
    │
    └── webapp
        └── student.jsp
```

---

# 10. Complete JSP example

### `student.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" %>

<!DOCTYPE html>
<html>

<head>
    <title>Student</title>
</head>

<body>

    <h1>Hello ${name}</h1>

    <p>Your age is ${age}</p>

</body>

</html>
```

But where do `name` and `age` come from?

The Servlet.

---

# 11. Complete Servlet + JSP

```java
package com.example;

import jakarta.servlet.RequestDispatcher;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/student")
public class StudentServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        String name = "John";
        int age = 25;

        request.setAttribute("name", name);
        request.setAttribute("age", age);

        RequestDispatcher dispatcher =
                request.getRequestDispatcher("/student.jsp");

        dispatcher.forward(request, response);
    }
}
```

Now the architecture becomes:

```text
Browser
   ↓
Servlet
   ↓
Java data
   ↓
request.setAttribute()
   ↓
JSP
   ↓
HTML
   ↓
Browser
```

This is much cleaner.

---

# 12. Why JSP was useful

JSP allows developers to write:

```html
<h1>Hello ${name}</h1>
```

instead of:

```java
out.println("<h1>Hello " + name + "</h1>");
```

So HTML becomes much easier to maintain.

But JSP has its own problems.

---

# 13. Why not JSP?

Don't teach this as:

> "JSP is bad."

Instead say:

> **JSP solved the Servlet HTML-generation problem, but modern Java applications often prefer template engines such as Thymeleaf.**

There are several reasons.

### Problem 1 — Java/JSP technology becomes mixed with HTML

Historically JSP pages could contain Java scriptlets:

```jsp
<%
    String name = "John";
%>

<h1>Hello <%= name %></h1>
```

Now Java and HTML are mixed together.

For a beginner:

```text
HTML
+
Java
+
JSP syntax
```

can become confusing.

---

# 14. Modern JSP with EL is cleaner

Modern JSP generally uses EL:

```jsp
<h1>Hello ${name}</h1>
```

That's much better.

But we're still working inside the JSP ecosystem.

Now let's look at what we really want.

---

# 15. What if our HTML could remain normal HTML?

Suppose we create:

```html
<!DOCTYPE html>
<html>

<head>
    <title>Student</title>
</head>

<body>

    <h1>Hello Student</h1>

</body>

</html>
```

This is completely normal HTML.

A browser can open it directly.

But we want Java to dynamically change:

```html
<h1>Hello Student</h1>
```

into:

```html
<h1>Hello John</h1>
```

without destroying the HTML structure.

**This is where Thymeleaf becomes very interesting.**

---

# 16. What does Thymeleaf actually do?

Thymeleaf lets us write:

```html
<h1 th:text="${name}">Hello Student</h1>
```

This is still an HTML element:

```html
<h1>
```

and:

```html
th:text
```

is a Thymeleaf attribute.

When Thymeleaf processes the page:

```text
${name}
   ↓
John
```

the final HTML becomes:

```html
<h1>Hello John</h1>
```

The browser receives the final HTML.

---

# 17. The important idea

Thymeleaf is **not JavaScript**.

It is **not running inside the browser**.

It runs on the **server**.

The flow is:

```text
              SERVER
                |
                |
Browser ──HTTP──→ Servlet
                |
                ↓
             Java data
                |
                ↓
            Thymeleaf
                |
                ↓
            HTML page
                |
                |
Browser ←─HTTP───
```

The browser ultimately receives normal HTML.

---

# 18. Why is it called a template engine?

Consider this:

```html
<h1>Hello <span th:text="${name}">Student</span></h1>
```

This HTML is a **template**.

`name` is dynamic.

For example:

```text
name = "John"
```

Thymeleaf processes the template:

```text
Template
   +
Data
   ↓
Thymeleaf
   ↓
Final HTML
```

Result:

```html
<h1>Hello <span>John</span></h1>
```

---

# 19. The most important Thymeleaf feature

This is the first syntax I would teach:

```html
th:text
```

Complete example:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<head>
    <title>Student</title>
</head>

<body>

    <h1 th:text="${name}">
        Student Name
    </h1>

</body>

</html>
```

The important part:

```html
<h1 th:text="${name}">
    Student Name
</h1>
```

If:

```text
name = John
```

the output becomes:

```html
<h1>
    John
</h1>
```

---

# 20. Why does Thymeleaf have normal HTML inside it?

This is one of the biggest advantages to explain to first-time learners.

Look at:

```html
<h1 th:text="${name}">
    Student Name
</h1>
```

If Thymeleaf isn't processing it and a browser simply opens the HTML file, the browser can still understand:

```html
<h1>Student Name</h1>
```

The browser ignores the unknown `th:text` attribute.

So the page can still behave like HTML.

This idea is often called **natural templating**.

---

# 21. Now build the first complete Thymeleaf application

Now we move from theory to an actual project.

Project:

```text
thymeleaf-demo
│
├── pom.xml
│
└── src
    └── main
        ├── java
        │   └── com.example
        │       └── StudentServlet.java
        │
        └── webapp
            └── WEB-INF
                └── templates
                    └── student.html
```

---

# 22. Complete `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>thymeleaf-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>war</packaging>

    <properties>

        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>

        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

    </properties>

    <dependencies>

        <!-- Servlet API -->

        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>6.0.0</version>
            <scope>provided</scope>
        </dependency>

        <!-- Thymeleaf -->

        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf</artifactId>
            <version>3.1.3.RELEASE</version>
        </dependency>

    </dependencies>

    <build>

        <finalName>thymeleaf-demo</finalName>

        <plugins>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.4.0</version>
            </plugin>

        </plugins>

    </build>

</project>
```

---

# 23. Create the HTML template

File:

```text
src/main/webapp/WEB-INF/templates/student.html
```

Complete code:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<head>

    <meta charset="UTF-8">

    <title>Student Page</title>

</head>

<body>

    <h1>Student Information</h1>

    <h2 th:text="${name}">
        Student Name
    </h2>

    <p th:text="${age}">
        Student Age
    </p>

</body>

</html>
```

---

# 24. But who processes this HTML?

We need Thymeleaf's `TemplateEngine`.

Our Servlet will:

1. receive request
2. create data
3. give data to Thymeleaf
4. ask Thymeleaf to process HTML
5. send resulting HTML to browser

---

# 25. Complete Servlet

```java
package com.example;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.WebContext;
import org.thymeleaf.web.IWebExchange;
import org.thymeleaf.web.servlet.JakartaServletWebApplication;

import java.io.IOException;

@WebServlet("/student")
public class StudentServlet extends HttpServlet {

    private TemplateEngine templateEngine;

    @Override
    public void init() throws ServletException {

        templateEngine = new TemplateEngine();
    }

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        String name = "John";
        int age = 25;

        JakartaServletWebApplication application =
                JakartaServletWebApplication.buildApplication(getServletContext());

        IWebExchange exchange =
                application.buildExchange(request, response);

        WebContext context =
                new WebContext(exchange);

        context.setVariable("name", name);
        context.setVariable("age", age);

        response.setContentType("text/html;charset=UTF-8");

        templateEngine.process(
                "student",
                context,
                response.getWriter()
        );
    }
}
```

Now:

```text
http://localhost:8080/thymeleaf-demo/student
```

will render the page.

---

# 26. Understand the Servlet carefully

The first important line:

```java
private TemplateEngine templateEngine;
```

This represents Thymeleaf's engine.

Then:

```java
templateEngine = new TemplateEngine();
```

creates the engine.

Then:

```java
WebContext context =
        new WebContext(exchange);
```

creates the context containing information Thymeleaf can use.

Then:

```java
context.setVariable("name", name);
```

means:

```text
"name" → "John"
```

And:

```java
context.setVariable("age", age);
```

means:

```text
"age" → 25
```

Then:

```java
templateEngine.process(
        "student",
        context,
        response.getWriter()
);
```

means:

> Thymeleaf, process the `student` template using this data and write the resulting HTML into the HTTP response.

---

# 27. Connect Java and HTML

This is the most important mental model.

Java:

```java
String name = "John";

context.setVariable("name", name);
```

HTML:

```html
<h2 th:text="${name}">
    Student Name
</h2>
```

Connection:

```text
Java

"name" → "John"

        ↓

Thymeleaf Context

        ↓

${name}

        ↓

th:text

        ↓

<h2>John</h2>
```

---

# 28. Now compare all three approaches

### Approach 1 — Servlet generates HTML

```java
out.println("<h1>Hello " + name + "</h1>");
```

Architecture:

```text
Servlet
  ↓
HTML generation
  ↓
Browser
```

Problem:

**Java becomes full of HTML.**

---

### Approach 2 — JSP

```jsp
<h1>${name}</h1>
```

Architecture:

```text
Servlet
  ↓
JSP
  ↓
HTML
  ↓
Browser
```

Much cleaner.

---

### Approach 3 — Thymeleaf

```html
<h1 th:text="${name}">
    Student Name
</h1>
```

Architecture:

```text
Servlet
  ↓
Java data
  ↓
Thymeleaf
  ↓
HTML
  ↓
Browser
```

---

# 29. Where does JPA come into this?

This is where your students can connect everything they've already learned.

Suppose JPA gets a Student from the database.

Entity:

```java
@Entity
public class Student {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    public Student() {
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```

JPA retrieves:

```text
Student
id = 1
name = John
age = 25
```

Servlet receives the object.

Then:

```java
context.setVariable("student", student);
```

Thymeleaf can access:

```html
<h1 th:text="${student.name}">
    Student Name
</h1>
```

and:

```html
<p th:text="${student.age}">
    Student Age
</p>
```

So your complete architecture becomes:

```text
                 DATABASE
                     ↑
                     |
                    JPA
                     |
                     ↓
                  Entity
                     |
                     ↓
                  Servlet
                     |
                     ↓
               Thymeleaf
                     |
                     ↓
                   HTML
                     |
                     ↓
                  Browser
```

That is the point where students start seeing the entire backend architecture.

---

# 30. The bigger picture

You have already taught them:

```text
JDBC
 ↓
Database
```

then:

```text
JPA
 ↓
Database
```

and:

```text
Servlet
 ↓
HTTP
```

Now Thymeleaf connects the backend to the frontend:

```text
                 USER
                  |
                  ↓
              BROWSER
                  |
               Request
                  |
                  ↓
               SERVLET
                  |
          ┌───────┴───────┐
          ↓               ↓
        JPA          Business Logic
          ↓
      DATABASE
          |
          ↓
       Entity
          |
          └───────┐
                  ↓
              THYMELEAF
                  |
                  ↓
                HTML
                  |
                  ↓
              RESPONSE
                  |
                  ↓
               BROWSER
```

This is the **main conceptual lesson** I would establish before teaching individual Thymeleaf attributes.

---

# 31. Then start Thymeleaf syntax

Only after students understand the architecture should you start:

### `th:text`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<head>
    <title>Student</title>
</head>

<body>

    <h1 th:text="${name}">
        Student Name
    </h1>

</body>

</html>
```

### `th:if`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<head>
    <title>Student</title>
</head>

<body>

    <h1 th:text="${name}">
        Student Name
    </h1>

    <p th:if="${age >= 18}">
        Student is an adult.
    </p>

</body>

</html>
```

### `th:each`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<head>
    <title>Students</title>
</head>

<body>

    <h1>Students</h1>

    <ul>

        <li th:each="student : ${students}"
            th:text="${student.name}">
            Student Name
        </li>

    </ul>

</body>

</html>
```

### `th:href`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<head>
    <title>Student</title>
</head>

<body>

    <a th:href="@{/student}">
        View Student
    </a>

</body>

</html>
```

### `th:value`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<head>
    <title>Student Form</title>
</head>

<body>

    <form>

        <input
            type="text"
            th:value="${name}"
        >

        <button type="submit">
            Submit
        </button>

    </form>

</body>

</html>
```

---

