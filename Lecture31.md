# Servlet Filters and FilterChain in Java

## 1. Learning Objectives

After completing this lesson, students will understand:

1. What a Servlet Filter is and why it is needed.
2. How the Servlet request-response lifecycle works.
3. What `Filter`, `FilterChain`, and `FilterConfig` are.
4. How `doFilter()` works, line by line.
5. How multiple filters execute in a specific order.
6. The difference between code before and after `chain.doFilter()`.
7. How to configure filters using annotations and `web.xml`.
8. How to use filters for logging, authentication checks, and request validation.
9. How to build and test a complete filter application using JSP and Servlets.

---

# Part 1: What Is a Servlet Filter?

## 2. Understanding the Problem

Suppose we are building a student portal with these pages:

- Login
- Dashboard
- Profile
- Attendance
- Results

Whenever a student opens a page, we may want to perform some common operations:

- Log the requested URL.
- Record how long a request takes.
- Check whether the user is logged in.
- Set character encoding.
- Add security-related response headers.
- Reject certain requests before they reach the Servlet.

One approach would be to write all this code inside every Servlet.

For example:

```java
protected void doGet(
        HttpServletRequest request,
        HttpServletResponse response
) throws IOException {

    System.out.println("Request received");

    // Authentication check

    // Request validation

    // Actual business logic
}
```

We would have to repeat this code in many Servlets.

That creates duplication and makes the application harder to maintain.

**Solution: Servlet Filters.**

A Filter lets us run code before a request reaches a target resource and/or after that resource processes the request, without putting all that common code inside the Servlet.

## 3. What Is a Servlet Filter?

A **Servlet Filter** is a Java component that can inspect or modify requests and responses as they pass through the web application's filter chain.

A filter can:

- Perform logging.
- Check authentication.
- Validate incoming requests.
- Set request character encoding.
- Add response headers.
- Block a request.
- Wrap a request or response to change how it behaves.

A filter implements:

```java
jakarta.servlet.Filter
```

The central method is:

```java
public void doFilter(
        ServletRequest request,
        ServletResponse response,
        FilterChain chain
) throws IOException, ServletException
```

The method receives three important objects:

| Parameter | Purpose |
|---|---|
| `request` | Represents the incoming request |
| `response` | Represents the response being produced |
| `chain` | Allows processing to continue to the next filter or target resource |

---

# Part 2: Understanding the Filter Lifecycle

## 4. Where Does a Filter Execute?

Suppose the browser requests:

```text
http://localhost:8080/filter-demo/dashboard
```

A filter can intercept the request before `DashboardServlet` executes.

```text
                  Browser
                     |
                     | HTTP Request
                     v
              LoggingFilter
                     |
                     | chain.doFilter()
                     v
             AuthenticationFilter
                     |
                     | chain.doFilter()
                     v
             DashboardServlet
                     |
                     | HTTP Response
                     v
             AuthenticationFilter
                     |
                     v
               LoggingFilter
                     |
                     v
                  Browser
```

The important idea is that filters form a chain.

Each filter decides whether to continue processing by calling `chain.doFilter()`.

If it does, processing moves to the next filter or the target resource. When the target finishes, control returns to the earlier filters, allowing them to perform post-processing.

## 5. Understanding `FilterChain`

`FilterChain` is an interface provided by the Servlet API.

Its important method is:

```java
chain.doFilter(request, response);
```

This means:

> Continue processing this request through the next filter in the chain, or invoke the target resource if no more filters remain.

Consider:

```java
@Override
public void doFilter(
        ServletRequest request,
        ServletResponse response,
        FilterChain chain
) throws IOException, ServletException {

    System.out.println("Before Servlet");

    chain.doFilter(request, response);

    System.out.println("After Servlet");
}
```

The console output will be:

```text
Before Servlet
Servlet is executing
After Servlet
```

Why?

1. The filter starts.
2. It prints `Before Servlet`.
3. `chain.doFilter()` transfers control to the next component.
4. The Servlet executes.
5. Control returns to the filter.
6. The filter prints `After Servlet`.

**Important:** If a filter does not call `chain.doFilter()`, the remaining chain and target resource are not invoked through that call. This is how a filter can block a request.

