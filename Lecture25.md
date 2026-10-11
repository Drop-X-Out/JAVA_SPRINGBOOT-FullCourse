# Servlet Application Scope, ServletResponse, and HttpServletResponse

---

# Part A — Application Scope in Servlet

## 1. What does application scope mean?

Imagine you are building a college website with several Servlets:

- `HomeServlet`
- `StudentServlet`
- `CourseServlet`

Suppose all three Servlets need to know the institute name:

```text
PW Institute of Innovation
```

One option is to write the same value in every Servlet. But that duplicates data.

Instead, we can store a value in **application scope** so that different components of the same web application can access it.

In Servlet applications, application scope is represented by the **`ServletContext`** object.

```java
ServletContext context = getServletContext();
context.setAttribute("instituteName", "PW Institute of Innovation");
```

Another Servlet can retrieve it:

```java
String instituteName =
        (String) getServletContext().getAttribute("instituteName");
```

### Simple definition

**Application scope** stores data that can be shared by components within the same web application, for as long as that application context remains active and the data is retained.

The data is not automatically shared with every web application on the server.

---

## 2. Understand the four common scopes

| Scope | Main object | How long data is available | Typical use |
|---|---|---|---|
| Request | `HttpServletRequest` | During the current request/dispatch | Passing data to a JSP |
| Session | `HttpSession` | Across requests in one session | User-specific information |
| Application | `ServletContext` | Shared across the web application while it is active | Shared configuration or application-wide data |
| Page (JSP) | JSP page context | During the current JSP page execution | Temporary JSP variables |

For this lesson, focus on application scope.

### Remember

```java
request.setAttribute("key", value);           // Request scope
request.getSession().setAttribute("key", value); // Session scope
getServletContext().setAttribute("key", value);  // Application scope
```

The object on which `setAttribute()` is called determines the scope.

---

## 3. What is `ServletContext`?

`ServletContext` represents the web application as managed by the Servlet container.

Think of it as a shared application-level storage area and interface for application-wide information.

You can obtain it from a Servlet using:

```java
ServletContext context = getServletContext();
```

Or from a request:

```java
ServletContext context = request.getServletContext();
```

Both refer to the `ServletContext` for the web application handling that request.

Import:

```java
import jakarta.servlet.ServletContext;
```

### Common methods

| Method | Purpose |
|---|---|
| `setAttribute(name, value)` | Store a shared application attribute |
| `getAttribute(name)` | Read an application attribute |
| `removeAttribute(name)` | Remove an application attribute |
| `getAttributeNames()` | Get the names of current attributes |
| `getContextPath()` | Get the application's context path |
| `getInitParameter(name)` | Read an application initialization parameter |

Application attributes are suitable for shared objects or data, but concurrent requests may access them simultaneously. If you store mutable objects such as a list or map, you must consider thread safety.

---

## 4. Full project structure

```text
ApplicationScopeDemo/
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/
        │       └── example/
        │           └── scope/
        │               ├── AppStartupListener.java
        │               ├── SetApplicationServlet.java
        │               └── ReadApplicationServlet.java
        └── webapp/
            ├── index.html
            └── WEB-INF/
                └── web.xml
```

We will build an example where:

1. The application listener sets a shared value when the application starts.
2. One Servlet updates the application value.
3. Another Servlet reads the value.
4. The browser can make separate requests and still see the same application attribute.

---

## 5. File 1 — `pom.xml`

Path: `ApplicationScopeDemo/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>ApplicationScopeDemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
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
</project>
```

### Understand the important parts

- `packaging` is `war` because this is a web application.
- `jakarta.servlet-api` gives us Servlet classes and interfaces at compile time.
- `provided` means Tomcat supplies the Servlet API at runtime.
- This configuration is for Tomcat 10.1. Tomcat 9 uses the older `javax.servlet` namespace.

---

## 6. File 2 — `AppStartupListener.java`

Path:

`src/main/java/com/example/scope/AppStartupListener.java`

```java
package com.example.scope;

import jakarta.servlet.ServletContext;
import jakarta.servlet.ServletContextEvent;
import jakarta.servlet.ServletContextListener;
import jakarta.servlet.annotation.WebListener;

@WebListener
public class AppStartupListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent event) {

        // Get the ServletContext for this web application.
        ServletContext context = event.getServletContext();

        // Create an application-wide attribute.
        context.setAttribute(
                "instituteName",
                "PW Institute of Innovation"
        );

        System.out.println("Application started.");
    }

    @Override
    public void contextDestroyed(ServletContextEvent event) {
        System.out.println("Application is shutting down.");
    }
}
```

