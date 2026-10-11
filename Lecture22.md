# Servlet Technology: GenericServlet and HttpServlet

**Technologies:** Java 17 · Maven · Apache Tomcat 10.1 · Jakarta Servlet 6.0 · IntelliJ IDEA · Smart Tomcat

## 1. Our Learning Journey

We have already learned JDBC, Hibernate, and Maven. Now we are learning how the frontend communicates with the Java server.

### Part 1: What we learned earlier

```text
Java Application
       |
       v
  JDBC / Hibernate
       |
       v
    Database
```

### Part 2: What we are learning now

```text
Browser / Frontend
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

### Part 3: What we will build in future lectures

```text
Frontend
   |
   v
Servlet
   |
   v
Hibernate / JDBC
   |
   v
Database
```

**Important:** We will first understand the frontend-to-server connection. We will connect Hibernate and the database in future lectures.

---

## 2. What Is a Servlet?

A Servlet is a Java component that runs inside a servlet container and processes requests from clients.

For example, when a browser opens a URL, the server receives the request and can execute Java code to generate a response.

A servlet can:

- Receive client requests.
- Process information.
- Execute business logic.
- Generate responses.
- Communicate with other Java components.

**Deep one-liner:** A servlet is a server-side Java component that processes client requests and generates responses through a servlet container.

There are two important servlet classes we will learn today:

1. `GenericServlet`
2. `HttpServlet`

## 3. What Is GenericServlet?

`GenericServlet` is an abstract class provided by the Servlet API. It provides a protocol-independent foundation for writing servlets.

It implements the `Servlet` interface and provides default implementations of several lifecycle and configuration methods.

When we extend `GenericServlet`, we generally implement its abstract `service()` method.

### Deep one-liner

> GenericServlet provides a general servlet foundation in which the developer implements `service()` to process requests without relying on HTTP-specific convenience methods.

### Basic example

```java
package servlet;

import jakarta.servlet.GenericServlet;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.ServletException;

import java.io.IOException;

public class MyGenericServlet extends GenericServlet {

    @Override
    public void service(
            ServletRequest request,
            ServletResponse response
    ) throws ServletException, IOException {

        response.setContentType("text/plain;charset=UTF-8");

        response.getWriter().println(
            "Hello from GenericServlet!"
        );
    }
}
```

### Explanation of every important line

**1. Package declaration**

```java
package servlet;
```

This places our class inside the `servlet` package. All our Java servlet classes will use this package.

**2. Import GenericServlet**

```java
import jakarta.servlet.GenericServlet;
```

This imports the abstract class that our servlet will extend.

**3. Import request and response**

```java
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
```

- `ServletRequest` represents the incoming request.
- `ServletResponse` represents the outgoing response.

These interfaces are not restricted to HTTP.

**4. Class declaration**

```java
public class MyGenericServlet extends GenericServlet
```

The `extends` keyword allows our class to inherit the functionality provided by `GenericServlet`.

**5. Override service**

```java
public void service(
        ServletRequest request,
        ServletResponse response
) throws ServletException, IOException
```

The container calls `service()` to let our servlet process the request.

`ServletException` represents a servlet-related processing error, while `IOException` represents an input/output error.

**6. Set the content type**

```java
response.setContentType("text/plain;charset=UTF-8");
```

This tells the browser that the response contains plain text encoded using UTF-8.

**7. Write the response**

```java
response.getWriter().println(
    "Hello from GenericServlet!"
);
```

This sends text back to the client.

### Remember

`GenericServlet` does not provide HTTP-specific methods such as `doGet()` and `doPost()`. It gives us a general `service()` method.

---

## 4. What Is HttpServlet?

`HttpServlet` is an abstract class designed specifically for HTTP-based web applications.

It extends `GenericServlet` and provides HTTP-specific request handling.

Instead of implementing the general `service()` method ourselves, we commonly override methods such as:

- `doGet()` — handles GET requests.
- `doPost()` — handles POST requests.
- `doPut()` — handles PUT requests.
- `doDelete()` — handles DELETE requests.

### Deep one-liner

> HttpServlet specializes GenericServlet for HTTP and dispatches requests to methods such as doGet() and doPost() according to the HTTP method.

### Basic example

```java
package servlet;

import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/http")
public class MyHttpServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws IOException {

        response.setContentType("text/plain;charset=UTF-8");