---

# Part 3: Complete Maven Project

## 6. Project File Structure

Create a Maven Web Application in IntelliJ IDEA.

```text
filter-demo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │   └── servlet/
        │       ├── LoggingFilter.java
        │       ├── AuthenticationFilter.java
        │       ├── DashboardServlet.java
        │       ├── LoginServlet.java
        │       └── LogoutServlet.java
        │
        └── webapp/
            ├── index.jsp
            ├── dashboard.jsp
            └── WEB-INF/
                └── web.xml
```

All Java classes use the same package:

```java
package servlet;
```

We will create two filters:

1. `LoggingFilter` — records request information and execution time.
2. `AuthenticationFilter` — checks whether a user has logged in before accessing the dashboard.

We will also create three Servlets:

- `LoginServlet`
- `DashboardServlet`
- `LogoutServlet`

JSP pages will provide the UI.

## 7. Maven Configuration — `pom.xml`

File location:

```text
filter-demo/pom.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>filter-demo</artifactId>
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
        <finalName>filter-demo</finalName>

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

- `packaging=war` means Maven builds a web application archive.
- `jakarta.servlet-api` provides the Servlet interfaces used by the filters and Servlets.
- `scope=provided` means Tomcat supplies the API at runtime.
- Java 17 is the compilation target.
- Tomcat 10.1 supports Jakarta Servlet 6.0.

---

# Part 4: Create the First Filter

## 8. `LoggingFilter.java`

Location:

```text
src/main/java/servlet/LoggingFilter.java
```

Complete code:

```java
package servlet;

import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.FilterConfig;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;

import java.io.IOException;

@WebFilter("/*")
public class LoggingFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig)
            throws ServletException {

        System.out.println("LoggingFilter initialized");
    }

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException {

        long startTime = System.currentTimeMillis();

        if (request instanceof HttpServletRequest httpRequest) {

            System.out.println("----- Incoming Request -----");

            System.out.println(
                    "Method: " + httpRequest.getMethod()
            );

            System.out.println(
                    "URI: " + httpRequest.getRequestURI()
            );

        }

        System.out.println("LoggingFilter: Before chain");

        try {

            chain.doFilter(request, response);

        } finally {

            long endTime = System.currentTimeMillis();

            long executionTime = endTime - startTime;

            System.out.println("LoggingFilter: After chain");

            System.out.println(
                    "Request processing time: "
                            + executionTime + " ms"
            );

            System.out.println("----------------------------");
        }
    }

    @Override
    public void destroy() {

        System.out.println("LoggingFilter destroyed");
    }
}
```

## 9. Understand `LoggingFilter` Line by Line

### Step 1: Implement the Filter interface

```java
public class LoggingFilter implements Filter
```

The class implements the Servlet API's `Filter` interface.

The container can then manage the filter lifecycle.

### Step 2: `init()`

```java
@Override
public void init(FilterConfig filterConfig)
        throws ServletException {

    System.out.println("LoggingFilter initialized");
}
```

The container calls `init()` when it initializes the filter.

This method is called once for that filter instance, not once for every request.

It is a place to initialize resources or read filter configuration.

### Step 3: `doFilter()`

```java
public void doFilter(
        ServletRequest request,
        ServletResponse response,
        FilterChain chain
)
```

The container invokes this method when a matching request passes through the filter.

Unlike `HttpServlet`, the `Filter` interface uses the general `ServletRequest` and `ServletResponse` types.

### Step 4: Record the start time

```java
long startTime = System.currentTimeMillis();
```

This captures the current time in milliseconds.

We use it to calculate how long the request takes to process.

### Step 5: Access HTTP-specific request information

```java
if (request instanceof HttpServletRequest httpRequest)
```

`ServletRequest` is the general request type. `HttpServletRequest` adds HTTP-specific methods, such as `getMethod()` and `getRequestURI()`.

The `instanceof` pattern checks the type and gives us the HTTP-specific variable `httpRequest`.

### Step 6: Print the HTTP method

```java
System.out.println(
        "Method: " + httpRequest.getMethod()
);
```

Possible values include:

```text
GET
POST
PUT
DELETE
```

The method depends on the HTTP request.

### Step 7: Print the request URI

```java
System.out.println(
        "URI: " + httpRequest.getRequestURI()
);
```

For a request to:

```text
http://localhost:8080/filter-demo/dashboard
```

the URI might be:

```text
/filter-demo/dashboard
```

The URI includes the application context path.

### Step 8: Continue the chain

```java
chain.doFilter(request, response);
```

This is the central line of the filter.

It allows the request to continue to the next filter or the target Servlet.

### Step 9: Execute post-processing

```java
finally {
    long endTime = System.currentTimeMillis();
    long executionTime = endTime - startTime;
}
```

The `finally` block runs whether downstream processing completes normally or throws an exception.

That makes it useful for recording elapsed time.

The measurement includes downstream processing performed during this call, which can include other filters and the target resource.

### Step 10: `destroy()`

```java
@Override
public void destroy() {
    System.out.println("LoggingFilter destroyed");
}
```

The container calls `destroy()` when it removes the filter from service, for example during application shutdown or redeployment.

---

# Part 5: Create the Authentication Filter

## 10. What Is an Authentication Filter?

Suppose the student opens:

```text
/filter-demo/dashboard
```

If the student is logged in, the dashboard should be accessible.

If the student is not logged in, the request should be redirected to the login page.

Without a filter, every protected Servlet might repeat the same check.

With a filter, we can centralize the check.

## 11. `AuthenticationFilter.java`

Location:

```text
src/main/java/servlet/AuthenticationFilter.java
```

```java
package servlet;

