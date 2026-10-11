# XML vs Annotation Approach in Servlet Technology

## 1. Learning Objectives

By the end of this lecture, students will understand:

- What XML-based servlet configuration means.
- What annotation-based servlet configuration means.
- How `web.xml` registers and maps a servlet.
- How `@WebServlet` maps a servlet.
- The difference between the two approaches.
- How to run both approaches using Maven, IntelliJ IDEA, and Smart Tomcat.
- When to use XML configuration and when to use annotations.

## 2. Revision: What Have We Learned?

We have already learned JDBC, Hibernate, and Maven.

Previously, we focused on connecting Java applications to databases. Now we are learning how the frontend communicates with the Java server.

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

Browser / Frontend
       |
       v
Tomcat + Servlet
       |
       v
HTTP Response
       |
       v
Browser
```

In the previous lecture, we learned `GenericServlet` and `HttpServlet`. Now we will learn two ways to configure a servlet.

**Deep one-liner:** XML and annotations are two configuration approaches that tell the servlet container which servlet should handle a particular URL.

---

## 3. What Is Servlet Configuration?

Suppose we have created a servlet class named `HelloServlet`.

```java
package servlet;

import jakarta.servlet.http.HttpServlet;

public class HelloServlet extends HttpServlet {
}
```

The class exists, but the container still needs to know how to associate a URL with it.

For example, we want this URL to invoke our servlet:

```text
http://localhost:8080/servlet-demo/hello
```

We need to configure the servlet's name, class, and URL mapping.

There are two common approaches:

1. XML configuration using `web.xml`
2. Annotation configuration using `@WebServlet`

---

## 4. Approach One: XML Configuration

XML stands for **Extensible Markup Language**.

In traditional Java web applications, `web.xml` is the deployment descriptor used to configure servlets, URL mappings, filters, and other web application settings.

Instead of writing the URL mapping in the Java class, we define it in an XML file.

### Deep one-liner

> The XML approach configures servlet registration and URL mappings externally in the web application's `web.xml` deployment descriptor.

### Folder structure

```text
servlet-demo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │   └── servlet/
        │       └── HelloServlet.java
        │
        └── webapp/
            ├── index.html
            │
            └── WEB-INF/
                └── web.xml
```

The important file is:

```text
src/main/webapp/WEB-INF/web.xml
```

### File 1: `HelloServlet.java`

Location:

`src/main/java/servlet/HelloServlet.java`

```java
package servlet;

import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws IOException {

        response.setContentType("text/html;charset=UTF-8");

        response.getWriter().println(
            "<h1>Hello from XML Configuration!</h1>"
        );
    }
}
```

### Line-by-line explanation

**1. Package declaration**

```java
package servlet;
```

This places our Java class in the `servlet` package.

**2. Import `HttpServlet`**

```java
import jakarta.servlet.http.HttpServlet;
```

This provides the base class for HTTP servlet implementations.

**3. Import request and response**

```java
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
```

- `HttpServletRequest` provides information about the incoming HTTP request.
- `HttpServletResponse` allows us to generate the outgoing HTTP response.

**4. Extend `HttpServlet`**

```java
public class HelloServlet extends HttpServlet
```

Our class inherits HTTP request-handling functionality.

**5. Override `doGet()`**

```java
protected void doGet(
        HttpServletRequest request,
        HttpServletResponse response
) throws IOException
```

This method handles HTTP GET requests dispatched to the servlet.

**6. Set the response type**

```java
response.setContentType("text/html;charset=UTF-8");
```

This tells the browser to interpret the response as HTML using UTF-8 encoding.

**7. Send the response**

```java
response.getWriter().println(
    "<h1>Hello from XML Configuration!</h1>"
);
```

This writes HTML into the HTTP response.

**Important:** There is no `@WebServlet` annotation in this class. We will configure its URL mapping in `web.xml`.

### File 2: `web.xml`

Location:

`src/main/webapp/WEB-INF/web.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee
         https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
         version="6.0">

    <servlet>
        <servlet-name>HelloServlet</servlet-name>
        <servlet-class>servlet.HelloServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>HelloServlet</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>

</web-app>
```

This deployment descriptor uses the Jakarta Servlet 6.0 schema, compatible with Tomcat 10.1.

### Understand every important XML element

**1. XML declaration**

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

This identifies the XML version and character encoding.

**2. Root element**

```xml
<web-app ... version="6.0">
```

The `web-app` element contains web application deployment configuration.

The namespace and schema identify the Jakarta EE deployment descriptor format.

**3. Servlet registration**

```xml
<servlet>
    <servlet-name>HelloServlet</servlet-name>
    <servlet-class>servlet.HelloServlet</servlet-class>
</servlet>
```

This declares a servlet.

- `servlet-name` assigns a logical name to the servlet.
- `servlet-class` specifies the fully qualified Java class name.

Because our class belongs to the `servlet` package, its fully qualified name is `servlet.HelloServlet`.

**4. Servlet mapping**

```xml
<servlet-mapping>
    <servlet-name>HelloServlet</servlet-name>
    <url-pattern>/hello</url-pattern>
