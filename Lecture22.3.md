# ServletRequest and HttpServletRequest in Java — Complete Tutorial

Beginner-friendly notes • Complete project • Folder structure • URL analysis • Request handling

This tutorial explains `ServletRequest`, `HttpServletRequest`, `getServletPath()`, `getPathInfo()`, and `getQueryString()` from the beginning, with a complete Java Servlet project that students can run in Apache Tomcat.

# 1. First understand: What is a request?

Imagine a student opens this URL in a browser:

```
http://localhost:8080/RequestDemo/student/profile/101?name=Ali&semester=3
```

The browser sends a request to the server. The server receives information such as:

- Which application the student wants to access.
- Which resource or servlet should handle the request.
- Additional path information.
- Query parameters such as `name=Ali`.
- The HTTP method, such as GET or POST.

A Java Servlet can inspect this information using the request object.

Browser

Sends an HTTP request

Apache Tomcat

Receives and routes the request

Java Servlet

Reads request information and generates a response

# 2. What is `ServletRequest`?

`ServletRequest` is an interface from the Java Servlet API. It represents a client's request to a server and provides general request information.

Package:

```
jakarta.servlet.ServletRequest
```

It provides methods for tasks such as:

- Reading request parameters.
- Reading request attributes.
- Accessing input data.
- Getting the request character encoding.
- Accessing information about the server and client.

For example:

```
String name = request.getParameter("name");
```

If the URL is:

```
http://localhost:8080/RequestDemo/student?name=Ali
```

Then:

```
request.getParameter("name");
```

returns:

```
Ali
```

## Where is `ServletRequest` used?

It provides common request functionality that is not specific to HTTP. In a typical web application, you will work with HTTP requests, so you will generally use `HttpServletRequest`.

# 3. What is `HttpServletRequest`?

`HttpServletRequest` is an interface that extends `ServletRequest` and adds functionality specifically for HTTP.

Package:

```
jakarta.servlet.http.HttpServletRequest
```

It supports features such as:

- HTTP request methods like GET and POST.
- HTTP headers.
- Cookies.
- Sessions.
- The request URI and context path.
- Servlet path and extra path information.
- The query string.

For example:

```
String method = request.getMethod();
```

If the browser sends a GET request, the result is:

```
GET
```

## Relationship between the two interfaces

ServletRequest

General request functionality

HttpServletRequest

General functionality + HTTP-specific functionality

The key Java concept is inheritance between interfaces: `HttpServletRequest` extends `ServletRequest`.

A servlet's `doGet()` method receives an `HttpServletRequest` object, so it can use methods from both interfaces.

```
@Override
protected void doGet(
        HttpServletRequest request,
        HttpServletResponse response) {

    String name = request.getParameter("name");
    String method = request.getMethod();
}
```

Notice that `request` can access both `getParameter()` and `getMethod()`.

# 4. Understand the URL before learning the three methods

We will use this example throughout the tutorial:

```
http://localhost:8080/RequestDemo/student/profile/101?name=Ali&semester=3
```

Break the URL into parts.

PROTOCOL + HOST + PORT

[http://localhost:8080](http://localhost:8080)

The server address

CONTEXT PATH

### /RequestDemo

The deployed web application's name

SERVLET PATH

### /student

The URL pattern that maps to our servlet

PATH INFO

### /profile/101

Additional path after the servlet mapping

QUERY STRING

### name=Ali&semester=3

Additional data after the question mark (?)

These parts are interpreted according to the servlet mapping and the web container. In this project, we will deliberately configure the mapping as `/student/*` so that the difference between the servlet path and path info is easy to observe.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://tomcat.apache.org\&sz=32)

Apache Tomcat 10.1.59)

+1



## 5. `getServletPath()`

Definition: Returns the part of the URL that matches the servlet's mapped path, excluding extra path information and the query string.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://tomcat.apache.org\&sz=32)

Apache Tomcat 10.1.59)



Suppose the servlet has this mapping:

