# Scopes in Servlet: Request Scope and Session Scope

> These notes use **Jakarta Servlet** (`jakarta.servlet.*`) and a Maven web application. They are written for first-time learners. If your class uses an older Tomcat version, see the compatibility note near the end.

---

## 1. What does “scope” mean?

Imagine a college website. A student enters their name, visits another page, and wants the website to remember that name.

A **scope** answers two questions:

1. **Where can we store information?**
2. **How long should that information remain available?**

In Servlet applications, scope is about how long data is available and which parts of the application can access it.

For this lesson, we will learn two scopes:

- **Request scope** — data belongs to one HTTP request.
- **Session scope** — data belongs to one user's session and can be reused across multiple requests.

### Simple real-life example

Imagine you send a parcel:

- **Request scope:** a note attached to one delivery. It is useful for that delivery only.
- **Session scope:** a customer card kept while a customer continues using a website. It can be used on several pages during their session.

---

## 2. First understand request and response

When a browser opens a URL, it sends an HTTP request to the server.

```text
Browser  ---- request ---->  Servlet
Browser  <--- response ----  Servlet
```

For example, the browser requests:

```text
http://localhost:8080/ScopeDemo/request-demo
```

The Servlet receives the request, performs some work, and sends a response.

Important objects:

- `HttpServletRequest request`: information about the incoming request; it can also hold request attributes.
- `HttpServletResponse response`: used to build the response sent back to the browser.

---

# Part A — Request Scope

## 3. What is request scope?

Request scope means:

> Store data for the current request so that another resource handling that same request can access it.

In Servlets, request-scope data is commonly stored as a **request attribute**:

```java
request.setAttribute("name", "Usama");
```

Read it with:

```java
String name = (String) request.getAttribute("name");
```

The name `"name"` is the attribute key. `"Usama"` is the value.

### What does `setAttribute()` do?

```java
request.setAttribute("studentName", "Ayesha");
```

Breakdown:

- `request`: the request object for the current request.
- `setAttribute`: stores an object as an attribute.
- `"studentName"`: the key used to find the value later.
- `"Ayesha"`: the value being stored.

### What does `getAttribute()` do?

```java
String studentName =
        (String) request.getAttribute("studentName");
```

- `getAttribute("studentName")` retrieves the value stored under that key.
- `(String)` is a cast because `getAttribute()` returns an `Object`.
- The value is assigned to the `studentName` variable.

Request attributes can hold Java objects, not only strings.

---

## 4. When should we use request scope?

Use request scope when data is needed only while processing one request, for example:

- Passing a form's result from a Servlet to a JSP.
- Showing a success or validation message.
- Passing a list of products to a JSP for the current page.
- Passing data to another Servlet using `RequestDispatcher.forward()`.

**Important:** Request scope does not automatically preserve data for a completely new browser request.

---

## 5. Request-scope example: full project

### File structure

```text
ScopeDemo/
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/
        │       └── example/
        │           └── scope/
        │               └── RequestScopeServlet.java
        └── webapp/
            ├── index.html
            └── WEB-INF/
                ├── web.xml
                └── views/
                    └── request-result.jsp
```

We will use **Smart Tomcat** or a regular Tomcat server to run this Maven web application. The application context path in the examples is `ScopeDemo`; your IDE or server may assign a different one.

### File 1: `pom.xml`

Create this file in the `ScopeDemo` project root:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>ScopeDemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- Tomcat 10.1 provides the Servlet API; Tomcat supplies it at runtime. -->
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>6.0.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

</project>
```

#### `pom.xml`, line by line

- `<packaging>war</packaging>` says this is a web application packaged as a WAR.
- The `jakarta.servlet-api` dependency provides the types needed to compile Servlet code.
- `<scope>provided</scope>` means the server supplies the Servlet API at runtime, so it should not be bundled as an application library.
- This setup is for **Tomcat 10.1 / Jakarta Servlet 6.0**. Do not use it unchanged with Tomcat 9.

### File 2: `src/main/webapp/index.html`

This page contains a simple HTML form:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Request Scope Demo</title>
</head>
<body>
    <h1>Request Scope Demo</h1>

    <form action="request-demo" method="post">
        <label for="studentName">Enter your name:</label>
        <input type="text" id="studentName" name="studentName" required>

        <button type="submit">Submit</button>
    </form>
</body>
</html>
```

#### Understand the important lines

