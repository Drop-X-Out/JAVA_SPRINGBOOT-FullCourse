# ServletRequest vs HttpServletRequest — getRequestURI() and getContextPath()

## 1. Learning Objectives

By the end of this lecture, students will understand:

- What `ServletRequest` is.
- What `HttpServletRequest` is.
- The difference between `ServletRequest` and `HttpServletRequest`.
- Why `getRequestURI()` and `getContextPath()` belong to `HttpServletRequest`.
- How to read the requested URL path inside a servlet.
- How to identify the context path of a web application.
- How `getServletPath()` and `getPathInfo()` differ from `getRequestURI()`.
- How to run a complete Maven web application using IntelliJ IDEA and Smart Tomcat.

## 2. Revision: Where Are We in Our Learning Journey?

We have already learned JDBC, Hibernate, and Maven.

Previously, we learned how Java communicates with a database. Now, we are learning how a browser communicates with a Java server.

```text
ALREADY LEARNED

Java Application
       |
       v
  JDBC / Hibernate
       |
       v
    Database
```

```text
LEARNING NOW

Browser
   |
   | HTTP Request
   v
Tomcat
   |
   v
Servlet
   |
   | HTTP Response
   v
Browser
```

In this lecture, we will understand how a servlet identifies information about the request received from the browser.

**Deep one-liner:** `ServletRequest` provides general request information, while `HttpServletRequest` provides additional information specific to HTTP requests.

---

## 3. What Is ServletRequest?

`ServletRequest` is an interface in the Jakarta Servlet API that represents a request received by a servlet.

It provides methods to access general request information, including:

- Request parameters
- Input data
- Character encoding
- Content type
- Request attributes
- The request's input stream

For example:

```java
String name = request.getParameter("name");
```

This reads a request parameter named `name`.

### Basic example

```java
ServletRequest request;
```

This declares a reference whose type is `ServletRequest`.

However, we cannot use this reference to call HTTP-specific methods such as `getRequestURI()` or `getContextPath()`.

For example, this will not compile:

```java
request.getRequestURI();
```

Why?

Because `getRequestURI()` is not declared in the `ServletRequest` interface.

**Deep one-liner:** `ServletRequest` provides protocol-independent request functionality but does not expose HTTP-specific URL methods.

---

## 4. What Is HttpServletRequest?

`HttpServletRequest` is an interface that extends `ServletRequest` and provides additional functionality for HTTP requests.

It allows us to access information such as:

- HTTP request method
- Request URI
- Context path
- Servlet path
- Path information
- Query string
- Request headers
- Cookies
- Session information

For example:

```java
String uri = request.getRequestURI();
String contextPath = request.getContextPath();
```

These methods are available when the request reference is typed as `HttpServletRequest`.

### Relationship between the interfaces

```text
ServletRequest
      |
      | Extended by
      v
HttpServletRequest
```

This means `HttpServletRequest` inherits the general request functionality of `ServletRequest` and adds HTTP-specific methods.

### Deep one-liner

> HttpServletRequest extends ServletRequest with HTTP-specific information, including the request URI, context path, headers, cookies, and HTTP method.

---

## 5. Difference Between ServletRequest and HttpServletRequest

| ServletRequest | HttpServletRequest |
|---|---|
| General request interface | HTTP-specific request interface |
| Does not depend on HTTP-specific functionality | Designed for HTTP requests |
| Provides `getParameter()` | Also provides `getParameter()` |
| Does not provide `getRequestURI()` | Provides `getRequestURI()` |
| Does not provide `getContextPath()` | Provides `getContextPath()` |
| Does not provide `getMethod()` | Provides `getMethod()` |
| Used as the request type in `GenericServlet.service()` | Used as the request type in `HttpServlet.doGet()` and `doPost()` |

### Important concept

Consider these two methods:

```java
public void service(
        ServletRequest request,
        ServletResponse response
)
```

```java
protected void doGet(
        HttpServletRequest request,
        HttpServletResponse response
)
```

In the first method, the request is declared using the general `ServletRequest` interface.

In the second method, the request is declared using the HTTP-specific `HttpServletRequest` interface.

Therefore, the second method can directly call:

```java
request.getRequestURI();
request.getContextPath();
```

The first method cannot call those methods directly without obtaining an HTTP-specific reference.

---

## 6. What Is getRequestURI()?

The `getRequestURI()` method returns the URI identifying the requested resource, including the application's context path and the path within the application.

It does **not** include the query string.

### Syntax

```java
String uri = request.getRequestURI();
```

### Example URL

Suppose the browser requests:

```text
http://localhost:8080/servlet-demo/request-info/student/profile?name=Rahul
```

The returned URI is:

```text
/servlet-demo/request-info/student/profile
```

Notice that the URI contains:

- `/servlet-demo`
- `/request-info`
- `/student/profile`

It does not contain:

- `http://`
- `localhost`
- `8080`
- `?name=Rahul`

### Why is the query string excluded?

The query string is separate information in the URL. You can retrieve it using:

```java
String query = request.getQueryString();
```

For our example, the result is:

```text
name=Rahul
```

### Deep one-liner

> `getRequestURI()` returns the requested path, including the context path, but excludes the scheme, host, port, and query string.

---

## 7. What Is getContextPath()?

The `getContextPath()` method returns the context path of the web application handling the request.

The context path identifies the deployed web application within the server.

### Syntax

```java
String contextPath = request.getContextPath();
```

Suppose our application is deployed as:

```text
/servlet-demo
```

Then:

```java
request.getContextPath();
```

returns:

```text
/servlet-demo
```

### Example

Requested URL:

```text
http://localhost:8080/servlet-demo/request-info/student/profile?name=Rahul
```

Context path:

```text
/servlet-demo
```

The context path is not the entire URI. It is only the application prefix.

### What if the application is deployed at the root?

If the web application is deployed at the root context, the context path is an empty string:

```java
request.getContextPath(); // ""
```

It does not return `/`.

### Deep one-liner

> `getContextPath()` identifies the web application handling the request, independently of the particular servlet or route being requested.

---

## 8. Understand the Complete URL

Let's break down this URL:

```text
http://localhost:8080/servlet-demo/request-info/student/profile?name=Rahul
```

| Part | Value | Meaning |
|---|---|---|
| Scheme | `http` | Communication protocol |
| Host | `localhost` | Computer running the server |
| Port | `8080` | Server port |
| Context path | `/servlet-demo` | Web application path |
| Servlet mapping | `/request-info` | URL mapping of our servlet |
| Path info | `/student/profile` | Additional path matched by the mapping |
| Query string | `name=Rahul` | Query parameter information |

For this URL, the expected method results are:

| Method | Expected result |
|---|---|
| `getRequestURI()` | `/servlet-demo/request-info/student/profile` |
| `getContextPath()` | `/servlet-demo` |
| `getServletPath()` | `/request-info` |
| `getPathInfo()` | `/student/profile` |
| `getQueryString()` | `name=Rahul` |
| `getMethod()` | `GET` |

These results assume the application is deployed under `/servlet-demo` and the servlet mapping is `/request-info/*`.

**Remember:** The exact path values depend on the URL mapping and deployment configuration.

---

## 9. Complete Project Folder Structure

We will build a working Maven web application using IntelliJ IDEA and Smart Tomcat.

All Java classes will be inside one package named `servlet`.

```text
servlet-demo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │   └── servlet/
        │       ├── RequestInfoServlet.java
        │       └── GenericRequestServlet.java
        │
        └── webapp/
            └── index.html
```

### Purpose of each file

| File | Purpose |
|---|---|
| `pom.xml` | Maven project configuration and Servlet API dependency |
| `RequestInfoServlet.java` | Demonstrates HTTP request URI and context path |
| `GenericRequestServlet.java` | Demonstrates the difference between `ServletRequest` and `HttpServletRequest` |
| `index.html` | Provides links to test both servlets |

We will use annotation-based URL mappings, so we do not need a `web.xml` file for this example.

---

## 10. File 1: pom.xml

Location:

`servlet-demo/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>servlet-demo</artifactId>
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
        <finalName>servlet-demo</finalName>

        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.4.0</version>
            </plugin>
        </plugins>
    </build>

</project>
```

### Explanation

**Group ID**