```
@WebServlet("/student/*")
public class StudentServlet extends HttpServlet {
}
```

And the browser requests:

```
http://localhost:8080/RequestDemo/student/profile/101?name=Ali
```

Then:

```
String servletPath = request.getServletPath();
System.out.println(servletPath);
```

Output:

```
/student
```

### Why is the output `/student`?

Because `/student/*` is the mapping pattern.

- `/student` is the fixed part matched as the servlet path.
- `/*` allows additional path information.
- `/profile/101` is the extra part, so it is available through `getPathInfo()`.

The asterisk is a mapping wildcard, not a literal part of the URL.

## 6. `getPathInfo()`

Definition: Returns the additional path information after the servlet path and before the query string. If there is no additional path information, it returns `null`.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://tomcat.apache.org\&sz=32)

Apache Tomcat 10.1.59)

+1



Example:

```
String pathInfo = request.getPathInfo();
System.out.println(pathInfo);
```

For this URL:

```
/RequestDemo/student/profile/101?name=Ali
```

Output:

```
/profile/101
```

### Real-world use: Dynamic routes

Imagine your application has many student profiles:

```
/student/profile/101
/student/profile/102
/student/profile/103
```

You don't necessarily need to create a separate servlet for every student. You can use one servlet mapped to `/student/*` and inspect the extra path.

For example:

```
String pathInfo = request.getPathInfo();

if ("/profile/101".equals(pathInfo)) {
    // Handle student 101
} else if ("/profile/102".equals(pathInfo)) {
    // Handle student 102
}
```

This is a simple teaching example. In a larger application, you would normally parse the path and retrieve the student ID from a database rather than hardcoding every route.

## 7. `getQueryString()`

Definition: Returns the raw query string appearing after `?` in the URL. It does not automatically return a map of parameters, and it returns `null` when no query string is present.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://tomcat.apache.org\&sz=32)

Apache Tomcat 10.1.59)

+1



Example URL:

```
/RequestDemo/student/profile/101?name=Ali&semester=3
```

Code:

```
String queryString = request.getQueryString();
System.out.println(queryString);
```

Output:

```
name=Ali&semester=3
```

### Important: `getQueryString()` vs `getParameter()`

```
request.getQueryString();
```

Returns the entire raw query string:

```
name=Ali&semester=3
```

Whereas:

```
request.getParameter("name");
request.getParameter("semester");
```

Returns individual parameter values:

```
Ali
3
```

Use `getQueryString()` when you need to inspect the raw query portion. Use `getParameter()` when you want to read an individual parameter.

# 8. Complete project: Build it from scratch

We will create a working Maven web application using:

- Java 17
- Apache Tomcat 10.1
- Jakarta Servlet API 6.0
- Maven
- Annotation-based servlet configuration

Tomcat 10.1 uses the `jakarta.servlet` package, so do not use the older `javax.servlet` imports in this project.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://tomcat.apache.org\&sz=32)

Apache Tomcat 10.1.59)

+1



## Folder structure

Create a Maven project named `RequestDemo` with this structure:

RequestDemo/

`pom.xml`

`src/`

`main/`

`java/`

`com/example/`

`StudentServlet.java`

`webapp/`

`index.html`

We will use `@WebServlet` for mapping, so a `web.xml` file is not required.

## Step 1. Create `pom.xml`

This file defines the project dependencies and tells Maven to package the application as a WAR file.

```

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>RequestDemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.release>17</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
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
        <finalName>RequestDemo</finalName>
    </build>

</project>

```

### Understand the important parts

- `groupId`: Identifies the organization or project group.
- `artifactId`: Identifies the Maven project.
- `packaging`: `war` means Web Application Archive.
- `jakarta.servlet-api`: Provides the servlet interfaces and classes needed to compile the application.
- `provided`: Tomcat supplies the Servlet API at runtime, so the application should not package its own copy.

## Step 2. Create `StudentServlet.java`

Location:

`src/main/java/com/example/StudentServlet.java`