        response.getWriter().println(
            "Hello from HttpServlet!"
        );
    }
}
```

### Explanation of every important line

**1. Import HttpServlet**

```java
import jakarta.servlet.http.HttpServlet;
```

This imports the base class designed for HTTP servlet processing.

**2. Import HTTP request and response**

```java
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
```

These interfaces provide HTTP-specific functionality.

For example, `HttpServletRequest` lets us read query parameters, request headers, and the HTTP method.

**3. Map the URL**

```java
@WebServlet("/http")
```

This maps requests for `/http` within the deployed web application to `MyHttpServlet`.

**4. Extend HttpServlet**

```java
public class MyHttpServlet extends HttpServlet
```

Our class inherits HTTP request handling from `HttpServlet`.

**5. Override doGet**

```java
protected void doGet(
        HttpServletRequest request,
        HttpServletResponse response
) throws IOException
```

The container's HTTP dispatching mechanism invokes `doGet()` for a GET request matching this servlet.

**6. Configure the response**

```java
response.setContentType("text/plain;charset=UTF-8");
```

This specifies a plain-text response with UTF-8 encoding.

**7. Return the message**

```java
response.getWriter().println(
    "Hello from HttpServlet!"
);
```

The browser receives and displays the returned text.

---

## 5. Difference Between GenericServlet and HttpServlet

| GenericServlet | HttpServlet |
|---|---|
| General, protocol-independent servlet base class | HTTP-specific servlet base class |
| Implements `Servlet` and provides general servlet functionality | Extends `GenericServlet` |
| Commonly requires overriding `service()` | Commonly requires overriding `doGet()` or `doPost()` |
| Uses `ServletRequest` and `ServletResponse` | Uses `HttpServletRequest` and `HttpServletResponse` |
| Does not provide HTTP-specific convenience methods | Provides HTTP-specific request handling |
| Useful for understanding general servlet processing | Commonly used for web applications |

### Inheritance relationship

```text
Servlet Interface
       |
       v
GenericServlet
       |
       v
HttpServlet
```

### Deep one-liner

> GenericServlet provides general request processing, whereas HttpServlet adds HTTP-specific request dispatching and is generally the preferred base class for HTTP web applications.

---

## 6. Create the Maven Web Application in IntelliJ IDEA

Now let's create a working project that contains both servlet types.

### Step 1: Create a project

1. Open IntelliJ IDEA.
2. Select **New Project**.
3. Choose Maven as the build system.
4. Select JDK 17.
5. Set the project name to `servlet-demo`.

### Step 2: Enter the project coordinates

Use these values:

```text
Group ID:    com.example
Artifact ID: servlet-demo
Version:     1.0-SNAPSHOT
Packaging:   war
```

- `Group ID` identifies the project group.
- `Artifact ID` identifies our application.
- `Version` identifies the current project version.
- `Packaging` specifies that Maven should build a web application archive.

### Step 3: Understand the Maven archetype

If IntelliJ provides a Maven archetype selection screen, select:

```text
Group ID:    org.apache.maven.archetypes
Artifact ID: maven-archetype-webapp
```

This is the standard Maven Web Application archetype.

If the archetype is unavailable, create a normal Maven project and use the complete structure and configuration provided below.

**Important:** The archetype coordinates are for the template, not for our application's own Group ID and Artifact ID.

---

## 7. Complete Project Folder Structure

We will use only one Java package named `servlet`.

```text
servlet-demo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │   └── servlet/
        │       ├── MyGenericServlet.java
        │       └── MyHttpServlet.java
        │
        └── webapp/
            └── index.html
```

There is no need to create a `web.xml` file for these servlet mappings because we are using `@WebServlet`.

Maven and the configured Tomcat deployment will make the compiled Java classes and web resources available to the servlet container.

---

## 8. File 1 — `pom.xml`

Create or replace `pom.xml` in the project root.

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

### Why do we need this configuration?

- `packaging=war` makes Maven package the web application as a WAR.
- `maven.compiler.release=17` selects Java 17 for compilation.
- `jakarta.servlet-api` provides the classes and interfaces needed to compile our servlets.
- `scope=provided` means the servlet container supplies the API at runtime.
- `finalName=servlet-demo` gives the generated WAR a predictable name.

**Important:** We are using Tomcat 10.1, so the code imports `jakarta.servlet.*`, not `javax.servlet.*`.

### Do we need a separate Tomcat dependency in Maven?

No. We install Tomcat separately and configure Smart Tomcat to use that installation. We do not need to add the Tomcat server itself as an application dependency.

---

## 9. File 2 — `MyGenericServlet.java`

Location:

`src/main/java/servlet/MyGenericServlet.java`

```java
package servlet;