### Line-by-line explanation

**`implements ServletContextListener`**

This class listens for lifecycle events related to the web application's `ServletContext`.

**`@WebListener`**

This annotation tells the Servlet container to register the listener.

**`contextInitialized(...)`**

The container calls this method when the web application's context is initialized.

**`event.getServletContext()`**

Gets the `ServletContext` associated with this application event.

**`context.setAttribute("instituteName", "...")`**

Stores a shared attribute under the key `instituteName`.

**`contextDestroyed(...)`**

The container calls this when the application context is being destroyed. This example only prints a message.

### Why initialize the value here?

It means the shared value is prepared when the application starts, rather than waiting for a user to visit a particular Servlet first.

---

## 7. File 3 — `SetApplicationServlet.java`

Path:

`src/main/java/com/example/scope/SetApplicationServlet.java`

```java
package com.example.scope;

import java.io.IOException;

import jakarta.servlet.ServletContext;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/set-application")
public class SetApplicationServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {

        // Get the shared application context.
        ServletContext context = getServletContext();

        // Update an application-wide attribute.
        context.setAttribute("instituteName", "PW IOI Lucknow Campus");

        response.setContentType("text/plain;charset=UTF-8");
        response.getWriter().println("Application attribute updated.");
    }
}
```

### Important lines

- `@WebServlet("/set-application")` maps the Servlet to this URL.
- `getServletContext()` returns the application context shared by components of this web application.
- `setAttribute(...)` stores or replaces the value under that key.
- `setContentType(...)` tells the browser to interpret the response as plain text using UTF-8.
- `getWriter().println(...)` writes a text message into the response body.

**Important:** Application attributes are shared. Updating this value changes what other requests see, so do not use this technique for personal data such as one user's private username.

---

## 8. File 4 — `ReadApplicationServlet.java`

Path:

`src/main/java/com/example/scope/ReadApplicationServlet.java`

```java
package com.example.scope;

import java.io.IOException;

import jakarta.servlet.ServletContext;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/read-application")
public class ReadApplicationServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {

        // Get the same application's ServletContext.
        ServletContext context = getServletContext();

        // Read the shared attribute.
        String instituteName =
                (String) context.getAttribute("instituteName");

        response.setContentType("text/html;charset=UTF-8");

        var out = response.getWriter();

        out.println("<!DOCTYPE html>");
        out.println("<html lang='en'>");
        out.println("<head><title>Application Scope</title></head>");
        out.println("<body>");
        out.println("<h1>Application Scope Demo</h1>");

        if (instituteName != null) {
            out.println("<p>Institute: "
                    + escapeHtml(instituteName) + "</p>");
        } else {
            out.println("<p>No institute name has been configured.</p>");
        }

        out.println("<p><a href='"
                + request.getContextPath()
                + "/set-application'>Update shared value</a></p>");

        out.println("</body></html>");
    }

    private String escapeHtml(String value) {
        return value.replace("&", "&amp;")
                    .replace("<", "&lt;")
                    .replace(">", "&gt;")
                    .replace("\"", "&quot;")
                    .replace("'", "&#x27;");
    }
}
```

### Understand the main steps

1. `getServletContext()` obtains the application-wide context.
2. `getAttribute("instituteName")` reads the value stored by the listener or updated by the other Servlet.
3. The result is cast to `String` because `getAttribute()` returns an `Object`.
4. We check for `null` because an attribute may not have been set or may have been removed.
5. We escape HTML before inserting the text into the page.

The two Servlets use the same application context, so the second Servlet can read the value written by the first.

---

## 9. File 5 — `index.html`

Path: `src/main/webapp/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Application Scope Demo</title>
</head>
<body>
    <h1>Application Scope Demo</h1>

    <p>
        <a href="read-application">Read application attribute</a>
    </p>

    <p>
        <a href="set-application">Update application attribute</a>
    </p>
</body>
</html>
```

---

## 10. File 6 — `web.xml`

Path: `src/main/webapp/WEB-INF/web.xml`

The `@WebServlet` and `@WebListener` annotations already register the components, so this descriptor can remain minimal:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee
         https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
         version="6.0">
