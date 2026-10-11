# Session Management in Java Servlets: `HttpSession` and Cookies

## 1. Learning Objectives

After completing this lesson, students will understand:

1. Why session management is needed in web applications.
2. Why HTTP is called a stateless protocol.
3. What `HttpSession` is and how it works.
4. How to create, read, update, and invalidate a session.
5. What cookies are and how they work.
6. How to create, read, and delete cookies in Servlets.
7. The difference between sessions and cookies.
8. How to use JSP pages to display data.
9. How to build a complete Maven web application using IntelliJ IDEA and Tomcat 10.1.

---

# Part 1: Understanding Session Management

## 2. What Is HTTP?

HTTP stands for **Hypertext Transfer Protocol**. It is the protocol used by a browser and a web server to communicate.

Suppose a student opens a website.

The browser sends:

```http
GET /student HTTP/1.1
Host: localhost:8080
```

The server processes the request and sends a response.

```http
HTTP/1.1 200 OK
Content-Type: text/html
```

The browser displays the returned page.

Now the student opens another page.

```http
GET /courses HTTP/1.1
Host: localhost:8080
```

The server receives another request.

By default, HTTP does not automatically remember the identity or previous actions of the person who sent an earlier request.

This property is called **statelessness**.

## 3. Why Do We Need Session Management?

Imagine building a student portal.

A student logs in with:

```text
Email: student@example.com
Password: ********
```

After a successful login, the student opens:

- Dashboard
- Profile
- Courses
- Attendance
- Results

Should the student have to log in again every time they open another page?

No. We need a way to maintain information across multiple HTTP requests.

This is called **session management**.

Common examples include:

- Login systems
- Shopping carts
- Student dashboards
- Online examination portals
- User preferences
- Temporary messages after form submission

Two common mechanisms are:

1. `HttpSession`
2. Cookies

They are related, but they are not the same thing.

---

# Part 2: Understanding `HttpSession`

## 4. What Is `HttpSession`?

`HttpSession` is an interface provided by the Servlet API that allows a web application to store information associated with a user's session across multiple requests.

For example, after login, we might store:

```java
session.setAttribute("username", "Rahul");
```

When the user opens another page, we can retrieve that information:

```java
String username =
        (String) session.getAttribute("username");
```

The servlet container manages the session.

In a typical Servlet application, the browser receives a session identifier in a cookie such as:

```text
JSESSIONID=ABC123XYZ
```

The browser sends that identifier with subsequent requests, allowing the container to associate those requests with the existing session.

The identifier is just a reference to the session. The session attributes are maintained by the server, not normally stored directly in the session cookie.

> **Important:** Do not store passwords or other sensitive secrets in session attributes unnecessarily. Session identifiers should be protected because someone who steals a valid session identifier may be able to impersonate the session owner.

## 5. How `HttpSession` Works

Consider this flow:

```text
                 Browser
                    |
                    | 1. Request login page
                    v
                Login JSP
                    |
                    | 2. Submit username
                    v
               Login Servlet
                    |
                    | 3. Create/get session
                    |    session.setAttribute(...)
                    v
              Servlet Container
                    |
                    | 4. Session ID cookie
                    |    JSESSIONID=...
                    v
                 Browser
                    |
                    | 5. Request another page
                    |    Cookie: JSESSIONID=...
                    v
             Dashboard Servlet
                    |
                    | 6. Read session attribute
                    v
              Dashboard JSP
```

The browser normally stores the session identifier. The actual session data is maintained by the server.

## 6. Important `HttpSession` Methods

| Method | Purpose |
|---|---|
| `request.getSession()` | Returns the existing session or creates one |
| `request.getSession(false)` | Returns the existing session or `null` |
| `session.setAttribute(name, value)` | Stores an object in the session |
| `session.getAttribute(name)` | Reads an attribute |
| `session.removeAttribute(name)` | Removes an attribute |
| `session.invalidate()` | Invalidates the current session |
| `session.getId()` | Returns the session identifier |
| `session.isNew()` | Indicates whether the session was newly created for this request |
| `session.setMaxInactiveInterval(seconds)` | Sets the maximum inactive interval in seconds |
| `session.getMaxInactiveInterval()` | Gets the configured inactive interval |