import jakarta.servlet.GenericServlet;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;

import java.io.IOException;

@WebServlet("/generic")
public class MyGenericServlet extends GenericServlet {

    @Override
    public void service(
            ServletRequest request,
            ServletResponse response
    ) throws ServletException, IOException {

        response.setContentType("text/html;charset=UTF-8");

        response.getWriter().println("""
            <!DOCTYPE html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <title>GenericServlet Demo</title>
            </head>
            <body>
                <h1>Hello from GenericServlet!</h1>
                <p>This response was generated by a Java servlet.</p>
            </body>
            </html>
            """);
    }
}
```

### Explanation

**`package servlet;`**

This places the class inside our single Java package.

**`extends GenericServlet`**

This inherits the general servlet functionality.

**`@WebServlet("/generic")`**

This maps the servlet to the `/generic` path within our application.

**`service()`**

The container calls this method to process the request.

**`ServletRequest` and `ServletResponse`**

These provide general request and response interfaces without requiring HTTP-specific types.

**`setContentType()`**

This declares that the response contains HTML encoded using UTF-8.

**`getWriter().println()`**

This writes the HTML response that the browser renders.

The Java text block, written using triple quotation marks, makes multiline HTML easier to read.

---

## 10. File 3 — `MyHttpServlet.java`

Location:

`src/main/java/servlet/MyHttpServlet.java`

```java
package servlet;

import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/http")
public class MyHttpServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws IOException {

        response.setContentType("text/html;charset=UTF-8");

        response.getWriter().println("""
            <!DOCTYPE html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <title>HttpServlet Demo</title>
            </head>
            <body>
                <h1>Hello from HttpServlet!</h1>
                <p>This response was generated by doGet().</p>
            </body>
            </html>
            """);
    }
}
```

### Explanation

**`extends HttpServlet`**

This class inherits HTTP-specific servlet functionality.

**`@WebServlet("/http")`**

This maps requests to `/http` to our servlet.

**`doGet()`**

This method processes GET requests dispatched by `HttpServlet`.

**`HttpServletRequest`**

This provides access to HTTP request information. In a future lesson, we will use it to receive data from HTML forms.

**`HttpServletResponse`**

This lets us construct the HTTP response.

**`getWriter()`**

This writes the generated HTML back to the browser.

### Why are we using a different method here?

With `GenericServlet`, we implement `service()` directly.

With `HttpServlet`, we commonly override `doGet()` or `doPost()` and let the inherited HTTP dispatching mechanism choose the appropriate method.

---

## 11. File 4 — `index.html`

Location:

`src/main/webapp/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Servlet Demo</title>
</head>
<body>

    <h1>My First Servlet Application</h1>

    <p>Select a servlet to test:</p>

    <ul>
        <li>
            <a href="generic">Open GenericServlet</a>
        </li>

        <li>
            <a href="http">Open HttpServlet</a>
        </li>
    </ul>

</body>
</html>
```

### Explanation

**`<!DOCTYPE html>`**

Declares an HTML5 document.

**`<html lang="en">`**

Defines the HTML document and its language.

**`<meta charset="UTF-8">`**

Specifies the character encoding.

**`<meta name="viewport">`**

Helps the page display appropriately on different screen sizes.

**`<title>`**

Defines the title shown in the browser tab.

**`<a href="generic">`**

Creates a link to the `generic` path relative to the current application.

**`<a href="http">`**

Creates a link to the `http` path relative to the current application.

Because these are relative links, the application context path is retained automatically when the browser follows them from the application root.

This page is static HTML. The response pages generated by our two servlets are dynamic server responses.

---

## 12. Configure Smart Tomcat in IntelliJ IDEA

Smart Tomcat lets us launch our application using the Tomcat installation configured in IntelliJ IDEA.

Plugin page:

https://plugins.jetbrains.com/plugin/9492-smart-tomcat

### Step 1: Install the plugin

1. Open IntelliJ IDEA.
2. Go to **Settings → Plugins**.
3. Search for `Smart Tomcat`.
4. Install it.
5. Restart IntelliJ IDEA if requested.

### Step 2: Prepare Tomcat

Download and extract Tomcat 10.1:

https://tomcat.apache.org/download-10.cgi

Ensure JDK 17 is installed and working.

### Step 3: Create a Smart Tomcat run configuration

Open **Run → Edit Configurations** and add a Smart Tomcat configuration.

Configure the available fields, including:

- Tomcat installation directory.
- Project module or deployment directory.
- Context path.
- Port, if the configuration exposes it.

For this lesson, use `/servlet-demo` as the context path if you want the URLs shown in the examples below.

**Deployment note:** Smart Tomcat's configuration fields vary by plugin version. The selected deployment directory must expose the web resources and compiled classes in a valid web application layout. If the plugin uses a deployment directory directly, do not point it only at `src/main/java`.

### Step 4: Build the web application

From the project root, run:

```bash
mvn clean package war:exploded
```

This compiles the project, packages the WAR, and creates an exploded web application directory.

The relevant output should include:

```text
target/
├── servlet-demo.war
└── servlet-demo/
    ├── index.html
    └── WEB-INF/
        └── classes/
            └── servlet/
                ├── MyGenericServlet.class
                └── MyHttpServlet.class