</servlet-mapping>
```

This associates the servlet's logical name with a URL pattern.

- `servlet-name` refers to the previously registered servlet.
- `url-pattern` defines the URL path that should invoke it.

**5. Closing the root element**

```xml
</web-app>
```

This completes the deployment descriptor.

### Request flow

```text
Browser
   |
   | GET /servlet-demo/hello
   v
Tomcat
   |
   v
web.xml
   |
   | Maps /hello
   v
servlet.HelloServlet
   |
   v
doGet()
   |
   v
HTTP Response
```

### Test the XML approach

Run the application using Smart Tomcat with the context path `/servlet-demo`.

Open:

`http://localhost:8080/servlet-demo/hello`

Expected output:

**Hello from XML Configuration!**

---

## 5. Approach Two: Annotation Configuration

Annotations allow us to configure servlet mappings directly in Java code.

Instead of writing the servlet's URL mapping in `web.xml`, we use the `@WebServlet` annotation.

### Deep one-liner

> The annotation approach declares servlet configuration directly in the Java class using annotations such as `@WebServlet`.

### Folder structure

```text
servlet-demo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │   └── servlet/
        │       └── HelloServlet.java
        │
        └── webapp/
            └── index.html
```

For this example, the servlet mapping is declared in the Java source file.

### File: `HelloServlet.java`

```java
package servlet;

import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws IOException {

        response.setContentType("text/html;charset=UTF-8");

        response.getWriter().println(
            "<h1>Hello from Annotation Configuration!</h1>"
        );
    }
}
```

### Line-by-line explanation

**1. Import the annotation**

```java
import jakarta.servlet.annotation.WebServlet;
```

This imports the annotation used to declare servlet configuration.

**2. Map the URL**

```java
@WebServlet("/hello")
```

This declares `/hello` as the servlet's URL mapping.

When the application is deployed under `/servlet-demo`, the corresponding URL is:

`http://localhost:8080/servlet-demo/hello`

**3. Declare the servlet class**

```java
public class HelloServlet extends HttpServlet
```

This defines a servlet that inherits HTTP request-handling functionality.

**4. Override `doGet()`**

```java
protected void doGet(
        HttpServletRequest request,
        HttpServletResponse response
) throws IOException
```

This method processes GET requests dispatched by `HttpServlet`.

**5. Generate the response**

```java
response.getWriter().println(
    "<h1>Hello from Annotation Configuration!</h1>"
);
```

This sends the HTML heading to the browser.

### Request flow

```text
Browser
   |
   | GET /servlet-demo/hello
   v
Tomcat
   |
   | Reads servlet annotation
   v
@WebServlet("/hello")
   |
   v
HelloServlet.doGet()
   |
   v
HTTP Response
```

### Test the annotation approach

Run the application using Smart Tomcat with the context path `/servlet-demo`.

Open:

`http://localhost:8080/servlet-demo/hello`

Expected output:

**Hello from Annotation Configuration!**

---

## 6. Difference Between XML and Annotation Approaches

| XML approach | Annotation approach |
|---|---|
| Uses `web.xml` for servlet configuration | Uses annotations in Java classes |
| Mapping is separate from Java source code | Mapping is declared directly in the Java class |
| Uses `<servlet>` and `<servlet-mapping>` | Uses `@WebServlet` |
| Configuration can be managed centrally | Configuration stays close to the servlet class |
| Useful for centralized or legacy configuration | Convenient for straightforward servlet mappings |
| Requires maintaining XML configuration | Requires adding the appropriate annotation |
| A deployment descriptor can override annotation settings | Annotation behavior depends on deployment descriptor configuration and container rules |

### Which approach should we use?

For our beginner servlet exercises, **the annotation approach is usually easier** because the servlet and its URL mapping are in one place.

The XML approach is still important because existing Java web applications often use `web.xml`, and deployment descriptors support centralized configuration.

Neither approach is universally better. The right choice depends on project requirements and existing configuration.

---

## 7. Can We Use Both XML and Annotations?

Yes. A servlet application can use XML configuration and annotations together, but we must understand how the container combines them.

For example, if `web.xml` maps a servlet to `/welcome` and an annotation maps the same servlet to `/hello`, the effective mappings depend on the configuration and the applicable Servlet specification rules.

Also, if the deployment descriptor sets `metadata-complete="true"`, annotation scanning for deployment metadata is disabled.

For beginner exercises, it is clearer to use one approach at a time.

**Rule for this practical:** When demonstrating XML configuration, remove `@WebServlet` from that servlet. When demonstrating annotation configuration, do not add a duplicate mapping for the same servlet in `web.xml`.

---

## 8. Configure Maven for Both Approaches

Use the following `pom.xml` for Tomcat 10.1, Jakarta Servlet 6.0, and Java 17.

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