</web-app>
```

---

## 11. Run and test the application

Deploy on Tomcat 10.1. Assuming the context path is `/ApplicationScopeDemo`, open:

```text
http://localhost:8080/ApplicationScopeDemo/
```

Test these URLs:

```text
http://localhost:8080/ApplicationScopeDemo/read-application
http://localhost:8080/ApplicationScopeDemo/set-application
http://localhost:8080/ApplicationScopeDemo/read-application
```

Expected behavior:

1. The first read shows `PW Institute of Innovation`.
2. The update URL changes the value to `PW IOI Lucknow Campus`.
3. The next read shows `PW IOI Lucknow Campus`.

The value is shared between requests because it is stored in application scope.

### Application-scope flow

```text
Application starts
        |
        v
AppStartupListener
        |
        v
ServletContext.setAttribute(...)
        |
        v
+-----------------------------+
| Shared application context  |
+-----------------------------+
        ^              ^
        |              |
 SetApplication    ReadApplication
   Servlet             Servlet
        |              |
        v              v
 Update value       Read value
```

---

## 12. Important application-scope cautions

- Application attributes are shared by requests handled by the same web application.
- They are not automatically shared across different applications deployed on the same Tomcat.
- They normally do not survive application restarts unless you deliberately persist the data elsewhere.
- Multiple requests can run concurrently. Avoid unsynchronized read-modify-write operations on shared mutable objects.
- Do not store passwords or private per-user information in application scope.
- If storing a counter, a plain `Integer` attribute with `count = count + 1` is not a thread-safe counter. Use an appropriate concurrency mechanism, such as `AtomicInteger`, if a shared counter is genuinely required.

---

# Part B — ServletResponse and HttpServletResponse

## 13. First understand the response

When a browser requests a page, the Servlet receives a request and creates a response.

```text
Browser
   |
   | HTTP Request
   v
Servlet
   |
   | HTTP Response
   v
Browser
```

The response can contain:

1. **Status code** — for example, `200`, `404`, or `500`.
2. **Headers** — for example, content type or cache instructions.
3. **Body** — for example, HTML, plain text, JSON, or an image.

For example, a response might conceptually look like this:

```http
HTTP/1.1 200 OK
Content-Type: text/plain;charset=UTF-8

Hello from Servlet!
```

The exact protocol version and formatting depend on the server.

---

## 14. What is `ServletResponse`?

`ServletResponse` is an interface in the Servlet API. It represents the response that a Servlet sends back to the client.

It provides general response operations, including:

- Setting the response content type.
- Setting the character encoding.
- Obtaining a writer for character data.
- Obtaining an output stream for binary data.
- Buffering and flushing the response.

Import:

```java
import jakarta.servlet.ServletResponse;
```

A generic Servlet may use:

```java
public void service(ServletRequest request,
                    ServletResponse response)
```

### Example

```java
response.setContentType("text/plain;charset=UTF-8");
response.getWriter().println("Hello!");
```

Here:

- `setContentType(...)` describes the type of response body.
- `getWriter()` gives a writer for text.
- `println(...)` writes the text into the response body.

`ServletResponse` is general-purpose. It does not itself provide all HTTP-specific features, such as setting an HTTP status code or sending an HTTP redirect.

---

## 15. What is `HttpServletResponse`?

`HttpServletResponse` is an interface that extends `ServletResponse` and provides functionality specific to HTTP.

Import:

```java
import jakarta.servlet.http.HttpServletResponse;
```

It provides everything inherited from `ServletResponse`, plus HTTP operations such as:

- `setStatus(...)`
- `sendError(...)`
- `sendRedirect(...)`
- `setHeader(...)`
- `addHeader(...)`
- `setDateHeader(...)`
- `addCookie(...)`

When writing a class that extends `HttpServlet`, the HTTP handler methods typically look like this:

```java
protected void doGet(HttpServletRequest request,
                     HttpServletResponse response)
        throws IOException {
    // Handle the request and build the HTTP response.
}
```

The container passes the response object to the method.

---

## 16. Relationship between the two

```text
ServletResponse (interface)
        |
        | extended by
        v