import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.FilterConfig;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

import java.io.IOException;

@WebFilter("/dashboard")
public class AuthenticationFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig)
            throws ServletException {

        System.out.println("AuthenticationFilter initialized");
    }

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException {

        HttpServletRequest httpRequest =
                (HttpServletRequest) request;

        HttpServletResponse httpResponse =
                (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession(false);

        boolean loggedIn =
                session != null
                && session.getAttribute("username") != null;

        if (loggedIn) {

            System.out.println(
                    "AuthenticationFilter: User authenticated"
            );

            chain.doFilter(request, response);

        } else {

            System.out.println(
                    "AuthenticationFilter: Access denied"
            );

            httpResponse.sendRedirect(
                    httpRequest.getContextPath() + "/index.jsp"
            );

            // Do not call chain.doFilter() after blocking the request.
        }
    }

    @Override
    public void destroy() {

        System.out.println("AuthenticationFilter destroyed");
    }
}
```

## 12. Understand the Authentication Filter

### Step 1: Map the filter

```java
@WebFilter("/dashboard")
```

The filter is mapped to requests matching `/dashboard`.

The mapping is relative to the web application's context path.

### Step 2: Convert the request and response

```java
HttpServletRequest httpRequest =
        (HttpServletRequest) request;

HttpServletResponse httpResponse =
        (HttpServletResponse) response;
```

This application is an HTTP web application, so the container supplies HTTP request and response objects.

We cast them to access methods such as `getSession()` and `sendRedirect()`.

### Step 3: Obtain the existing session

```java
HttpSession session = httpRequest.getSession(false);
```

Using `false` means that we do not create a new session if the user does not already have one.

This is useful for protected resources because an unauthenticated visitor should not receive a new session simply because the filter checks authentication.

### Step 4: Check the login state

```java
boolean loggedIn =
        session != null
        && session.getAttribute("username") != null;
```

The user is treated as logged in when:

- A session exists.
- The session has a `username` attribute.

This is a simplified classroom authentication check. A real application should use a robust authentication mechanism and verify permissions on the server.

### Step 5: Allow the request

```java
if (loggedIn) {
    chain.doFilter(request, response);
}
```

If the user is authenticated, the filter continues the chain.

The dashboard Servlet can execute.

### Step 6: Block the request

```java
httpResponse.sendRedirect(
        httpRequest.getContextPath() + "/index.jsp"
);
```

If the user is not authenticated, the filter sends a redirect to the login page.

Notice that we do not call:

```java
chain.doFilter(request, response);
```

in the `else` branch.

That is intentional. The dashboard should not execute for an unauthenticated request.

---

# Part 6: Create the Login Page

## 13. `index.jsp`

Location:

```text
src/main/webapp/index.jsp
```

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

        <label for="username">Username:</label>

        <input
            type="text"
            id="username"
            name="username"
            required
        >

        <br><br>

        <label for="password">Password:</label>

        <input
            type="password"
            id="password"
            name="password"
            required
        >

        <br><br>

        <button type="submit">Login</button>

    </form>

</body>
</html>
```