### Example: Creating a session

```java
HttpSession session = request.getSession();

session.setAttribute("username", "Rahul");
session.setAttribute("course", "Java");
```

We now have two attributes associated with the session.

### Example: Reading session data

```java
HttpSession session = request.getSession(false);

if (session != null) {
    String username =
            (String) session.getAttribute("username");
}
```

We use `getSession(false)` when we want to check for an existing session without creating a new one.

### Example: Logging out

```java
HttpSession session = request.getSession(false);

if (session != null) {
    session.invalidate();
}
```

Invalidating a session ends that session and makes its attributes unavailable.

---

# Part 3: Complete Project Using `HttpSession`

## 7. Project File Structure

Create a Maven Web Application in IntelliJ IDEA.

```text
session-management-demo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │   └── servlet/
        │       ├── LoginServlet.java
        │       ├── DashboardServlet.java
        │       └── LogoutServlet.java
        │
        └── webapp/
            ├── index.jsp
            ├── dashboard.jsp
            ├── login-error.jsp
            │
            └── WEB-INF/
                └── web.xml
```

We will keep all Java classes in the same package:

```java
package servlet;
```

The JSP pages display the user interface. The Servlets process requests and manage the session.

This example uses hardcoded demonstration credentials. It is for learning session management, not for production authentication.

## 8. Maven Configuration — `pom.xml`

Location:

```text
session-management-demo/pom.xml
```

Complete code:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>session-management-demo</artifactId>
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
        <finalName>session-management-demo</finalName>

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

- `packaging=war` packages the project as a Java web application.
- `jakarta.servlet-api` provides the Servlet API during compilation.
- `scope=provided` means Tomcat provides the API at runtime.
- Java 17 is the compilation target.
- Tomcat 10.1 uses the `jakarta.servlet.*` package.

---

## 9. Login Page — `index.jsp`

Location:

```text
src/main/webapp/index.jsp
```

Complete code:

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Student Login</title>
</head>

<body>

    <h1>Student Login</h1>

    <p>Demo username: rahul</p>
    <p>Demo password: java123</p>

    <form action="${pageContext.request.contextPath}/login"
          method="post">

        <div>
            <label for="username">Username:</label>

            <input
                type="text"
                id="username"
                name="username"
                required
            >
        </div>

        <br>

        <div>
            <label for="password">Password:</label>

            <input
                type="password"
                id="password"
                name="password"
                required
            >
        </div>

        <br>

        <button type="submit">Login</button>

    </form>

</body>

</html>
```

### Explanation

**JSP directive**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```

This tells the JSP engine to generate an HTML response using UTF-8 encoding.

**Form action**

```jsp
action="${pageContext.request.contextPath}/login"
```

This builds the login URL using the application's context path. If the context path is `/session-management-demo`, the action becomes:

```text
/session-management-demo/login
```

**Form method**

```html
method="post"
```

The browser submits the form using HTTP POST.

**Input names**

```html
name="username"
name="password"
```

These names become request parameters that the servlet can read.

---

## 10. Login Servlet — `LoginServlet.java`

Location:

```text
src/main/java/servlet/LoginServlet.java
```

Complete code:

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

import java.io.IOException;

@WebServlet("/login")
public class LoginServlet extends HttpServlet {

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        String username = request.getParameter("username");
        String password = request.getParameter("password");

        // Demonstration credentials only.
        boolean validLogin =
                "rahul".equals(username)
                && "java123".equals(password);