```

The exploded directory is the expanded form of the web application. It contains the web resources and compiled classes in locations that a servlet container can use.

### Step 5: Configure the deployment directory

If your Smart Tomcat configuration asks for a deployment directory, use the generated exploded web application directory:

```text
target/servlet-demo
```

If your plugin version instead manages deployment using the selected module and its classpath, follow that configuration and ensure the compiled classes are available to Tomcat.

### Step 6: Run the application

Click **Run** in IntelliJ IDEA.

Check the Run console to verify that Tomcat starts successfully.

You do not need to manually copy the WAR into Tomcat's `webapps` directory for this configured development workflow.

If you modify the code, rebuild or refresh the deployed application as required by your configuration before testing the changes.

---

## 13. Test All Three URLs

Assuming the context path is `/servlet-demo` and Tomcat is listening on port `8080`, test the following URLs.

### URL 1: Application home page

```text
http://localhost:8080/servlet-demo/
```

Expected result: The static `index.html` page with two links.

### URL 2: GenericServlet

```text
http://localhost:8080/servlet-demo/generic
```

Expected result:

**Hello from GenericServlet!**

The request is handled by `MyGenericServlet.service()`.

### URL 3: HttpServlet

```text
http://localhost:8080/servlet-demo/http
```

Expected result:

**Hello from HttpServlet!**

The request is handled by `MyHttpServlet.doGet()`.

If you configure a different context path or port, update the URLs accordingly.

---

## 14. Understand the Complete Request Flow

When you open the home page, Tomcat serves the HTML resource.

When you click **Open GenericServlet**, the browser sends a GET request to `/generic`.

Tomcat matches that URL to `MyGenericServlet` and invokes its `service()` method.

When you click **Open HttpServlet**, the browser sends a GET request to `/http`.

Tomcat matches that URL to `MyHttpServlet`. The HTTP servlet dispatching mechanism calls its `doGet()` method.

### GenericServlet request flow

```text
Browser
   |
   | GET /servlet-demo/generic
   v
Tomcat
   |
   | URL mapping: /generic
   v
MyGenericServlet
   |
   | service()
   v
HTML Response
   |
   v
Browser
```

### HttpServlet request flow

```text
Browser
   |
   | GET /servlet-demo/http
   v
Tomcat
   |
   | URL mapping: /http
   v
MyHttpServlet
   |
   | doGet()
   v
HTML Response
   |
   v
Browser
```

### Deep one-liner

> Tomcat matches the requested URL to a servlet mapping, invokes the appropriate request-handling method, and returns the servlet's response to the browser.

---

## 15. Common Errors

### Error 1: `package jakarta.servlet does not exist`

**Cause:** The servlet API dependency is missing or Maven has not reloaded the project.

**Solution:** Verify `pom.xml`, reload Maven, and run the build again.

### Error 2: `404 Not Found`

**Cause:** The context path, servlet mapping, or deployment configuration is incorrect.

**Solution:** Verify the URL, `@WebServlet` annotation, and deployed application structure.

### Error 3: `Address already in use`

**Cause:** Another process is using the selected port.

**Solution:** Stop the conflicting process or configure another available port.

### Error 4: `javax.servlet` imports fail

**Cause:** Code from an older Java EE tutorial is being used with Tomcat 10.1.

**Solution:** Use `jakarta.servlet` imports and compatible Jakarta Servlet dependencies.

### Error 5: Home page works but servlet routes return 404

**Cause:** The HTML resources are deployed, but the compiled classes may not be available in `WEB-INF/classes`, or annotation scanning/deployment is not configured correctly.

**Solution:** Verify the exploded application structure and Smart Tomcat deployment configuration, then rebuild and redeploy.

---