The form submits a POST request to `/login`.

The Servlet will validate the demonstration credentials and create a session on successful login.

---

# Part 7: Create the Login Servlet

## 14. `LoginServlet.java`

Location:

```text
src/main/java/servlet/LoginServlet.java
```

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

        boolean validLogin =
                "rahul".equals(username)
                && "java123".equals(password);

        if (validLogin) {

            HttpSession oldSession = request.getSession(false);

            if (oldSession != null) {
                request.changeSessionId();
            }

            HttpSession session = request.getSession();

            session.setAttribute("username", username);

            response.sendRedirect(
                    request.getContextPath() + "/dashboard"
            );

        } else {

            response.sendRedirect(
                    request.getContextPath() + "/index.jsp"
            );

        }
    }
}
```

### Explanation

1. `getParameter()` reads the submitted username and password.
2. The example checks the credentials against fixed classroom values.
3. On success, an existing session ID is changed if a session already exists.
4. The username is stored in the session.
5. The browser is redirected to the dashboard.

The `changeSessionId()` call helps mitigate session-fixation attacks. In a real login system, use secure password storage, proper authentication, rate limiting, HTTPS, and appropriate error handling.

---

# Part 8: Create the Dashboard Servlet

## 15. `DashboardServlet.java`

Location:

```text
src/main/java/servlet/DashboardServlet.java
```

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

        HttpSession session = request.getSession(false);

        String username =
                (String) session.getAttribute("username");

        request.setAttribute("username", username);

        request.getRequestDispatcher("/dashboard.jsp")
               .forward(request, response);
    }
}
```

The filter should allow the request to reach this Servlet only when a session with the expected attribute exists.

The Servlet retrieves the username and forwards the request to the JSP.

For additional defensive programming, a Servlet may check for a null session before accessing its attributes. Here, the filter performs that check first.

---

# Part 9: Dashboard JSP

## 16. `dashboard.jsp`

Location:

```text
src/main/webapp/dashboard.jsp
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%
    String username = (String) request.getAttribute("username");

    if (username == null) {
        username = "Student";
    }

    String safeUsername = username
            .replace("&", "&amp;")
            .replace("<", "&lt;")
            .replace(">", "&gt;")
            .replace("\"", "&quot;")
            .replace("'", "&#x27;");
%>

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Student Dashboard</title>
</head>

<body>

    <h1>Student Dashboard</h1>

    <p>Welcome, <%= safeUsername %>!</p>

    <p>The AuthenticationFilter allowed this request.</p>

    <form action="${pageContext.request.contextPath}/logout"
          method="post">

        <button type="submit">Logout</button>

    </form>

</body>
</html>
```

The JSP displays the username supplied by the Servlet.

We escape the username before writing it into HTML because it originated from a request parameter.

---

# Part 10: Logout Servlet

## 17. `LogoutServlet.java`

Location:

```text
src/main/java/servlet/LogoutServlet.java
```

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

The Servlet invalidates the session and redirects the browser to the login page.

After logout, a request to `/dashboard` should be blocked by the authentication filter.

---

# Part 11: Understanding the Complete Request Flow

## 18. Scenario A — User Is Not Logged In

The browser requests:

```text
GET /filter-demo/dashboard
```

The flow is:

1. `LoggingFilter` records the request.
2. `AuthenticationFilter` checks the session.
3. No authenticated session is found.
4. The filter redirects to `/index.jsp`.
5. `DashboardServlet` is not invoked.

The important point is that the authentication filter does not call `chain.doFilter()` when it blocks the request.

## 19. Scenario B — User Is Logged In

The user submits the login form.

The flow is:

1. `LoggingFilter` processes the `/login` request.
2. `LoginServlet` validates the credentials.
3. The Servlet stores the username in the session.
4. The browser follows the redirect to `/dashboard`.
5. `LoggingFilter` processes the dashboard request.
6. `AuthenticationFilter` finds the session attribute.
7. `chain.doFilter()` allows processing to continue.
8. `DashboardServlet` executes.
9. `dashboard.jsp` displays the dashboard.

## 20. Example Console Output

Exact order between different filters can depend on the configured filter mapping order. For this example, the intended logical sequence is logging followed by authentication.

A dashboard request may produce output similar to:

```text
----- Incoming Request -----
Method: GET
URI: /filter-demo/dashboard
LoggingFilter: Before chain
AuthenticationFilter: User authenticated
DashboardServlet executing
LoggingFilter: After chain
Request processing time: 18 ms
----------------------------
```

If the user is not logged in, the output may instead include:

```text
AuthenticationFilter: Access denied
```

The dashboard Servlet should not execute for that blocked request.

---

# Part 12: Understanding Filter Order

## 21. What Happens When Multiple Filters Are Mapped?

Suppose we have:

```java
@WebFilter("/*")
```

and:

```java
@WebFilter("/dashboard")
```

Both may apply to a request for `/dashboard`.

The order of filters configured through annotations should not be assumed. If a particular order matters, configure the filter mappings in `web.xml` in the intended order.

The sequence of `<filter-mapping>` elements determines the order in which matching filter mappings are applied.

## 22. Configuring Filter Order Using `web.xml`

If you want a predictable order, remove the `@WebFilter` annotations from `LoggingFilter` and `AuthenticationFilter`, then declare them in `web.xml`.

Example:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         https://jakarta.ee/xml/ns/jakartaee
         https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
         version="6.0">

    <display-name>Filter Demo</display-name>

    <filter>
        <filter-name>LoggingFilter</filter-name>
        <filter-class>servlet.LoggingFilter</filter-class>
    </filter>

    <filter>
        <filter-name>AuthenticationFilter</filter-name>
        <filter-class>servlet.AuthenticationFilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>LoggingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <filter-mapping>
        <filter-name>AuthenticationFilter</filter-name>
        <url-pattern>/dashboard</url-pattern>
    </filter-mapping>

</web-app>
```

With these mappings, a dashboard request passes through `LoggingFilter` first and then `AuthenticationFilter`.

**Do not configure the same filter mapping with both `@WebFilter` and `web.xml` unless you intentionally want both mappings.** For this example, use one configuration approach at a time.

---

# Part 13: `Filter` vs `FilterChain`

## 23. Difference Between `Filter` and `FilterChain`

| Feature | `Filter` | `FilterChain` |
|---|---|---|
| What is it? | Interface implemented by your filter class | Interface used to continue processing |
| Main method | `doFilter()` | `doFilter()` |
| Purpose | Inspect or modify requests and responses | Invoke the next filter or target resource |
| Controls access? | Can block a request by not continuing | Continues the configured chain when invoked |
| Typical code | `class LoggingFilter implements Filter` | `chain.doFilter(request, response)` |

A filter is the component that performs your custom logic.

The chain is the mechanism that allows processing to continue.

## 24. Why Is `chain.doFilter()` So Important?

Compare these two examples.

### Example A — Continue the chain

```java
System.out.println("Before");

chain.doFilter(request, response);