        if (validLogin) {

            /*
             * Create a new session identifier after authentication.
             * This helps reduce session-fixation risk.
             */
            request.changeSessionIdIfAvailable();

            HttpSession session = request.getSession();

            session.setAttribute("username", username);
            session.setAttribute("course", "Java");

            // Redirect to the dashboard.
            response.sendRedirect(
                    request.getContextPath() + "/dashboard"
            );

        } else {

            response.sendRedirect(
                    request.getContextPath() + "/login-error.jsp"
            );

        }
    }
}
```

### Important correction for the code above

`HttpServletRequest` does **not** have a standard method named `changeSessionIdIfAvailable()`. Use the following correct, complete `LoginServlet.java` instead. It changes an existing session ID only if the request already has a session, and otherwise creates a new session after validating the credentials.

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

import java.io.IOException;

@WebServlet("/login")
public class LoginServlet extends HttpServlet {

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        String username = request.getParameter("username");
        String password = request.getParameter("password");

        // Demonstration credentials only.
        boolean validLogin =
                "rahul".equals(username)
                && "java123".equals(password);

        if (validLogin) {

            HttpSession existingSession = request.getSession(false);

            if (existingSession != null) {
                // Rotate the session ID after successful authentication.
                request.changeSessionId();
            }

            HttpSession session = request.getSession();

            session.setAttribute("username", username);
            session.setAttribute("course", "Java");

            response.sendRedirect(
                    request.getContextPath() + "/dashboard"
            );

        } else {

            response.sendRedirect(
                    request.getContextPath() + "/login-error.jsp"
            );

        }
    }
}
```

### Explanation

#### Step 1: Read the username and password

```java
String username = request.getParameter("username");
String password = request.getParameter("password");
```

These values come from the form fields in `index.jsp`.

#### Step 2: Validate the credentials

```java
boolean validLogin =
        "rahul".equals(username)
        && "java123".equals(password);
```

For this demonstration, only the provided username and password are accepted.

In a real application, credentials must be checked against a proper user store, passwords must be securely hashed, and login attempts should be protected against abuse.

#### Step 3: Create or obtain the session

```java
HttpSession session = request.getSession();
```

If an existing session is available, it is returned. Otherwise, a new session is created.

#### Step 4: Store data

```java
session.setAttribute("username", username);
session.setAttribute("course", "Java");
```

The session now stores the username and course.

These attributes remain available on later requests associated with the same session.

#### Step 5: Redirect

```java
response.sendRedirect(
        request.getContextPath() + "/dashboard"
);
```

The browser makes a new request to the dashboard URL.

---

## 11. Dashboard Servlet — `DashboardServlet.java`

Location:

```text
src/main/java/servlet/DashboardServlet.java
```

Complete code:

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

import java.io.IOException;

@WebServlet("/dashboard")
public class DashboardServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        // Look for an existing session without creating one.
        HttpSession session = request.getSession(false);

        if (session == null
                || session.getAttribute("username") == null) {

            response.sendRedirect(
                    request.getContextPath() + "/index.jsp"
            );

            return;
        }

        // Forward to the JSP that displays the dashboard.
        request.getRequestDispatcher("/dashboard.jsp")
               .forward(request, response);
    }
}
```

### Explanation

#### Why `getSession(false)`?

```java
HttpSession session = request.getSession(false);
```

We want to check whether the user already has a session.

If we used `request.getSession()` here, a new session could be created for an unauthenticated visitor. For a protected page, checking for an existing session is usually the better choice.

#### Check authentication state

```java
if (session == null
        || session.getAttribute("username") == null)
```

If there is no session or the expected username attribute is missing, the user is redirected to the login page.

#### Forward to JSP

```java
request.getRequestDispatcher("/dashboard.jsp")
       .forward(request, response);