- `<form>` groups the input and button.
- `action="request-demo"` sends the form to the URL mapping `request-demo`.
- `method="post"` sends the form data in an HTTP POST request.
- `name="studentName"` is important: the Servlet uses this exact name to read the submitted value.
- `required` asks the browser to prevent submission when the field is empty. Server-side validation is still recommended for real applications.

### File 3: `RequestScopeServlet.java`

Path:

```text
src/main/java/com/example/scope/RequestScopeServlet.java
```

Full code:

```java
package com.example.scope;

import java.io.IOException;

import jakarta.servlet.RequestDispatcher;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/request-demo")
public class RequestScopeServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request,
                          HttpServletResponse response)
            throws ServletException, IOException {

        // 1. Read the value submitted by the HTML form.
        String studentName = request.getParameter("studentName");

        // 2. Put the value into request scope.
        request.setAttribute("studentName", studentName);

        // 3. Find the JSP that will display the result.
        RequestDispatcher dispatcher =
                request.getRequestDispatcher("/WEB-INF/views/request-result.jsp");

        // 4. Forward the same request to the JSP.
        dispatcher.forward(request, response);
    }
}
```

#### Servlet code explained line by line

**`package com.example.scope;`**

This says the class belongs to the Java package `com.example.scope`. The folder path should match the package name.

**`import java.io.IOException;`**

The Servlet may encounter input/output problems while processing or responding to a request.

**`import jakarta.servlet.RequestDispatcher;`**

`RequestDispatcher` lets the server forward the current request to another server-side resource, such as a JSP.

**`import jakarta.servlet.ServletException;`**

This represents Servlet-related processing errors.

**`import jakarta.servlet.annotation.WebServlet;`**

This makes the `@WebServlet` annotation available.

**`import jakarta.servlet.http.HttpServlet;`**

`HttpServlet` is the base class for Servlets that handle HTTP requests.

**`import jakarta.servlet.http.HttpServletRequest;`**

This represents the incoming HTTP request.

**`import jakarta.servlet.http.HttpServletResponse;`**

This represents the HTTP response.

**`@WebServlet("/request-demo")`**

This maps the Servlet to the `/request-demo` URL inside the application.

**`public class RequestScopeServlet extends HttpServlet`**

We create a Servlet class by extending `HttpServlet`.

**`@Override`**

This tells Java that the next method overrides a method from the parent class.

**`protected void doPost(...)`**

The browser used `method="post"`, so the container calls `doPost()` for this form submission.

**`request.getParameter("studentName")`**

Reads the submitted form field named `studentName`. Form parameters are input values; they are different from request attributes.

**`request.setAttribute("studentName", studentName)`**

Stores the name as an attribute on this request. The JSP can read it while processing this same request.

**`request.getRequestDispatcher("/WEB-INF/views/request-result.jsp")`**

Finds the JSP resource to which we want to forward. A resource inside `WEB-INF` cannot normally be opened directly by typing its URL in the browser, but server-side code can forward to it.

**`dispatcher.forward(request, response)`**

Forwards the same request and response objects to the JSP. The request attribute remains available to the JSP. After forwarding, do not try to write a second independent response from the original Servlet.

### File 4: `request-result.jsp`

Path:

```text
src/main/webapp/WEB-INF/views/request-result.jsp
```

Full code:

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Request Scope Result</title>
</head>
<body>
    <h1>Hello!</h1>

    <p>
        Welcome,
        <strong><%= request.getAttribute("studentName") %></strong>.
    </p>

    <p>The name was stored in request scope and read by this JSP.</p>

    <a href="../../index.html">Go back to the form</a>
</body>
</html>
```

#### JSP explained

- The page directive sets the response content type and encoding.
- `request.getAttribute("studentName")` retrieves the attribute stored by the Servlet.
- `<%= ... %>` is a JSP expression that writes the result into the HTML response. It is shown here to teach the concept; for production applications, prefer JSTL/EL and escape user-provided output to prevent cross-site scripting (XSS).
- The link returns to the form.

### File 5: `web.xml`

Path:

```text
src/main/webapp/WEB-INF/web.xml
```

For this example, the Servlet is mapped using `@WebServlet`, so `web.xml` is optional. If you want to include it, use:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee
         https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
         version="6.0">
</web-app>
```

### Test the request-scope example