```

package com.example;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/student/*")
public class StudentServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        response.setContentType("text/html;charset=UTF-8");

        // Read request information
        String method = request.getMethod();
        String contextPath = request.getContextPath();
        String servletPath = request.getServletPath();
        String pathInfo = request.getPathInfo();
        String queryString = request.getQueryString();
        String requestURI = request.getRequestURI();

        // Read individual query parameters
        String name = request.getParameter("name");
        String semester = request.getParameter("semester");

        PrintWriter out = response.getWriter();

        out.println("<!DOCTYPE html>");
        out.println("<html>");
        out.println("<head>");
        out.println("<title>Servlet Request Demo</title>");
        out.println("</head>");
        out.println("<body>");

        out.println("<h1>Servlet Request Information</h1>");

        out.println("<p><b>HTTP Method:</b> "
                + method + "</p>");

        out.println("<p><b>Context Path:</b> "
                + contextPath + "</p>");

        out.println("<p><b>Servlet Path:</b> "
                + servletPath + "</p>");

        out.println("<p><b>Path Info:</b> "
                + (pathInfo == null ? "null" : pathInfo)
                + "</p>");

        out.println("<p><b>Query String:</b> "
                + (queryString == null ? "null" : queryString)
                + "</p>");

        out.println("<p><b>Request URI:</b> "
                + requestURI + "</p>");

        out.println("<h2>Individual Parameters</h2>");

        out.println("<p><b>Name:</b> "
                + (name == null ? "Not provided" : name)
                + "</p>");

        out.println("<p><b>Semester:</b> "
                + (semester == null
                    ? "Not provided"
                    : semester)
                + "</p>");

        out.println("</body>");
        out.println("</html>");
    }
}

```

### Deep explanation of the servlet

1\. `@WebServlet("/student/*")`

```
@WebServlet("/student/*")
```

This tells Tomcat which URL pattern should invoke this servlet.

Examples that match include:

```
/student
/student/profile
/student/profile/101
/student/profile/101?name=Ali
```

The context path is not included in the annotation mapping.

2\. `doGet()`

```
protected void doGet(
        HttpServletRequest request,
        HttpServletResponse response)
```

When the browser sends an HTTP GET request to this servlet, Tomcat calls `doGet()`.

- `request` contains information about the incoming request.
- `response` lets the servlet construct the outgoing response.

3\. Read the path

```
String servletPath = request.getServletPath();
String pathInfo = request.getPathInfo();
```

These methods return different parts of the path because our servlet mapping has a fixed prefix and a wildcard suffix.

4\. Read the query

```
String queryString = request.getQueryString();
String name = request.getParameter("name");
```

The first reads the raw query string; the second reads a named parameter.

5\. Write the response

```
PrintWriter out = response.getWriter();
out.println("<h1>Servlet Request Information</h1>");
```

This generates HTML that the browser renders as a web page.

For simplicity, this demonstration prints raw values directly. In production, escape user-provided values before inserting them into HTML to prevent cross-site scripting (XSS).

## Step 3. Create `index.html`

Location:

`src/main/webapp/index.html`

```

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Servlet Request Demo</title>
</head>
<body>

    <h1>Servlet Request Demo</h1>

    <p>Click below to inspect a servlet request.</p>

    <a href="student/profile/101?name=Ali&semester=3">
        View Student 101
    </a>

</body>
</html>

```

This page gives students a clickable link to generate a request with both path information and query parameters.

# 9. Run the application end to end

1. Install JDK 17, Maven, and Apache Tomcat 10.1.
2. Open the `RequestDemo` project in IntelliJ IDEA or Eclipse as a Maven project.
3. Open a terminal in the project root and run:

   `bash     mvn clean package     `
4. After a successful build, find `RequestDemo.war` inside the `target` directory.
5. Copy `target/RequestDemo.war` into Tomcat's `webapps` directory and start Tomcat. If it is already running, make sure WAR deployment is enabled.
6. Open the application in your browser:

   `text     http://localhost:8080/RequestDemo/     `