HttpServletResponse (interface)
```

The relationship is:

```java
HttpServletResponse extends ServletResponse
```

So an `HttpServletResponse` supports the general response operations as well as HTTP-specific operations.

### Quick comparison

| Feature | `ServletResponse` | `HttpServletResponse` |
|---|---|---|
| Type | Interface | Interface |
| General content type | Yes | Yes, inherited |
| Character encoding | Yes | Yes, inherited |
| `getWriter()` | Yes | Yes, inherited |
| `getOutputStream()` | Yes | Yes, inherited |
| Set HTTP status code | Not in the generic interface | Yes |
| Send HTTP error | No HTTP-specific method | Yes |
| Send HTTP redirect | No HTTP-specific method | Yes |
| Set HTTP headers | No HTTP-specific methods | Yes |

In an `HttpServlet`, you normally work directly with `HttpServletResponse`.

---

## 17. Full response-demo project structure

```text
ResponseDemo/
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/
        │       └── example/
        │           └── response/
        │               └── ResponseDemoServlet.java
        └── webapp/
            ├── index.html
            └── WEB-INF/
                └── web.xml
```

Use the same `pom.xml` structure from Part A, but set:

```xml
<artifactId>ResponseDemo</artifactId>
```

Keep the same Servlet API dependency for Tomcat 10.1.

---

## 18. File 1 — `ResponseDemoServlet.java`

Path:

`src/main/java/com/example/response/ResponseDemoServlet.java`

```java
package com.example.response;

import java.io.IOException;

import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/response-demo")
public class ResponseDemoServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {

        // 1. Set the response body type and encoding.
        response.setContentType("text/html;charset=UTF-8");

        // 2. Set a custom response header.
        response.setHeader("X-Lesson", "Servlet Response Demo");

        // 3. Set the HTTP status code.
        response.setStatus(HttpServletResponse.SC_OK);

        // 4. Get a writer to send character data.
        var out = response.getWriter();

        // 5. Write HTML into the response body.
        out.println("<!DOCTYPE html>");
        out.println("<html lang='en'>");
        out.println("<head><meta charset='UTF-8'>");
        out.println("<title>Response Demo</title></head>");
        out.println("<body>");
        out.println("<h1>Hello from HttpServletResponse!</h1>");
        out.println("<p>Status code: 200 OK</p>");
        out.println("<p>Content type: text/html</p>");
        out.println("</body></html>");
    }
}
```

### Line-by-line explanation

**`response.setContentType("text/html;charset=UTF-8")`**

Tells the browser that the body is HTML and uses UTF-8 character encoding. Set the encoding before obtaining the writer or committing the response.

**`response.setHeader("X-Lesson", "...")`**

Sets an HTTP response header. `X-Lesson` is a custom header for this teaching example; it is not a built-in Servlet header.

**`response.setStatus(HttpServletResponse.SC_OK)`**

Sets the status code to 200. `SC_OK` is a readable constant for the HTTP 200 status.

**`response.getWriter()`**

Obtains a `PrintWriter` for writing character data into the response body.

**`out.println(...)`**

Writes HTML text. The browser interprets it as HTML because we set the content type to `text/html`.

---

## 19. File 2 — `index.html`

Path: `src/main/webapp/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Response Demo</title>
</head>
<body>
    <h1>Servlet Response Examples</h1>

    <ul>
        <li><a href="response-demo">HTML response</a></li>
        <li><a href="plain-text">Plain-text response</a></li>
        <li><a href="json-response">JSON response</a></li>
        <li><a href="not-found">Send a 404 error</a></li>
        <li><a href="redirect">Redirect example</a></li>
    </ul>
</body>
</html>
```

The links below demonstrate common response types. Add the additional mappings shown in the next sections to use them.

---

## 20. Example: send plain text

Add this method to a Servlet class or create a separate Servlet:

```java
@WebServlet("/plain-text")
public class PlainTextServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {

        response.setContentType("text/plain;charset=UTF-8");

        response.getWriter().println("Hello!");
        response.getWriter().println("This is plain text.");
    }
}
```

### What does `text/plain` mean?

It tells the browser to treat the body as plain text instead of interpreting it as HTML.

For example:

```text
Hello!
This is plain text.
```

Compare:

```java
response.setContentType("text/html;charset=UTF-8");
```

with:

```java
response.setContentType("text/plain;charset=UTF-8");
```

The content type tells the client how the response body should be interpreted.

---

## 21. Example: send JSON

JSON is commonly used when a Servlet sends data to JavaScript or another client.

```java
@WebServlet("/json-response")
public class JsonResponseServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {

        response.setContentType("application/json;charset=UTF-8");

        response.getWriter().println(
                "{\"name\":\"John\",\"course\":\"Servlet\"}"
        );
    }
}
```

Response body:

```json
{
  "name": "John",
  "course": "Servlet"
}
```

In this example, the JSON is written as a Java string, so quotation marks inside the JSON are escaped with backslashes. For real applications, use a JSON library when building JSON from dynamic data rather than manually concatenating strings.

---

## 22. Example: set an HTTP status code

A status code communicates the result of the request.

```java
response.setStatus(HttpServletResponse.SC_OK);
```

Common constants:

| Constant | Code | Meaning |
|---|---:|---|
| `SC_OK` | 200 | Successful request |
| `SC_CREATED` | 201 | Resource created |
| `SC_BAD_REQUEST` | 400 | Invalid request |
| `SC_UNAUTHORIZED` | 401 | Authentication required or failed |
| `SC_FORBIDDEN` | 403 | Access forbidden |
| `SC_NOT_FOUND` | 404 | Resource not found |
| `SC_INTERNAL_SERVER_ERROR` | 500 | Server error |

Example:

```java
@WebServlet("/not-found")
public class NotFoundServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {

        response.sendError(
                HttpServletResponse.SC_NOT_FOUND,
                "The requested resource was not found."
        );
    }
}
```

### Difference between `setStatus()` and `sendError()`

- `setStatus(404)` sets the status code, but does not itself generate the standard error response or trigger configured error-page processing in the same way.
- `sendError(404, "...")` sends an HTTP error response and allows the container's configured error handling to run.

If you have configured a matching error page in `web.xml`, `sendError()` is commonly used to invoke it.

Do not write a normal response body and then call `sendError()` after the response has already been committed.

---

## 23. Example: redirect the browser

A redirect tells the browser to make a new request to another URL.

```java
@WebServlet("/redirect")
public class RedirectServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {

        response.sendRedirect(
                request.getContextPath() + "/response-demo"
        );
    }
}
```

### What happens?

```text
Browser requests /redirect
          |
          v
RedirectServlet
          |
          v
sendRedirect(...)
          |
          v
Server returns redirect status and Location header
          |
          v
Browser requests /response-demo
```

The browser makes a new request. This differs from `RequestDispatcher.forward()`, which dispatches on the server using the existing request.

---

## 24. Example: add a response header

```java
response.setHeader("X-Lesson", "Servlet");
```

A response header is metadata sent with the response.

Other useful examples:

```java
response.setHeader("Cache-Control", "no-store");
response.setHeader("X-Content-Type-Options", "nosniff");
```

Headers must generally be set before the response is committed. Avoid setting security headers manually without understanding the requirements of the application and its deployment environment.

To add another value for a header rather than replace its existing value, use `addHeader()` where appropriate.

---

## 25. `getWriter()` vs `getOutputStream()`

These methods are both inherited from `ServletResponse`, but serve different purposes.

### For character data

```java
response.setContentType("text/plain;charset=UTF-8");
response.getWriter().println("Hello");
```

Use `getWriter()` for text such as HTML, plain text, or JSON.

### For binary data

```java
response.setContentType("application/octet-stream");

byte[] bytes = {65, 66, 67};

response.getOutputStream().write(bytes);
```

This writes three bytes, corresponding to `A`, `B`, and `C`.

Use `getOutputStream()` for binary content such as images, PDFs, or downloadable files.

**Important:** Do not use both `getWriter()` and `getOutputStream()` to write the body of the same response. The Servlet API does not allow switching between them for the same response.

---

## 26. `setContentType()` vs `setCharacterEncoding()`

These methods are related but different.

```java
response.setContentType("text/html");
response.setCharacterEncoding("UTF-8");
```

- `setContentType()` specifies the media type, such as `text/html`.
- `setCharacterEncoding()` specifies the character encoding for character data.

You can combine them:

```java
response.setContentType("text/html;charset=UTF-8");
```

Set the content type and encoding before calling `getWriter()` whenever possible.

---

## 27. What does “committed response” mean?

The container may buffer some response data before sending it to the client. When the response is committed, the status and headers have been sent and cannot normally be changed.

For example, this is too late if the response has already been committed:

```java
response.getWriter().println("Some output");