```xml
<groupId>com.example</groupId>
```

Identifies the project group.

**Artifact ID**

```xml
<artifactId>servlet-demo</artifactId>
```

Identifies our web application.

**Packaging**

```xml
<packaging>war</packaging>
```

Tells Maven to package the application as a web application archive.

**Java version**

```xml
<maven.compiler.release>17</maven.compiler.release>
```

Configures Java 17 compilation.

**Servlet API dependency**

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.0.0</version>
    <scope>provided</scope>
</dependency>
```

This provides the interfaces and classes needed to compile our servlet code.

The `provided` scope means the runtime container supplies the API. Tomcat 10.1 provides a compatible Jakarta Servlet 6.0 implementation.

---

## 11. File 2: RequestInfoServlet.java

Location:

`src/main/java/servlet/RequestInfoServlet.java`

This is our main servlet. It demonstrates `getRequestURI()`, `getContextPath()`, and related methods.

### Complete code

```java
package servlet;

import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/request-info/*")
public class RequestInfoServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws IOException {

        response.setContentType("text/html;charset=UTF-8");

        String requestURI = request.getRequestURI();
        String contextPath = request.getContextPath();
        String servletPath = request.getServletPath();
        String pathInfo = request.getPathInfo();
        String queryString = request.getQueryString();
        String method = request.getMethod();

        response.getWriter().println("""
            <!DOCTYPE html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <title>Request Information</title>
            </head>
            <body>
                <h1>HTTP Request Information</h1>

                <p><strong>Request URI:</strong> %s</p>
                <p><strong>Context Path:</strong> %s</p>
                <p><strong>Servlet Path:</strong> %s</p>
                <p><strong>Path Info:</strong> %s</p>
                <p><strong>Query String:</strong> %s</p>
                <p><strong>HTTP Method:</strong> %s</p>

                <a href="%s/">Back to Home</a>
            </body>
            </html>
            """.formatted(
                escapeHtml(requestURI),
                escapeHtml(contextPath),
                escapeHtml(servletPath),
                escapeHtml(pathInfo),
                escapeHtml(queryString),
                escapeHtml(method),
                escapeHtml(contextPath)
            ));
    }

    private String escapeHtml(String value) {
        if (value == null) {
            return "(null)";
        }

        return value
                .replace("&", "&amp;")
                .replace("<", "&lt;")
                .replace(">", "&gt;")
                .replace("\"", "&quot;")
                .replace("'", "&#39;");
    }
}
```

### Line-by-line explanation

**1. Package declaration**

```java
package servlet;
```

Places the class in the `servlet` package.

**2. Import `WebServlet`**

```java
import jakarta.servlet.annotation.WebServlet;
```

Allows us to declare the servlet's URL mapping using an annotation.

**3. Import `HttpServlet`**

```java
import jakarta.servlet.http.HttpServlet;
```

Provides the base class for HTTP servlet implementations.

**4. Import request and response**

```java
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
```

These interfaces provide access to HTTP request information and the outgoing HTTP response.

**5. URL mapping**

```java
@WebServlet("/request-info/*")
```

Maps the servlet to `/request-info` and paths beneath that mapping.

The `/*` suffix is a path mapping. It allows us to test an additional path, such as `/student/profile`.

For example:

```text
/servlet-demo/request-info/student/profile
```

**6. Class declaration**

```java
public class RequestInfoServlet extends HttpServlet
```

Defines our HTTP servlet.

**7. Override `doGet()`**

```java
protected void doGet(
        HttpServletRequest request,
        HttpServletResponse response
) throws IOException
```

This method handles GET requests dispatched by the servlet container.

**8. Configure the response**

```java
response.setContentType("text/html;charset=UTF-8");
```

Tells the browser that the response contains HTML using UTF-8 encoding.

**9. Read the request URI**

```java
String requestURI = request.getRequestURI();
```

Gets the requested URI path, including the application context path but excluding the query string.

**10. Read the context path**

```java
String contextPath = request.getContextPath();
```

Gets the path identifying the deployed web application.

**11. Read the servlet path**

```java
String servletPath = request.getServletPath();
```

Gets the portion of the path that matched the servlet mapping, excluding the context path and path info.

For our mapping `/request-info/*`, this is normally `/request-info`.

**12. Read the path info**

```java
String pathInfo = request.getPathInfo();
```

Gets the extra path information after the servlet mapping. For `/request-info/student/profile`, it is `/student/profile`.

It can be `null` when there is no additional path information.

**13. Read the query string**

```java
String queryString = request.getQueryString();
```

Gets the raw query string, excluding the question mark.

For `?name=Rahul`, it returns `name=Rahul`.

It can be `null` when there is no query string.

**14. Read the HTTP method**

```java
String method = request.getMethod();
```

Returns the HTTP method, such as `GET` or `POST`.

**15. Write the response**

```java
response.getWriter().println(...);
```

Writes the HTML response to the browser.

**16. Escape displayed values**

```java
escapeHtml(requestURI)
```

The helper method escapes HTML-sensitive characters before displaying request-derived values in HTML. This is a useful security practice when displaying data that might contain untrusted input.

---

## 12. File 3: GenericRequestServlet.java

Location:

`src/main/java/servlet/GenericRequestServlet.java`

This file demonstrates how `ServletRequest` differs from `HttpServletRequest`.

### Complete code

```java
package servlet;

import jakarta.servlet.GenericServlet;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServletRequest;

import java.io.IOException;

@WebServlet("/generic-info")
public class GenericRequestServlet extends GenericServlet {

    @Override
    public void service(
            ServletRequest request,
            ServletResponse response
    ) throws ServletException, IOException {

        response.setContentType("text/html;charset=UTF-8");

        StringBuilder output = new StringBuilder();

        output.append("<!DOCTYPE html>");
        output.append("<html lang='en'>");
        output.append("<head><meta charset='UTF-8'>");
        output.append("<title>Generic Request</title></head>");
        output.append("<body>");

        output.append("<h1>ServletRequest Example</h1>");

        output.append("<p>Request parameter name: ")
              .append(escapeHtml(request.getParameter("name")))
              .append("</p>");

        output.append("<p>Request object class: ")
              .append(escapeHtml(request.getClass().getName()))
              .append("</p>");

        if (request instanceof HttpServletRequest httpRequest) {

            output.append("<h2>HTTP-specific information</h2>");

            output.append("<p>Request URI: ")
                  .append(escapeHtml(httpRequest.getRequestURI()))
                  .append("</p>");

            output.append("<p>Context Path: ")
                  .append(escapeHtml(httpRequest.getContextPath()))
                  .append("</p>");

            output.append("<p>HTTP Method: ")
                  .append(escapeHtml(httpRequest.getMethod()))
                  .append("</p>");

        } else {

            output.append(
                "<p>This request is not an HTTP request.</p>"
            );
        }

        output.append("<a href='")
              .append(escapeHtml(request.getContextPathIfAvailable()))
              .append("'>Home</a>");

        output.append("</body></html>");

        response.getWriter().println(output);
    }

    private String escapeHtml(String value) {
        if (value == null) {
            return "(null)";
        }

        return value
                .replace("&", "&amp;")
                .replace("<", "&lt;")
                .replace(">", "&gt;")
                .replace("\"", "&quot;")
                .replace("'", "&#39;");
    }
}
```

**Correction before running:** `ServletRequest` does not have a `getContextPathIfAvailable()` method. We must not invent a method that is absent from the Servlet API. Replace the home-link section above with the following valid code:

```java
if (request instanceof HttpServletRequest httpRequest) {
    output.append("<a href='")
          .append(escapeHtml(httpRequest.getContextPath()))
          .append("/'>Home</a>");
}
```

Use this corrected version in the complete file below.

### Correct complete file

```java
package servlet;

import jakarta.servlet.GenericServlet;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServletRequest;

import java.io.IOException;

@WebServlet("/generic-info")
public class GenericRequestServlet extends GenericServlet {

    @Override
    public void service(
            ServletRequest request,
            ServletResponse response
    ) throws ServletException, IOException {

        response.setContentType("text/html;charset=UTF-8");

        StringBuilder output = new StringBuilder();

        output.append("<!DOCTYPE html>");
        output.append("<html lang='en'>");
        output.append("<head><meta charset='UTF-8'>");
        output.append("<title>Generic Request</title></head>");
        output.append("<body>");

        output.append("<h1>ServletRequest Example</h1>");

        output.append("<p>Request parameter name: ")
              .append(escapeHtml(request.getParameter("name")))
              .append("</p>");

        output.append("<p>Request object class: ")
              .append(escapeHtml(request.getClass().getName()))
              .append("</p>");

        if (request instanceof HttpServletRequest httpRequest) {

            output.append("<h2>HTTP-specific information</h2>");

            output.append("<p>Request URI: ")
                  .append(escapeHtml(httpRequest.getRequestURI()))
                  .append("</p>");

            output.append("<p>Context Path: ")
                  .append(escapeHtml(httpRequest.getContextPath()))
                  .append("</p>");

            output.append("<p>HTTP Method: ")
                  .append(escapeHtml(httpRequest.getMethod()))
                  .append("</p>");

            output.append("<a href='")
                  .append(escapeHtml(httpRequest.getContextPath()))
                  .append("/'>Home</a>");

        } else {
            output.append(
                "<p>This request is not an HTTP request.</p>"
            );
        }

        output.append("</body></html>");

        response.getWriter().println(output);
    }

    private String escapeHtml(String value) {
        if (value == null) {
            return "(null)";
        }

        return value
                .replace("&", "&amp;")
                .replace("<", "&lt;")
                .replace(">", "&gt;")
                .replace("\"", "&quot;")
                .replace("'", "&#39;");
    }
}
```

### Explanation

**1. Extend `GenericServlet`**

```java
public class GenericRequestServlet extends GenericServlet
```

This demonstrates the general servlet programming model.

**2. Implement `service()`**

```java
public void service(
        ServletRequest request,
        ServletResponse response
)
```

Notice that the parameter is declared as `ServletRequest`, not `HttpServletRequest`.

We can directly use methods such as:

```java
request.getParameter("name");
```

But we cannot directly call:

```java
request.getRequestURI();
```

That method is not declared by `ServletRequest`.

**3. Check the runtime type**

```java
if (request instanceof HttpServletRequest httpRequest)
```

This checks whether the request object also implements `HttpServletRequest`.

The pattern variable `httpRequest` is available inside the `if` block if the check succeeds.

Since this servlet is running in Tomcat and receiving HTTP requests, the request object normally implements `HttpServletRequest`.

**4. Call HTTP-specific methods**

```java
httpRequest.getRequestURI();
httpRequest.getContextPath();
httpRequest.getMethod();
```

We can call these methods because `httpRequest` is typed as `HttpServletRequest`.

**5. Why use `instanceof`?**

The check ensures that we only use HTTP-specific methods when the request actually supports the HTTP interface. This is safer than blindly casting a general request reference.

**6. Write the response**

```java
response.getWriter().println(output);
```

The `StringBuilder` constructs an HTML response, and the writer sends it to the browser.

**Deep one-liner:** A `ServletRequest` reference cannot directly access HTTP-specific methods, but an HTTP request can be safely checked and accessed through the `HttpServletRequest` interface.

---

## 13. File 4: index.html

Location:

`src/main/webapp/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Servlet Request Demo</title>
</head>
<body>

    <h1>Servlet Request Information</h1>

    <h2>HttpServletRequest</h2>

    <p>
        <a href="request-info/student/profile?name=Rahul">
            View Request URI and Context Path
        </a>
    </p>

    <h2>ServletRequest</h2>

    <p>
        <a href="generic-info?name=Rahul">
            View GenericServlet Request Information
        </a>
    </p>

</body>
</html>
```

### Explanation

**`href="request-info/student/profile?name=Rahul"`**

This relative URL requests our `RequestInfoServlet`.

The servlet mapping is `/request-info/*`, and the remaining path is `/student/profile`.

**`href="generic-info?name=Rahul"`**

This requests our `GenericRequestServlet` and passes the query parameter `name=Rahul`.

**Why use relative links?**

The links automatically retain the application's current context path when followed from the deployed application root.

---

## 14. Run the Project Using Smart Tomcat

### Step 1: Install Tomcat 10.1

Download Tomcat from the official website:

https://tomcat.apache.org/download-10.cgi

Extract the archive and ensure JDK 17 is installed.

### Step 2: Install Smart Tomcat

In IntelliJ IDEA:

1. Open Settings.
2. Select Plugins.
3. Search for `Smart Tomcat`.
4. Install the plugin.
5. Restart the IDE if required.

Plugin page:

https://plugins.jetbrains.com/plugin/9492-smart-tomcat

### Step 3: Configure the run configuration

Open **Run → Edit Configurations** and add a Smart Tomcat configuration.

Select:

- The Tomcat installation directory.
- The appropriate module or deployment directory.
- The application context path.
- The server port, if configurable.

For this example, use the context path:

```text
/servlet-demo
```

Smart Tomcat configuration fields vary by plugin version, so make sure the configured deployment contains the compiled classes and web resources.

### Step 4: Build the project

Run the following command from the project root:

```bash
mvn clean package war:exploded
```

This builds the application and generates the WAR and exploded web application output.

### Step 5: Run

Click **Run** in IntelliJ IDEA.

Once Tomcat starts, open the application root URL:

```text
http://localhost:8080/servlet-demo/
```

You should see the two links from `index.html`.

---

## 15. Test the Application

Assuming the application context path is `/servlet-demo` and Tomcat uses port `8080`, test these URLs.

### Test 1: Home page

```text
http://localhost:8080/servlet-demo/
```

Expected result: The page contains links to both servlet examples.

### Test 2: Request URI and context path

```text
http://localhost:8080/servlet-demo/request-info/student/profile?name=Rahul
```

Expected output:

```text
Request URI: /servlet-demo/request-info/student/profile
Context Path: /servlet-demo
Servlet Path: /request-info
Path Info: /student/profile
Query String: name=Rahul
HTTP Method: GET
```

### Test 3: GenericServlet example

```text
http://localhost:8080/servlet-demo/generic-info?name=Rahul
```

Expected result: The page displays the request parameter, request object's runtime class, request URI, context path, and HTTP method.

The runtime class name is container-specific, so its exact value can vary.

---

## 16. Common Mistakes

### Mistake 1: Calling `getRequestURI()` on `ServletRequest`

Incorrect:

```java
ServletRequest request = ...;
request.getRequestURI();
```

This does not compile because `ServletRequest` does not declare `getRequestURI()`.

Correct:

```java
if (request instanceof HttpServletRequest httpRequest) {
    String uri = httpRequest.getRequestURI();
}
```

### Mistake 2: Assuming `getRequestURI()` includes the query string

For:

```text
/servlet-demo/request-info?name=Rahul
```

The URI is:

```text
/servlet-demo/request-info
```

The query string is:

```text
name=Rahul
```

Use `getQueryString()` to read the raw query string.

### Mistake 3: Assuming the context path is always `/servlet-demo`

The context path depends on deployment configuration. It can be different, or empty when the application is deployed at the root.

### Mistake 4: Confusing the context path with the servlet mapping

For the URL:

```text
/servlet-demo/request-info/student/profile
```

The context path is `/servlet-demo`, and the servlet mapping is `/request-info/*`.

### Mistake 5: Mixing `javax.servlet` and `jakarta.servlet`

Tomcat 10.1 uses the Jakarta Servlet namespace.

Use:

```java
import jakarta.servlet.http.HttpServletRequest;
```

Do not mix it with older `javax.servlet` imports.

---

## 17. Practical Assignments

1. Change the query parameter to `name=John` and observe the output.
2. Open `/request-info/college/student` and identify `getRequestURI()`, `getContextPath()`, and `getPathInfo()`.
3. Open `/request-info` without extra path information and observe the value of `getPathInfo()`.
4. Create a servlet mapped to `/products/*`.
5. Display the URI and context path for multiple requests.
6. Explain why `getRequestURI()` cannot be called directly on a `ServletRequest` reference.
7. Deploy the application under a different context path and observe how the returned values change.

**Final takeaway:** `ServletRequest` provides general request information, while `HttpServletRequest` gives us access to HTTP-specific details. `getRequestURI()` tells us which URI was requested, and `getContextPath()` tells us which web application is handling the request.