```

The servlet checks access, then forwards the same request to the JSP for rendering.

This is a good separation of responsibilities:

- Servlet: request processing and access checks.
- JSP: presentation and HTML output.

---

## 12. Dashboard UI — `dashboard.jsp`

Location:

```text
src/main/webapp/dashboard.jsp
```

Complete code:

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="jakarta.servlet.http.HttpSession" %>
<%@ page import="java.net.URLEncoder" %>
<%@ page import="java.nio.charset.StandardCharsets" %>

<%
    HttpSession userSession = request.getSession(false);

    if (userSession == null
            || userSession.getAttribute("username") == null) {

        response.sendRedirect(
                request.getContextPath() + "/index.jsp"
        );

        return;
    }

    String username =
            (String) userSession.getAttribute("username");

    String course =
            (String) userSession.getAttribute("course");

    String safeUsername =
            username.replace("&", "&amp;")
                    .replace("<", "&lt;")
                    .replace(">", "&gt;")
                    .replace("\"", "&quot;")
                    .replace("'", "&#x27;");

    String safeCourse =
            course.replace("&", "&amp;")
                  .replace("<", "&lt;")
                  .replace(">", "&gt;")
                  .replace("\"", "&quot;")
                  .replace("'", "&#x27;");
%>

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Student Dashboard</title>
</head>

<body>

    <h1>Student Dashboard</h1>

    <p>Welcome, <%= safeUsername %>!</p>

    <p>Your course is: <%= safeCourse %></p>

    <h2>Session Information</h2>

    <p>
        Session ID:
        <%= userSession.getId() %>
    </p>

    <p>
        Session is new:
        <%= userSession.isNew() %>
    </p>

    <form action="${pageContext.request.contextPath}/logout"
          method="post">

        <button type="submit">Logout</button>

    </form>

</body>

</html>
```

### Explanation

The JSP retrieves session attributes:

```java
userSession.getAttribute("username");
userSession.getAttribute("course");
```

It then displays the values.

The session ID is shown only for classroom demonstration. Avoid exposing session identifiers in real user interfaces or logs, because possession of a valid identifier can grant access to the session.

The JSP repeats the access check as a defensive measure, but the servlet should remain the main place for access control.

In larger applications, prefer the JSP Expression Language and JSTL rather than putting Java scriptlets directly in JSP files. Scriptlets are used here to make the underlying session operations explicit for beginners.

---

## 13. Login Error Page — `login-error.jsp`

Location:

```text
src/main/webapp/login-error.jsp
```

Complete code:

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Login Failed</title>
</head>

<body>

    <h1>Login Failed</h1>

    <p>Invalid username or password.</p>

    <a href="${pageContext.request.contextPath}/index.jsp">
        Try Again
    </a>

</body>

</html>
```

---

## 14. Logout Servlet — `LogoutServlet.java`

Location:

```text
src/main/java/servlet/LogoutServlet.java
```

Complete code:

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

import java.io.IOException;

@WebServlet("/logout")
public class LogoutServlet extends HttpServlet {

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        HttpSession session = request.getSession(false);

        if (session != null) {
            session.invalidate();
        }

        response.sendRedirect(
                request.getContextPath() + "/index.jsp"
        );
    }
}
```

### Explanation

```java
HttpSession session = request.getSession(false);
```

This checks for an existing session without creating a new one.

```java
session.invalidate();
```

This invalidates the session, ending the server-side session associated with that identifier.

The browser may still temporarily have the old session cookie. However, that old identifier should no longer resolve to the invalidated session. If a later request creates a new session, the browser may receive a new session identifier.

Finally, the browser is redirected to the login page.

---

## 15. Optional `web.xml`

Location:

```text
src/main/webapp/WEB-INF/web.xml
```

Complete code:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         https://jakarta.ee/xml/ns/jakartaee
         https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
         version="6.0">

    <display-name>Session Management Demo</display-name>

    <session-config>
        <session-timeout>20</session-timeout>
        <cookie-config>
            <http-only>true</http-only>
        </cookie-config>
    </session-config>