System.out.println("After");
```

Expected sequence:

```text
Before
Next Filter or Servlet
After
```

### Example B — Do not continue the chain

```java
System.out.println("Request blocked");
```

The next filter and target resource are not invoked through this code path.

This is useful for authentication checks, request rejection, and maintenance-mode behavior.

### Important

Calling `chain.doFilter()` more than once is usually not appropriate for ordinary filters. It can cause downstream processing to execute more than once, which may produce duplicate work or unexpected responses.

---

# Part 14: Filter Lifecycle

## 25. Three Main Lifecycle Methods

A Filter has three important lifecycle methods:

### 1. `init()`

Called when the container initializes the filter.

```java
public void init(FilterConfig filterConfig)
        throws ServletException {
    System.out.println("Filter initialized");
}
```

Use it for initialization and configuration.

### 2. `doFilter()`

Called for matching requests.

```java
public void doFilter(
        ServletRequest request,
        ServletResponse response,
        FilterChain chain
) throws IOException, ServletException {
    chain.doFilter(request, response);
}
```

Use it for request/response processing.

### 3. `destroy()`

Called when the container takes the filter out of service.

```java
public void destroy() {
    System.out.println("Filter destroyed");
}
```

Use it to release resources created by the filter.

---

# Part 15: Run the Application

## 26. IntelliJ IDEA and Smart Tomcat

1. Open the Maven project in IntelliJ IDEA.
2. Reload Maven dependencies.
3. Ensure Java 17 is configured.
4. Configure Tomcat 10.1.
5. Install or enable the Smart Tomcat plugin.
6. Create a Smart Tomcat run configuration.
7. Set the context path to `/filter-demo`.
8. Start the application.

Build using Maven:

```bash
mvn clean package
```

The WAR file should be created at:

```text
target/filter-demo.war
```

## 27. Test the Application

### Test 1: Open the login page

```text
http://localhost:8080/filter-demo/index.jsp
```

### Test 2: Access the dashboard without logging in

```text
http://localhost:8080/filter-demo/dashboard
```

Expected result: the request is redirected to the login page.

### Test 3: Log in

Use the classroom credentials:

```text
Username: rahul
Password: java123
```

After successful login, the browser should navigate to the dashboard.

### Test 4: Refresh the dashboard

The session should still contain the username, so the filter should allow the request.

### Test 5: Log out

Click Logout, then try opening the dashboard again.

Expected result: the filter redirects the user to the login page.

---

# Part 16: Common Mistakes

## 28. Mistake 1 — Forgetting `chain.doFilter()`

If a filter neither calls the chain nor intentionally generates a response, the request may stop without the intended resource being executed.

## 29. Mistake 2 — Calling the Chain After Redirecting

Incorrect:

```java
response.sendRedirect("/login");
chain.doFilter(request, response);
```

The filter should not continue to the protected resource after deciding to block the request.

Correct:

```java
response.sendRedirect(
        request.getContextPath() + "/index.jsp"
);

return;
```

## 30. Mistake 3 — Creating a Session for Every Visitor

For a protected resource, prefer:

```java
request.getSession(false);
```

This checks for an existing session without creating one.

## 31. Mistake 4 — Assuming Annotation Order

Do not assume the order of multiple annotation-mapped filters. Use `web.xml` mappings when deterministic ordering is required.

## 32. Mistake 5 — Treating a Username Attribute as Full Authorization

This example checks for a session attribute to teach filter mechanics. Production systems should use reliable authentication and authorization, protect session IDs, use HTTPS, and verify permissions for each protected operation.

---

# Part 17: Practice Questions

## 33. Beginner-Level Questions

1. What is a Servlet Filter?
2. Why do we use filters?
3. What is `FilterChain`?
4. What does `chain.doFilter()` do?
5. What is the purpose of `init()`?
6. When is `destroy()` called?
7. Can a filter block a request?
8. Why is `getSession(false)` useful in an authentication filter?
9. What is the difference between a filter and a Servlet?
10. What is the purpose of `@WebFilter`?

## 34. Practical Assignments

1. Create a filter that prints every requested URI.
2. Create a filter that measures request execution time.
3. Create a filter that sets UTF-8 request encoding.
4. Create a filter that adds a custom response header.
5. Create an authentication filter that protects `/dashboard`.
6. Create a filter that blocks requests to a restricted route when no login session exists.
7. Create two filters and demonstrate their execution order.
8. Configure the same filter through `web.xml` instead of annotations.
9. Create a maintenance filter that returns a 503 status without invoking the target Servlet.
10. Record the order of `Before` and `After` messages when two filters are chained.

---

# Final Summary

A Servlet Filter is used for common request and response processing outside the main business logic of a Servlet.

`FilterChain` allows processing to continue to the next filter or target resource:

```java
chain.doFilter(request, response);
```

A typical filter can perform work before the chain:

```java
System.out.println("Before");
```

Continue processing:

```java
chain.doFilter(request, response);
```

And perform work afterward:

```java
System.out.println("After");
```

Remember: **A filter runs before the target resource and can run again after downstream processing returns. If it intentionally blocks a request, it must not continue the chain.**