1. Start the application on Tomcat 10.1.
2. Open `http://localhost:8080/ScopeDemo/` (adjust the context path if yours differs).
3. Enter a name and click **Submit**.
4. The Servlet reads the form parameter, saves it as a request attribute, and forwards to the JSP.
5. The JSP displays the name.

### Request-scope flow

```text
Browser submits form
        |
        v
RequestScopeServlet.doPost()
        |
        v
request.getParameter("studentName")
        |
        v
request.setAttribute("studentName", studentName)
        |
        v
RequestDispatcher.forward(...)
        |
        v
request-result.jsp reads getAttribute("studentName")
        |
        v
HTML response sent to browser
```

### Does request scope survive a redirect?

Usually, **no**. A redirect tells the browser to make a new HTTP request. The new request has a different request object, so the old request attributes are not carried over automatically.

```java
response.sendRedirect("some-page");
```

Compare that with:

```java
request.getRequestDispatcher("/some-page.jsp")
       .forward(request, response);
```

A forward keeps the same request. A redirect starts a new request from the browser.

---

# Part B — Session Scope

## 6. What is session scope?

Session scope means:

> Store data associated with one user's session so it can be available across multiple requests during that session.

For example, a user logs in and visits several pages. The application may need to remember the user's ID between requests.

Servlets use `HttpSession` for this:

```java
HttpSession session = request.getSession();
session.setAttribute("username", "Usama");
```

To retrieve the value later:

```java
String username = (String) session.getAttribute("username");
```

### What is a session?

HTTP is normally **stateless**. This means each request is independent by default; the server does not automatically know that two requests came from the same browsing session.

A session gives the application a way to associate data with a series of requests from the same client. The container commonly uses a session ID, often carried in a cookie, to recognize the session.

```text
Request 1: user opens login page
Request 2: user logs in
Request 3: user opens profile page
Request 4: user opens dashboard
```

If the same session is maintained, session attributes can be available on requests 2, 3, and 4.

---

## 7. When should we use session scope?

Common examples:

- A logged-in user's ID or username.
- A shopping cart associated with a user's session.
- A temporary multi-step form workflow.
- A user's selected language or other session-level preference.

Do not put every piece of information into the session. Session data consumes server resources, can become stale, and should not contain passwords or other secrets unnecessarily.

---

## 8. Session-scope example: full project

We will add two Servlets:

1. `SetSessionServlet` — saves a name in the session.
2. `ReadSessionServlet` — reads the name in a separate request.

### File structure

```text
ScopeDemo/
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/
        │       └── example/
        │           └── scope/
        │               ├── RequestScopeServlet.java
        │               ├── SetSessionServlet.java
        │               └── ReadSessionServlet.java
        └── webapp/
            ├── index.html
            └── WEB-INF/
                ├── web.xml
                └── views/
                    └── request-result.jsp
```

The `pom.xml` and `web.xml` can remain the same as in Part A.

### File 1: Update `index.html`

Add a second form below the request-scope form:

```html
<hr>

<h2>Session Scope Demo</h2>

<form action="set-session" method="post">
    <label for="sessionName">Enter your name:</label>
    <input type="text" id="sessionName" name="sessionName" required>
    <button type="submit">Save in Session</button>
</form>

<p>
    <a href="read-session">Read saved session value</a>
</p>
```

This form submits the name to `/set-session`. The link later makes a separate GET request to `/read-session`.

### File 2: `SetSessionServlet.java`

Path:

```text
src/main/java/com/example/scope/SetSessionServlet.java
```

Full code:

```java
package com.example.scope;

import java.io.IOException;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

@WebServlet("/set-session")
public class SetSessionServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request,
                          HttpServletResponse response)
            throws ServletException, IOException {

        // 1. Read the name from the submitted form.
        String name = request.getParameter("sessionName");

        // 2. Get the current session, or create one if necessary.
        HttpSession session = request.getSession();

        // 3. Store the name in the session.
        session.setAttribute("username", name);

        // 4. Redirect the browser to a separate URL.
        response.sendRedirect(
                request.getContextPath() + "/read-session"
        );
    }
}
```

#### Line-by-line explanation

**`import jakarta.servlet.http.HttpSession;`**

Imports the type used to work with a user's session.

**`@WebServlet("/set-session")`**

Maps this Servlet to `/set-session`.

**`doPost(...)`**

The form uses POST, so the container calls `doPost()`.

**`request.getParameter("sessionName")`**

Reads the value submitted by the form. The HTML input's `name` must match `"sessionName"`.