</web-app>
```

### Explanation

`session-timeout` is measured in minutes. Here, the session's default inactivity timeout is 20 minutes.

`http-only` means client-side JavaScript cannot read the session cookie through the usual `document.cookie` API.

For production applications served over HTTPS, also configure secure cookie behavior and review the `SameSite` policy.

---

# Part 4: Running the `HttpSession` Project

## 16. Configure IntelliJ IDEA and Smart Tomcat

1. Open the Maven project in IntelliJ IDEA.
2. Reload Maven dependencies.
3. Ensure Java 17 is configured.
4. Install Tomcat 10.1.
5. Enable the Smart Tomcat plugin.
6. Create a Smart Tomcat run configuration.
7. Configure the Tomcat installation and application context path.
8. Start Tomcat.

Build the application:

```bash
mvn clean package
```

Expected WAR file:

```text
target/session-management-demo.war
```

The context path depends on the Smart Tomcat run configuration. If you set it to `/session-management-demo`, open:

```text
http://localhost:8080/session-management-demo/
```

### Test sequence

1. Enter username `rahul`.
2. Enter password `java123`.
3. Click Login.
4. Observe the dashboard.
5. Open another tab in the same browser and revisit the dashboard URL.
6. The same browser session should normally be recognized.
7. Click Logout.
8. Try opening the dashboard again.
9. You should be redirected to the login page.

---

# Part 5: Understanding Cookies

## 17. What Is a Cookie?

A **cookie** is a small piece of data that a server asks the browser to store.

For example, a server can send this response header:

```http
Set-Cookie: theme=dark; Path=/
```

The browser stores the cookie according to its attributes and policies.

When a later request matches the cookie's scope, the browser may send:

```http
Cookie: theme=dark
```

Cookies are often used for:

- Session identifiers
- Language preferences
- Theme preferences
- Remembering non-sensitive settings
- Certain tracking or analytics purposes

Cookies are stored on the client side, usually in the browser's cookie storage. They should not be treated as trusted data: a client can modify ordinary cookie values.

## 18. How Cookies Work

```text
             Browser
                |
                | 1. Request a page
                v
             Servlet
                |
                | 2. Create a cookie
                |    theme=dark
                v
             Response
                |
                | 3. Set-Cookie header
                v
             Browser
                |
                | 4. Store cookie
                |
                | 5. Make another request
                |    Cookie: theme=dark
                v
             Servlet
                |
                | 6. Read the cookie
                v
          Display preference
```

The server sets a cookie using the `Set-Cookie` response header. The browser later sends matching cookies using the `Cookie` request header.

## 19. Important `Cookie` Methods

In the Servlet API, `jakarta.servlet.http.Cookie` represents a cookie.

| Method | Purpose |
|---|---|
| `new Cookie(name, value)` | Creates a cookie |
| `response.addCookie(cookie)` | Adds a cookie to the response |
| `request.getCookies()` | Retrieves cookies sent with the request |
| `cookie.getName()` | Reads the cookie's name |
| `cookie.getValue()` | Reads the cookie's value |
| `cookie.setMaxAge(seconds)` | Sets the cookie's lifetime |
| `cookie.setPath(path)` | Sets the cookie's path |
| `cookie.setHttpOnly(true)` | Prevents ordinary client-side JavaScript access |
| `cookie.setSecure(true)` | Sends the cookie only over secure HTTPS connections |
| `cookie.setAttribute("SameSite", "Lax")` | Sets the SameSite attribute in Servlet 6.0 |

### Session cookie vs persistent cookie

A cookie with no explicit `Max-Age` or `Expires` is generally a browser-session cookie, although exact persistence behavior depends on the browser.

A cookie with a positive `Max-Age` is intended to persist for that many seconds.

Example:

```java
Cookie cookie = new Cookie("theme", "dark");

cookie.setMaxAge(7 * 24 * 60 * 60);
cookie.setPath("/");

response.addCookie(cookie);
```

This requests a cookie that lasts for seven days.

---

# Part 6: Complete Cookie Example Using a JSP

We will build a small theme preference feature.

The user chooses `light` or `dark`. The Servlet stores the choice in a cookie, then redirects to a JSP page. The JSP displays the preference.

## 20. Cookie Project File Structure

Add these files to the same Maven project:

```text
session-management-demo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │   └── servlet/
        │       ├── ThemeServlet.java
        │       └── ClearThemeServlet.java
        │
        └── webapp/
            ├── theme.jsp
            └── theme-result.jsp