7. Click View Student 101 to inspect the request information.

# 10. Expected output

Open this URL directly:

```
http://localhost:8080/RequestDemo/student/profile/101?name=Ali&semester=3
```

The page will display approximately:

## Servlet Request Information

HTTP Method

GET

Context Path

/RequestDemo

Servlet Path

/student

Path Info

/profile/101

Query String

name=Ali&semester=3

Request URI

/RequestDemo/student/profile/101

### Individual Parameters

Name

Ali

Semester

3

The context path depends on the deployed application name. The servlet path and path info depend on the URL mapping and requested path.

# 11. Experiment with different URLs

Use these links to help students understand how the returned values change.

## Request URL explorer

Change the path and query string to predict the servlet output.

Choose a URL

/student

/student/profile/...

/student/details/...

Student ID in path

Include `?name=Ali&semester=3`

Full URL

http\://localhost:8080/RequestDemo/student/profile/101?name=Ali&semester=3

getServletPath()

/student

getPathInfo()

/profile/101

getQueryString()

name=Ali&semester=3

getParameter("name")

Ali

Predicted values for this project's `/student/*` mapping, not a live Tomcat request.

Notice that when the URL has no extra path after `/student`, `getPathInfo()` returns `null`. When there is no query string, `getQueryString()` returns `null`.

# 12. More useful request methods

| Method                 | What it returns                | Example                                                 |
| ---------------------- | ------------------------------ | ------------------------------------------------------- |
| `getMethod()`          | HTTP method                    | `GET`                                                   |
| `getContextPath()`     | Application path               | `/RequestDemo`                                          |
| `getServletPath()`     | Mapped servlet path            | `/student`                                              |
| `getPathInfo()`        | Extra path                     | `/profile/101`                                          |
| `getQueryString()`     | Raw query                      | `name=Ali`                                              |
| `getParameter("name")` | Individual parameter           | `Ali`                                                   |
| `getRequestURI()`      | Request path including context | `/RequestDemo/student/profile/101`                      |
| `getRequestURL()`      | URL without query string       | `http://localhost:8080/RequestDemo/student/profile/101` |

These methods are documented in the Jakarta Servlet API and Tomcat's `HttpServletRequest` reference.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://tomcat.apache.org\&sz=32)

Apache Tomcat 10.1.59)

+1



# 13. Common mistakes

Mistake 1: Expecting `getPathInfo()` to return the entire path

For `/student/profile/101`, with `/student/*` mapped to the servlet:

```
request.getServletPath(); // /student
request.getPathInfo();    // /profile/101
```

Mistake 2: Expecting `getQueryString()` to return just the name

```
request.getQueryString();       // name=Ali&semester=3
request.getParameter("name");   // Ali
```

Mistake 3: Using `javax.servlet` with Tomcat 10.1

Use:

```
import jakarta.servlet.http.HttpServletRequest;
```

Mistake 4: Including the context path in `@WebServlet`

Correct:

```
@WebServlet("/student/*")
```

Not:

```
@WebServlet("/RequestDemo/student/*")
```

The context path is supplied by the deployed application, not the servlet mapping.

Mistake 5: Thinking that `getPathInfo()` always returns a string

It can return `null`; check before using operations such as `.equals()` or `.split()`.

# 14. Practice questions 

1. What is the difference between `ServletRequest` and `HttpServletRequest`?
2. What is the role of `@WebServlet("/student/*")`?
3. What does `getServletPath()` return for `/student/profile/101`?
4. What does `getPathInfo()` return for `/student/profile/101`?
5. What happens to `getPathInfo()` when the requested path is just `/student`?
6. What is the difference between `getQueryString()` and `getParameter()`?
7. What does `getRequestURI()` include that `getServletPath()` does not?
8. Why do we use `jakarta.servlet` rather than `javax.servlet` in Tomcat 10.1?
9. What happens if the URL has no query string?
10. Modify the servlet to display an `id` query parameter using `request.getParameter("id")`.