**`request.getSession()`**

Returns the current session if one exists, or creates one if it does not. It may create a session even for a first-time visitor.

**`session.setAttribute("username", name)`**

Stores the name in the session under the key `"username"`. The value can be retrieved by later requests associated with the same session.

**`response.sendRedirect(...)`**

Sends a redirect response to the browser. The browser then makes a new request to `/read-session`. The request object is new, but the session can still be the same, so the session attribute remains available.

**`request.getContextPath()`**

Returns the application's context path, such as `/ScopeDemo`. Using it avoids hard-coding the application name into the redirect URL.

### File 3: `ReadSessionServlet.java`

Path:

```text
src/main/java/com/example/scope/ReadSessionServlet.java
```

Full code:

```java
package com.example.scope;

import java.io.IOException;

import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

@WebServlet("/read-session")
public class ReadSessionServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {

        // 1. Get an existing session without creating a new one.
        HttpSession session = request.getSession(false);

        // 2. Tell the browser that we are returning HTML.
        response.setContentType("text/html;charset=UTF-8");

        // 3. Get the response writer.
        var out = response.getWriter();

        // 4. Check whether a session and username are available.
        if (session != null && session.getAttribute("username") != null) {

            String username = (String) session.getAttribute("username");

            out.println("<!DOCTYPE html>");
            out.println("<html lang='en'>");
            out.println("<head><title>Session Result</title></head>");
            out.println("<body>");
            out.println("<h1>Hello, " + escapeHtml(username) + "!</h1>");
            out.println("<p>The name was read from session scope.</p>");
            out.println("<a href='" + request.getContextPath()
                    + "/index.html'>Back to home</a>");
            out.println("</body></html>");

        } else {

            out.println("<!DOCTYPE html>");
            out.println("<html lang='en'>");
            out.println("<head><title>No Session Data</title></head>");
            out.println("<body>");
            out.println("<h1>No saved name found</h1>");
            out.println("<p>Please return to the home page and save a name first.</p>");
            out.println("<a href='" + request.getContextPath()
                    + "/index.html'>Back to home</a>");
            out.println("</body></html>");
        }
    }

    // Minimal HTML escaping for this teaching example.
    private String escapeHtml(String value) {
        return value.replace("&", "&amp;")
                    .replace("<", "&lt;")
                    .replace(">", "&gt;")
                    .replace("\"", "&quot;")
                    .replace("'", "&#x27;");
    }
}
```

#### Line-by-line explanation

**`@WebServlet("/read-session")`**

Maps the Servlet to `/read-session`.

**`doGet(...)`**

The browser opens a link, so it makes a GET request.

**`request.getSession(false)`**

This is an important difference from `getSession()`:

- `getSession()` returns the existing session or creates a new one.
- `getSession(false)` returns the existing session, but returns `null` if none exists.

We use `false` here because simply checking for saved data should not create an empty session.

**`response.setContentType("text/html;charset=UTF-8")`**

Tells the browser that the response is HTML encoded as UTF-8.

**`response.getWriter()`**

Gets a writer for sending text to the browser.

**`session != null`**

Checks whether a session exists before using it. Otherwise, calling a method on `null` would cause a `NullPointerException`.

**`session.getAttribute("username") != null`**

Checks whether the session contains a value under the `"username"` key.

**`session.getAttribute("username")`**

Retrieves the value saved by `SetSessionServlet`.

**`escapeHtml(username)`**

Escapes characters that have special meaning in HTML. User input should never be inserted into HTML without suitable output encoding.

**`else` branch**

If there is no session or no saved username, the page displays a helpful message instead of trying to use missing data.

---

## 9. Test the session example

1. Start the application.
2. Open `http://localhost:8080/ScopeDemo/`.
3. Enter a name in the **Session Scope Demo** form.
4. Click **Save in Session**.
5. The browser is redirected to `/read-session`.
6. The second Servlet reads the name from session scope and displays it.
7. Return to the home page and click **Read saved session value**. The name should still be available while the same session remains valid.

### Session-scope flow

```text
Browser submits name
        |
        v
SetSessionServlet.doPost()
        |
        v
request.getParameter("sessionName")
        |
        v
request.getSession()
        |
        v
session.setAttribute("username", name)
        |
        v
Browser receives redirect
        |
        v
Browser makes a NEW request to /read-session
        |
        v
ReadSessionServlet.doGet()
        |
        v
request.getSession(false)
        |
        v
session.getAttribute("username")
        |
        v
Name displayed
```