```

The earlier `HttpSession` files remain in the project.

## 21. Theme Form — `theme.jsp`

Location:

```text
src/main/webapp/theme.jsp
```

Complete code:

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Theme Preference</title>
</head>

<body>

    <h1>Choose Your Theme</h1>

    <form action="${pageContext.request.contextPath}/theme"
          method="post">

        <label for="theme">Select a theme:</label>

        <select id="theme" name="theme" required>
            <option value="light">Light</option>
            <option value="dark">Dark</option>
        </select>

        <button type="submit">Save Preference</button>

    </form>

    <p>
        <a href="${pageContext.request.contextPath}/theme-result.jsp">
            View Saved Preference
        </a>
    </p>

</body>

</html>
```

The form submits the selected value using POST.

For example:

```text
theme=dark
```

## 22. Create the Cookie — `ThemeServlet.java`

Location:

```text
src/main/java/servlet/ThemeServlet.java
```

Complete code:

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/theme")
public class ThemeServlet extends HttpServlet {

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        String theme = request.getParameter("theme");

        // Accept only the values supported by the application.
        if (!"light".equals(theme) && !"dark".equals(theme)) {

            response.sendError(
                    HttpServletResponse.SC_BAD_REQUEST,
                    "Invalid theme."
            );

            return;
        }

        Cookie themeCookie = new Cookie("theme", theme);

        // Keep the cookie for seven days.
        themeCookie.setMaxAge(7 * 24 * 60 * 60);

        // Send the cookie for the application path.
        themeCookie.setPath(request.getContextPath().isEmpty()
                ? "/"
                : request.getContextPath());

        // Prevent ordinary JavaScript access.
        themeCookie.setHttpOnly(true);

        // Add this attribute when the application is served over HTTPS.
        themeCookie.setSecure(request.isSecure());

        // Configure SameSite in Servlet 6.0.
        themeCookie.setAttribute("SameSite", "Lax");

        response.addCookie(themeCookie);

        response.sendRedirect(
                request.getContextPath() + "/theme-result.jsp"
        );
    }
}
```

### Explanation

#### Create a cookie

```java
Cookie themeCookie = new Cookie("theme", theme);
```

The cookie name is `theme`. The value is either `light` or `dark`.

#### Set the lifetime

```java
themeCookie.setMaxAge(7 * 24 * 60 * 60);
```

The cookie is configured to last for seven days, subject to browser policies and user actions.

#### Set the path

```java
themeCookie.setPath(request.getContextPath().isEmpty()
        ? "/"
        : request.getContextPath());
```

This limits the cookie to the application's context path, except when the application runs at the server root.

#### Set `HttpOnly`

```java
themeCookie.setHttpOnly(true);
```

This prevents ordinary JavaScript from reading the cookie. It does not stop the browser from sending it with matching requests.

#### Set `Secure`

```java
themeCookie.setSecure(request.isSecure());
```

On HTTPS requests, this marks the cookie as secure. For a production application, ensure the site consistently uses HTTPS and configure secure cookies accordingly.

#### Set `SameSite`

```java
themeCookie.setAttribute("SameSite", "Lax");
```

This requests a `SameSite=Lax` policy, which helps restrict cookies on certain cross-site requests. It is not a replacement for CSRF protection in applications that need it.

#### Send the cookie

```java
response.addCookie(themeCookie);
```

The response includes a `Set-Cookie` header. The browser stores the cookie and sends it on subsequent matching requests.

## 23. Read the Cookie — `theme-result.jsp`

Location:

```text
src/main/webapp/theme-result.jsp
```

Complete code:

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="jakarta.servlet.http.Cookie" %>

<%
    String theme = "not set";

    Cookie[] cookies = request.getCookies();

    if (cookies != null) {

        for (Cookie cookie : cookies) {

            if ("theme".equals(cookie.getName())) {
                theme = cookie.getValue();
                break;
            }
        }
    }

    // Only allow expected values to influence presentation.
    if (!"light".equals(theme) && !"dark".equals(theme)) {
        theme = "not set";
    }
%>

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Saved Theme</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 30px;
        }

        .light {
            background: white;
            color: black;
        }

        .dark {
            background: #222;
            color: white;
        }
    </style>
</head>

<body class="<%= theme.equals("dark") ? "dark" : "light" %>">

    <h1>Saved Theme Preference</h1>

    <p>Your saved theme is: <strong><%= theme %></strong></p>

    <p>
        <a href="${pageContext.request.contextPath}/theme.jsp">
            Change Theme
        </a>
    </p>

    <form action="${pageContext.request.contextPath}/clear-theme"
          method="post">

        <button type="submit">Delete Theme Cookie</button>

    </form>

</body>

</html>
```