// The response may now be committed.
response.sendRedirect("/another-page");
```

The redirect may fail with an `IllegalStateException` if the response is already committed.

**Rule:** Decide the status, headers, content type, and whether to redirect or send an error before writing a large response body.

---

## 28. `ServletResponse` in a generic Servlet

So far, we used `HttpServlet`, whose methods receive `HttpServletRequest` and `HttpServletResponse`.

For learning the generic interfaces, here is a small `GenericServlet` example:

```java
package com.example.response;

import java.io.IOException;

import jakarta.servlet.GenericServlet;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;

@WebServlet("/generic-response")
public class GenericResponseServlet extends GenericServlet {

    @Override
    public void service(ServletRequest request,
                        ServletResponse response)
            throws ServletException, IOException {

        response.setContentType("text/plain;charset=UTF-8");
        response.getWriter().println("Hello from GenericServlet!");
    }
}
```

### What is different here?

- `GenericServlet` is a protocol-independent base class.
- Its `service()` method receives generic `ServletRequest` and `ServletResponse` interfaces.
- `HttpServlet` is designed specifically for HTTP and provides `doGet()`, `doPost()`, and other HTTP-specific methods.
- `HttpServletResponse` provides HTTP status, redirect, error, cookie, and header functionality that the generic `ServletResponse` interface does not define.

---

## 29. Common beginner mistakes

### Mistake 1: Writing HTML but setting plain-text content type

```java
response.setContentType("text/plain");
response.getWriter().println("<h1>Hello</h1>");
```

The browser may show the literal tags rather than rendering a heading.

For HTML, use:

```java
response.setContentType("text/html;charset=UTF-8");
```

### Mistake 2: Forgetting the content type

Set the response content type deliberately so the client can interpret the body correctly.

### Mistake 3: Calling `sendRedirect()` after writing the response

A committed response cannot be redirected normally. Perform redirects before writing the body.

### Mistake 4: Confusing `setStatus()` and `sendError()`

`setStatus()` sets the status code. `sendError()` initiates container error handling for an HTTP error.

### Mistake 5: Mixing `getWriter()` and `getOutputStream()`

Choose the appropriate output method for the response body: writer for characters, output stream for bytes.

### Mistake 6: Mixing Jakarta and Javax

For Tomcat 10.1, use:

```java
import jakarta.servlet.*;
```

For Tomcat 9, use:

```java
import javax.servlet.*;
```

Use the matching Servlet API dependency and container.

---

## 30. Practice questions

1. What is application scope?
2. Which interface represents application scope?
3. What is the difference between `ServletContext` and `HttpSession`?
4. Can two Servlets in the same application read the same application attribute?
5. Is application scope automatically shared across separate deployed applications?
6. What is `ServletResponse`?
7. What does `HttpServletResponse` add to `ServletResponse`?
8. What is the purpose of `setContentType()`?
9. What is the difference between `getWriter()` and `getOutputStream()`?
10. What is the difference between `setStatus(404)` and `sendError(404)`?
11. What does `sendRedirect()` do?
12. Why should headers be set before the response is committed?

### Coding exercises

- Store the college name in application scope and read it from a second Servlet.
- Add a shared application attribute called `academicYear`.
- Create a Servlet that returns plain text.
- Create a Servlet that returns JSON.
- Create a Servlet that sends a 404 error.
- Create a Servlet that redirects to another Servlet.
- Create a Servlet that writes a small byte array using `getOutputStream()`.

---

# Final revision

## Application scope

```java
ServletContext context = getServletContext();

context.setAttribute("college", "PW IOI");

String college = (String) context.getAttribute("college");
```

**Meaning:** share an attribute across components of the same web application.

## Generic response

```java
response.setContentType("text/plain;charset=UTF-8");
response.getWriter().println("Hello");
```

**Meaning:** define the body type and write text into the response.

## HTTP-specific response

```java
response.setStatus(HttpServletResponse.SC_OK);
response.sendRedirect("/somewhere");
response.sendError(HttpServletResponse.SC_NOT_FOUND);
```

**Meaning:** control HTTP status codes, redirects, and error responses.

## The core distinction

- `ServletContext` → application-wide data and application context.
- `ServletResponse` → general response body, encoding, and output.
- `HttpServletResponse` → all of the above response capabilities plus HTTP-specific controls.