---

## 10. `setAttribute()` vs `getAttribute()`

These methods are used with both request and session objects.

| Method | Meaning |
|---|---|
| `setAttribute(key, value)` | Stores a value under a key |
| `getAttribute(key)` | Retrieves the value for that key |
| `removeAttribute(key)` | Removes the value for that key |

Request scope:

```java
request.setAttribute("message", "Saved successfully");
String message = (String) request.getAttribute("message");
```

Session scope:

```java
session.setAttribute("username", "Usama");
String username = (String) session.getAttribute("username");
```

The methods look similar, but the **object on which you call them determines the scope**.

- `request.setAttribute(...)` → request scope.
- `session.setAttribute(...)` → session scope.

---

## 11. Request scope vs session scope

| Feature | Request scope | Session scope |
|---|---|---|
| API | `HttpServletRequest` | `HttpSession` |
| Store data | `request.setAttribute()` | `session.setAttribute()` |
| Read data | `request.getAttribute()` | `session.getAttribute()` |
| Lifetime | Current request/dispatch | Across requests while the session remains valid |
| Common use | Forwarding data to a JSP | Remembering a user across pages |
| Works after a redirect? | Request attributes do not carry over automatically | Yes, if the same session is maintained |
| Shared with another user's session? | No, it belongs to this request | No, it belongs to this session |
| Explicit cleanup | Request ends after processing | Remove attributes or invalidate the session when appropriate |

---

## 12. Removing session data and logging out

To remove just one attribute:

```java
session.removeAttribute("username");
```

To end the current session:

```java
session.invalidate();
```

For example, a logout Servlet could use:

```java
package com.example.scope;

import java.io.IOException;

import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

@WebServlet("/logout")
public class LogoutServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {

        HttpSession session = request.getSession(false);

        if (session != null) {
            session.invalidate();
        }

        response.sendRedirect(request.getContextPath() + "/index.html");
    }
}
```

`getSession(false)` avoids creating a new session just to log out. `invalidate()` invalidates the current session, so its attributes are no longer available.

For a real application, logout should generally use POST and appropriate CSRF protection rather than a simple GET link.

---

## 13. How long does a session last?

A session is not necessarily permanent.

It may end when:

- The application invalidates it.
- The configured inactivity timeout is reached.
- The server or application restarts, depending on server configuration and session persistence.

You can set an inactivity timeout in minutes:

```java
HttpSession session = request.getSession();
session.setMaxInactiveInterval(15 * 60);
```

This sets the timeout to 900 seconds, or 15 minutes, of inactivity.

It does **not** mean that the session is guaranteed to last exactly 15 minutes from creation; activity can reset the inactivity timer.

---

## 14. Common mistakes beginners make

### Mistake 1: Using `getParameter()` to read an attribute

Incorrect:

```java
String name = request.getParameter("username");
```

This reads an HTTP request parameter, usually from a query string or submitted form.

Correct for a request attribute:

```java
String name = (String) request.getAttribute("username");
```

Correct for a session attribute:

```java
String name = (String) session.getAttribute("username");
```

### Mistake 2: Expecting request attributes to survive a redirect

Incorrect assumption:

```java
request.setAttribute("message", "Success");
response.sendRedirect("result.jsp");
```

The redirect starts a new request, so the attribute is not automatically carried over.

Use a forward when the target should handle the same request:

```java
request.setAttribute("message", "Success");
request.getRequestDispatcher("/result.jsp")
       .forward(request, response);
```

Use session scope only if the value genuinely needs to persist across requests.

### Mistake 3: Forgetting that `getSession()` can create a session

```java
HttpSession session = request.getSession();
```

This may create a session even if one did not exist.

If you only want to check for an existing session, use:

```java
HttpSession session = request.getSession(false);
```

Then check whether the result is `null`.

### Mistake 4: Using an attribute before checking whether it exists

```java
String name = (String) session.getAttribute("username");
```

The attribute might not exist, in which case `name` will be `null`. Check for that before using it.

### Mistake 5: Mixing Servlet API versions

For Tomcat 10.1, imports use:

```java
import jakarta.servlet.*;
```

For Tomcat 9, imports use:

```java
import javax.servlet.*;
```

Do not mix the two namespaces in the same application. The matching Maven dependency and deployment environment must agree.

---