### Explanation

#### Read cookies

```java
Cookie[] cookies = request.getCookies();
```

The method returns cookies included with the request. It can return `null` if no cookies were sent.

#### Find the desired cookie

```java
for (Cookie cookie : cookies) {
    if ("theme".equals(cookie.getName())) {
        theme = cookie.getValue();
        break;
    }
}
```

We inspect each cookie and select the one named `theme`.

#### Why validate the value?

Cookie values originate from the client and can be changed. We only accept the values the application supports.

For this example, the only accepted values are:

```text
light
dark
```

The JSP applies a corresponding CSS class.

---

## 24. Delete a Cookie — `ClearThemeServlet.java`

Location:

```text
src/main/java/servlet/ClearThemeServlet.java
```

Complete code:

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/clear-theme")
public class ClearThemeServlet extends HttpServlet {

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        Cookie themeCookie = new Cookie("theme", "");

        // A zero age requests immediate removal.
        themeCookie.setMaxAge(0);

        // Use the same path as when the cookie was created.
        themeCookie.setPath(request.getContextPath().isEmpty()
                ? "/"
                : request.getContextPath());

        themeCookie.setHttpOnly(true);
        themeCookie.setSecure(request.isSecure());
        themeCookie.setAttribute("SameSite", "Lax");

        response.addCookie(themeCookie);

        response.sendRedirect(
                request.getContextPath() + "/theme-result.jsp"
        );
    }
}
```

### How does cookie deletion work?

Cookies are not generally deleted by calling a Java method on the browser.

Instead, the server sends another cookie with the same name and matching scope, but with a zero `Max-Age`.

```java
themeCookie.setMaxAge(0);
```

The browser receives the updated cookie instruction and removes the matching cookie.

**Important:** The cookie's path and domain must match the cookie you are trying to remove. Otherwise, a separate cookie may be affected, or the original may remain.

---

# Part 7: Testing the Cookie Example

## 25. Run the Project

Start the application through Smart Tomcat.

Assuming the context path is `/session-management-demo`, open:

```text
http://localhost:8080/session-management-demo/theme.jsp
```

### Test 1: Save a theme

1. Choose `dark`.
2. Click **Save Preference**.
3. The browser redirects to `theme-result.jsp`.
4. The page displays `dark` and applies the dark background.

### Test 2: Refresh the result page

Refresh `theme-result.jsp`.

The browser sends the saved `theme` cookie again, so the page should continue to display the selected preference.

### Test 3: Inspect the cookie

In Chrome:

1. Open Developer Tools.
2. Select the **Application** tab.
3. Open **Storage → Cookies**.
4. Select the application's origin.
5. Look for the `theme` cookie.

You should see a cookie similar to:

```text
Name: theme
Value: dark
Path: /session-management-demo
HttpOnly: true
```

The cookie's Secure flag depends on whether the request was served over HTTPS.

### Test 4: Delete the cookie

Click **Delete Theme Cookie**.

The browser is redirected to the result page. The page should now display:

```text
Your saved theme is: not set
```

---

# Part 8: `HttpSession` vs Cookies

## 26. Main Differences

| Feature | `HttpSession` | Cookie |
|---|---|---|
| Where data is normally stored | Server | Browser |
| Main purpose | Associate server-side data with a user session | Store small client-side values or identifiers |
| Accessed using | `HttpSession` | `Cookie` |
| Typical creation | `request.getSession()` | `new Cookie(name, value)` |
| Typical retrieval | `session.getAttribute(name)` | `request.getCookies()` |
| Can store Java objects? | Yes, as session attributes | No; cookie values are strings |
| Data visible to the client? | Session attributes normally stay server-side | Cookie values can be visible to the client |
| Automatically sent on matching requests? | The browser sends the session ID cookie | Yes, subject to cookie scope and browser policies |
| Lifetime | Inactivity timeout, invalidation, or container lifecycle | Browser-session lifetime or configured expiry |
| Example | Store authenticated username | Remember a theme preference |

## 27. How Are They Related?

In many Servlet applications, `HttpSession` uses a cookie named `JSESSIONID` to help identify the session.

For example:

```text
Browser cookie:
JSESSIONID=ABC123XYZ
```

The browser sends that identifier to the server with subsequent requests.

The servlet container uses it to locate the associated session and its attributes.

The actual username may be stored on the server:

```java
session.setAttribute("username", "rahul");
```

The cookie typically holds the identifier, not the complete session object.

---

# Part 9: Important Security Concepts

## 28. Never Trust Cookie Values

A user can modify ordinary cookie values.

Do not assume that this is safe:

```java
Cookie roleCookie = new Cookie("role", "admin");
```

If authorization depends on a client-controlled cookie, a user may change its value.

Use server-side authorization checks and a trusted source of role information.

## 29. Protect Session Identifiers

A session identifier can grant access to a user's session.

Recommended practices include:

- Use HTTPS.
- Mark session cookies `Secure` when using HTTPS.
- Use `HttpOnly` for session cookies.
- Configure an appropriate `SameSite` policy.
- Regenerate the session identifier after authentication when appropriate.
- Invalidate the session during logout.
- Avoid placing session identifiers in URLs.
- Avoid logging or displaying session identifiers in production.
- Configure suitable session inactivity timeouts.

## 30. `HttpSession` Is Not Permanent Storage

Session data is not a replacement for a database.

For example:

```java
session.setAttribute("username", "rahul");
```

This information is available while the session remains valid and the application retains the session state.

It can disappear when the session expires, is invalidated, or the application environment loses its session data.

For persistent student records, use a database. For temporary per-user web state, `HttpSession` is often suitable.

---

# Part 10: Practice Questions

## 31. Beginner-Level Tasks

1. Store a student's name in the session and display it on a JSP.
2. Store a student's semester in the session.
3. Add a button that updates the course stored in the session.
4. Display whether the current session is new.
5. Set the session inactivity timeout to five minutes.
6. Create a logout button that invalidates the session.
7. Create a cookie named `language` with value `English`.
8. Read and display the `language` cookie in a JSP.
9. Add a dropdown to choose `English` or `Hindi` and save it in a cookie.
10. Add a button to delete the language cookie.

## 32. Intermediate-Level Tasks

1. Create a login form that redirects to a dashboard after authentication.
2. Restrict the dashboard to users who have a valid login session.
3. Redirect unauthenticated users to the login page.
4. Add a profile page that reads the username from the session.
5. Create a cookie that remembers a preferred theme for seven days.
6. Display a welcome message using session attributes.
7. Remove one session attribute without invalidating the whole session.
8. Invalidate the session and verify that the dashboard becomes inaccessible.
9. Compare the session ID before and after authentication.
10. Explain why the session identifier should not be displayed in a production dashboard.

---

# Final Revision

**`HttpSession`**

```java
HttpSession session = request.getSession();

session.setAttribute("username", "Rahul");

String username =
        (String) session.getAttribute("username");

session.removeAttribute("username");

session.invalidate();
```

**Cookie**

```java
Cookie cookie = new Cookie("theme", "dark");

cookie.setMaxAge(7 * 24 * 60 * 60);
cookie.setPath("/");
cookie.setHttpOnly(true);

response.addCookie(cookie);
```

**Read cookies**

```java
Cookie[] cookies = request.getCookies();

if (cookies != null) {
    for (Cookie cookie : cookies) {
        System.out.println(
                cookie.getName() + " = " + cookie.getValue()
        );
    }
}
```
