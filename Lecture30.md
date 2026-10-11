# Session Management in Java Servlets
# Part C: URL Rewriting and Part D: Hidden Form Fields

## 1. Learning Objectives

After completing this lesson, students will understand:

1. Why URL rewriting and hidden form fields are used.
2. How URL rewriting carries data between requests.
3. How `response.encodeURL()` works.
4. How hidden form fields transfer values from one form submission to another.
5. The difference between URL rewriting, hidden fields, cookies, and `HttpSession`.
6. How to build complete working examples using JSP and Servlets.
7. The limitations and security risks of these techniques.

---

# Part A: URL Rewriting

## 2. What Is URL Rewriting?

URL rewriting is a session-tracking technique in which information is added to a URL so that it can be passed to another request.

Suppose a student visits:

```text
http://localhost:8080/session-demo/profile
```

We want to pass the student's name to another page.

One possibility is to add the name as a query parameter:

```text
http://localhost:8080/session-demo/profile?username=Rahul
```

The URL now contains:

```text
username=Rahul
```

The destination servlet can read the value using:

```java
String username = request.getParameter("username");
```

This is a simple form of passing data through a URL.

### Important distinction

There are two related concepts:

- **Query-parameter URL rewriting:** Appending parameters such as `?username=Rahul` to a URL.
- **Servlet session URL rewriting:** Encoding a URL using `response.encodeURL()` so the container can append a session identifier, often in the form `;jsessionid=...`, when necessary.

Both are relevant to session management, but they serve different purposes. Query parameters carry application data, while `encodeURL()` supports container-managed session tracking.

## 3. How Query-Parameter URL Rewriting Works

```text
Browser
   |
   | 1. Open profile form
   v
Profile.jsp
   |
   | 2. Click Continue
   |    URL includes username
   v
Browser
   |
   | 3. GET /profile?username=Rahul
   v
ProfileServlet
   |
   | 4. Read request parameter
   v
ProfileResult.jsp
   |
   | 5. Display username
   v
Browser
```

The data is included in the URL, so it can be sent with the next request.

### Example URL

```text
http://localhost:8080/session-demo/profile-result?username=Rahul
```

The servlet reads it using:

```java
String username = request.getParameter("username");
```

---

## 4. Complete Maven Project Structure

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
        │       ├── UrlStartServlet.java
        │       └── UrlResultServlet.java
        │
        └── webapp/
            ├── index.jsp
            ├── url-start.jsp
            ├── url-result.jsp
            │
            └── WEB-INF/
                └── web.xml
```

If you already created the project in the previous session management lesson, you can add these files to that project instead of creating another one.

## 5. Maven Configuration — `pom.xml`

Location:

```text
session-management-demo/pom.xml
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

- `war` packages the application as a web archive.
- `jakarta.servlet-api` supplies the Servlet API during compilation.
- `provided` means Tomcat supplies the Servlet API at runtime.
- Java 17 is used for compilation.
- Tomcat 10.1 uses `jakarta.servlet.*`, not `javax.servlet.*`.

---

## 6. Home Page — `index.jsp`

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
    <title>Session Management Demo</title>
</head>
<body>

    <h1>Session Management</h1>

    <h2>URL Rewriting</h2>

    <a href="${pageContext.request.contextPath}/url-start">
        Open URL Rewriting Example
    </a>

</body>
</html>
```

The link opens the servlet mapped to `/url-start`.

The JSP expression:

```jsp
${pageContext.request.contextPath}
```

automatically includes the application's context path. This avoids hardcoding the application name in every URL.

---

## 7. URL Rewriting Start Servlet — `UrlStartServlet.java`

Location:

```text
src/main/java/servlet/UrlStartServlet.java
```

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/url-start")
public class UrlStartServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        request.getRequestDispatcher("/url-start.jsp")
               .forward(request, response);
    }
}
```

### Explanation

```java
@WebServlet("/url-start")
```

Maps the URL `/url-start` to this servlet.

```java
request.getRequestDispatcher("/url-start.jsp")
       .forward(request, response);
```

Forwards the same request internally to the JSP that displays the form.

The browser's address bar remains `/url-start` because this is a forward, not a redirect.

---

## 8. Start Form — `url-start.jsp`

Location:

```text
src/main/webapp/url-start.jsp
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>URL Rewriting</title>
</head>
<body>

    <h1>URL Rewriting Example</h1>

    <form action="${pageContext.request.contextPath}/url-result"
          method="get">

        <label for="username">Enter Your Name:</label>

        <input
            type="text"
            id="username"
            name="username"
            required
        >

        <button type="submit">Continue</button>

    </form>

</body>
</html>
```

### Explanation

The form uses:

```html
method="get"
```

The browser sends the submitted value as a query parameter.

If the student enters `Rahul`, the resulting URL will look like:

```text
/session-management-demo/url-result?username=Rahul
```

Spaces and special characters are encoded by the browser as necessary.

**Important:** Query parameters are visible in the URL. Do not use them for passwords, authentication secrets, or other sensitive information.

---

## 9. URL Result Servlet — `UrlResultServlet.java`

Location:

```text
src/main/java/servlet/UrlResultServlet.java
```

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/url-result")
public class UrlResultServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        String username = request.getParameter("username");

        if (username == null || username.isBlank()) {
            response.sendError(
                    HttpServletResponse.SC_BAD_REQUEST,
                    "Username is required."
            );

            return;
        }

        request.setAttribute("username", username);

        request.getRequestDispatcher("/url-result.jsp")
               .forward(request, response);
    }
}
```

### Explanation

```java
String username = request.getParameter("username");
```

Reads the `username` query parameter from the URL.

For this URL:

```text
/url-result?username=Rahul
```

the variable contains:

```text
Rahul
```

The servlet then forwards the request to the result JSP.

---

## 10. Result Page — `url-result.jsp`

Location:

```text
src/main/webapp/url-result.jsp
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="java.util.Objects" %>

<%
    String username = (String) request.getAttribute("username");

    if (username == null) {
        username = "Unknown";
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
    <title>URL Rewriting Result</title>
</head>
<body>

    <h1>URL Rewriting Result</h1>

    <p>Welcome, <%= safeUsername %>!</p>

    <p>
        The name was passed through the URL as a query parameter.
    </p>

    <a href="${pageContext.request.contextPath}/url-start">
        Try Again
    </a>

</body>
</html>
```

### What happens here?

The servlet stores the value as a request attribute:

```java
request.setAttribute("username", username);
```

The JSP reads it:

```jsp
String username = (String) request.getAttribute("username");
```

This works because the servlet forwards the same request to the JSP.

Notice the distinction:

- `request.getParameter("username")` reads a submitted query parameter.
- `request.getAttribute("username")` reads a server-side attribute placed on the request.

They are not the same thing.

---

## 11. How to Run the URL Rewriting Example

1. Reload Maven dependencies in IntelliJ IDEA.
2. Configure Smart Tomcat to use Tomcat 10.1.
3. Set the application context path to `/session-management-demo`.
4. Start the application.
5. Open:

```text
http://localhost:8080/session-management-demo/
```

6. Click **Open URL Rewriting Example**.
7. Enter `Rahul`.
8. Click **Continue**.

The address bar should display a URL similar to:

```text
http://localhost:8080/session-management-demo/url-result?username=Rahul
```

The page displays:

```text
URL Rewriting Result

Welcome, Rahul!
```

### What have we learned?

The browser passed the name using the URL. The servlet read the value from the request parameter and forwarded it to a JSP.

However, this example demonstrates **passing data through a URL**. It does not create a server-side session or prove that a user is authenticated.

---

# Part B: Servlet Session URL Rewriting with `encodeURL()`

## 12. Why Is `response.encodeURL()` Important?

Servlet containers can track sessions using cookies. If cookies are unavailable or disabled, URL-based session tracking may be used when enabled and supported by the container.

The Servlet API provides:

```java
response.encodeURL(url);
```

The container can encode the URL with the session identifier when needed.

A URL might become:

```text
/dashboard;jsessionid=ABC123XYZ
```

The exact output depends on the container and session-tracking configuration. If URL encoding is unnecessary, the URL may remain unchanged.

### Example

```java
String dashboardUrl =
        response.encodeURL(
                request.getContextPath() + "/dashboard"
        );
```

Then output the encoded URL as a link.

### Important rules

- Use `encodeURL()` for URLs that point to resources within the application.
- Use `encodeRedirectURL()` for URLs passed to `sendRedirect()`.
- Do not manually append `;jsessionid=...`.
- Never expose session IDs unnecessarily, because they are credentials for accessing the associated session.

## 13. Demonstration Servlet — `EncodedUrlServlet.java`

Add this file:

```text
src/main/java/servlet/EncodedUrlServlet.java
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
import java.io.PrintWriter;

@WebServlet("/encoded-url")
public class EncodedUrlServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        HttpSession session = request.getSession();

        session.setAttribute("username", "Rahul");

        String dashboardUrl = response.encodeURL(
                request.getContextPath() + "/session-dashboard"
        );

        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();

        out.println("<!DOCTYPE html>");
        out.println("<html><body>");

        out.println("<h1>Servlet URL Encoding</h1>");

        out.println("<p>Click the link to continue.</p>");

        out.println("<a href=\"" + dashboardUrl + "\">");
        out.println("Open Session Dashboard");
        out.println("</a>");

        out.println("</body></html>");
    }
}
```

## 14. Dashboard Servlet — `SessionDashboardServlet.java`

Add this file:

```text
src/main/java/servlet/SessionDashboardServlet.java
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
import java.io.PrintWriter;

@WebServlet("/session-dashboard")
public class SessionDashboardServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        HttpSession session = request.getSession(false);

        if (session == null) {
            response.sendError(
                    HttpServletResponse.SC_UNAUTHORIZED,
                    "No session exists."
            );

            return;
        }

        String username =
                (String) session.getAttribute("username");

        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();

        out.println("<!DOCTYPE html>");
        out.println("<html><body>");

        out.println("<h1>Session Dashboard</h1>");

        out.println("<p>Username: "
                + escapeHtml(username == null ? "Unknown" : username)
                + "</p>");

        out.println("</body></html>");
    }

    private String escapeHtml(String value) {
        return value
                .replace("&", "&amp;")
                .replace("<", "&lt;")
                .replace(">", "&gt;")
                .replace("\"", "&quot;")
                .replace("'", "&#x27;");
    }
}
```

### Test

Open:

```text
http://localhost:8080/session-management-demo/encoded-url
```

Click the dashboard link.

When URL-based session tracking is active, the container may include a `jsessionid` path parameter. When cookie-based tracking is sufficient, the URL may remain unchanged.

To test URL-based tracking specifically, you need an environment where cookies are disabled or not accepted and URL-based tracking is enabled. A browser that already has a valid session cookie may not demonstrate URL rewriting.

**Security note:** Session identifiers in URLs can leak through browser history, copied links, server logs, screenshots, or referrer headers. Cookie-based session tracking over HTTPS is generally preferable.

---

# Part C: Hidden Form Fields

## 15. What Is a Hidden Form Field?

A hidden form field is an HTML input whose type is `hidden`.

Example:

```html
<input type="hidden" name="username" value="Rahul">
```

It is not displayed as a normal input in the page, but its value is submitted when its form is submitted.

A hidden field can pass a value from one form submission to another.

### Important concept

A hidden form field is not a secure storage mechanism.

The user can inspect or modify it through browser developer tools or by crafting an HTTP request.

Never trust a hidden field to prove that a user is an administrator, has paid for an order, or is authorized to perform an operation.

## 16. How Hidden Fields Work

Suppose a student enters their name on the first page.

```text
Name: Rahul
```

The next page includes:

```html
<input type="hidden" name="username" value="Rahul">
```

When the student submits the next form, the browser sends the hidden field with the other form parameters.

The servlet reads it using:

```java
String username = request.getParameter("username");
```

Flow:

```text
First JSP
   |
   | Student enters name
   v
Start Servlet
   |
   | Forward to second JSP
   v
Second JSP
   |
   | Hidden field contains username
   | Student submits next form
   v
Browser sends POST request
   |
   v
Result Servlet
   |
   | Read hidden field
   v
Result JSP
```

The hidden field carries the value only when the relevant form is submitted.

If the user navigates to another page without submitting that form, the hidden field is not automatically sent.

---

## 17. Hidden Form Field Project Structure

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
        │       ├── HiddenStartServlet.java
        │       └── HiddenResultServlet.java
        │
        └── webapp/
            ├── hidden-start.jsp
            ├── hidden-confirm.jsp
            └── hidden-result.jsp
```

---

## 18. Start Servlet — `HiddenStartServlet.java`

Location:

```text
src/main/java/servlet/HiddenStartServlet.java
```

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/hidden-start")
public class HiddenStartServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        request.getRequestDispatcher("/hidden-start.jsp")
               .forward(request, response);
    }
}
```

The servlet forwards the browser's request to the JSP containing the first form.

## 19. First Form — `hidden-start.jsp`

Location:

```text
src/main/webapp/hidden-start.jsp
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hidden Field Example</title>
</head>
<body>

    <h1>Hidden Form Field Example</h1>

    <form action="${pageContext.request.contextPath}/hidden-confirm"
          method="post">

        <label for="username">Enter Your Name:</label>

        <input
            type="text"
            id="username"
            name="username"
            required
        >

        <button type="submit">Continue</button>

    </form>

</body>
</html>
```

When the user enters a name and submits the form, the browser sends a POST request to `/hidden-confirm`.

## 20. Confirmation Servlet

To keep the example focused on form processing, add this servlet:

```text
src/main/java/servlet/HiddenConfirmServlet.java
```

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/hidden-confirm")
public class HiddenConfirmServlet extends HttpServlet {

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        String username = request.getParameter("username");

        if (username == null || username.isBlank()) {
            response.sendError(
                    HttpServletResponse.SC_BAD_REQUEST,
                    "Username is required."
            );

            return;
        }

        request.setAttribute("username", username.trim());

        request.getRequestDispatcher("/hidden-confirm.jsp")
               .forward(request, response);
    }
}
```

This servlet receives the first form submission and forwards the name to the confirmation JSP.

## 21. Confirmation Page — `hidden-confirm.jsp`

Location:

```text
src/main/webapp/hidden-confirm.jsp
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%!
    private String escapeHtml(String value) {
        return value
                .replace("&", "&amp;")
                .replace("<", "&lt;")
                .replace(">", "&gt;")
                .replace("\"", "&quot;")
                .replace("'", "&#x27;");
    }
%>

<%
    String username = (String) request.getAttribute("username");

    if (username == null) {
        response.sendError(
                HttpServletResponse.SC_BAD_REQUEST,
                "Username is missing."
        );

        return;
    }

    String safeUsername = escapeHtml(username);
%>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Confirm Details</title>
</head>
<body>

    <h1>Confirm Your Details</h1>

    <p>Welcome, <%= safeUsername %>!</p>

    <form action="${pageContext.request.contextPath}/hidden-result"
          method="post">

        <input
            type="hidden"
            name="username"
            value="<%= safeUsername %>"
        >

        <label for="course">Choose Your Course:</label>

        <select id="course" name="course" required>
            <option value="">Select a course</option>
            <option value="Java">Java</option>
            <option value="Python">Python</option>
            <option value="Web Development">Web Development</option>
        </select>

        <button type="submit">Submit Details</button>

    </form>

</body>
</html>
```

### Explanation

The hidden field is:

```html
<input type="hidden" name="username" value="Rahul">
```

The field is not shown as a normal text input. However, the browser includes it when the form is submitted.

For the name `Rahul` and course `Java`, the next POST request contains parameters similar to:

```text
username=Rahul
course=Java
```

The `username` value is carried forward through the hidden field.

**Notice:** The value has been escaped for safe HTML output, but escaping does not make a hidden field trustworthy. A user can still change its value.

Also, `HttpServletResponse` must be imported if you use it directly in a JSP scriptlet. To keep the file compiling, add this directive at the top of `hidden-confirm.jsp`:

```jsp
<%@ page import="jakarta.servlet.http.HttpServletResponse" %>
```

---

## 22. Result Servlet — `HiddenResultServlet.java`

Location:

```text
src/main/java/servlet/HiddenResultServlet.java
```

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/hidden-result")
public class HiddenResultServlet extends HttpServlet {

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        String username = request.getParameter("username");
        String course = request.getParameter("course");

        if (username == null || username.isBlank()
                || course == null || course.isBlank()) {

            response.sendError(
                    HttpServletResponse.SC_BAD_REQUEST,
                    "Username and course are required."
            );

            return;
        }

        if (!"Java".equals(course)
                && !"Python".equals(course)
                && !"Web Development".equals(course)) {

            response.sendError(
                    HttpServletResponse.SC_BAD_REQUEST,
                    "Invalid course."
            );

            return;
        }

        request.setAttribute("username", username.trim());
        request.setAttribute("course", course);

        request.getRequestDispatcher("/hidden-result.jsp")
               .forward(request, response);
    }
}
```

### Explanation

```java
String username = request.getParameter("username");
String course = request.getParameter("course");
```

Both values are read from the second form submission.

The servlet validates the course against the allowed choices. In a real application, it should not assume that a submitted value is valid just because it came from a dropdown.

Then the servlet forwards the values to the result JSP.

## 23. Result Page — `hidden-result.jsp`

Location:

```text
src/main/webapp/hidden-result.jsp
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%!
    private String escapeHtml(String value) {
        return value
                .replace("&", "&amp;")
                .replace("<", "&lt;")
                .replace(">", "&gt;")
                .replace("\"", "&quot;")
                .replace("'", "&#x27;");
    }
%>

<%
    String username = (String) request.getAttribute("username");
    String course = (String) request.getAttribute("course");

    if (username == null || course == null) {
        response.sendError(
                HttpServletResponse.SC_BAD_REQUEST,
                "Result data is missing."
        );

        return;
    }

    String safeUsername = escapeHtml(username);
    String safeCourse = escapeHtml(course);
%>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hidden Field Result</title>
</head>
<body>

    <h1>Submission Successful</h1>

    <p>Student Name: <%= safeUsername %></p>

    <p>Selected Course: <%= safeCourse %></p>

    <p>
        The username was carried to the second form using a hidden field.
    </p>

    <a href="${pageContext.request.contextPath}/hidden-start">
        Start Again
    </a>

</body>
</html>
```

Add this directive at the top of `hidden-result.jsp`, because the JSP references `HttpServletResponse`:

```jsp
<%@ page import="jakarta.servlet.http.HttpServletResponse" %>
```

---

# Part D: Testing Hidden Form Fields

## 24. Run the Application

Start Tomcat 10.1 through Smart Tomcat.

Open:

```text
http://localhost:8080/session-management-demo/hidden-start
```

Follow these steps:

1. Enter `Rahul`.
2. Click **Continue**.
3. The confirmation page displays Rahul's name.
4. Choose `Java`.
5. Click **Submit Details**.
6. The result page displays the name and course.

### What happened?

The first form sent the name to `HiddenConfirmServlet`.

That servlet forwarded the value to `hidden-confirm.jsp`.

The JSP placed the name inside a hidden field.

The second form submitted both the hidden username and selected course to `HiddenResultServlet`.

The result servlet displayed the received values.

### Test the limitation

Open browser developer tools and inspect the confirmation page. You can change the hidden field's value before submitting the form.

This demonstrates why hidden fields must not be trusted for authorization or sensitive data.

---

# Part E: Comparison of Session Management Techniques

## 25. URL Rewriting vs Hidden Form Fields vs Cookies vs `HttpSession`

| Feature | URL Rewriting | Hidden Form Field | Cookie | `HttpSession` |
|---|---|---|---|---|
| Where data is carried | URL | HTML form | Browser cookie | Server-side session |
| Visible in URL | Yes, for query parameters or URL session IDs | No | Usually not in URL | Session attributes are not normally in the URL |
| Sent automatically on every matching request | Query parameters only when included in the URL | No, only when the relevant form is submitted | Yes, subject to scope and browser policies | Session ID is typically sent via a cookie |
| Works across ordinary link navigation | If the data is included in the link | No | Usually, if the cookie matches | Usually, if the session ID is sent |
| Can store arbitrary Java objects on server | No | No | No | Yes, as session attributes |
| Main limitation | URLs can be copied, logged, or exposed | Users can inspect and modify fields | Users can inspect and modify ordinary values | Requires lifecycle and security management |
| Typical use | Passing a non-sensitive identifier or URL-based session tracking | Carrying a value through a form workflow | Theme or language preference | Login state or per-user server-side data |

---

# Part F: Important Security Rules

## 26. Rule 1 — Never Pass Passwords Through URLs

Unsafe example:

```text
/login?username=rahul&password=java123
```

URLs can appear in browser history, logs, copied links, and referrer information.

Use HTTPS and POST for credential submission, and follow secure authentication practices.

POST does not, by itself, encrypt data. HTTPS provides transport encryption.

## 27. Rule 2 — Never Trust Hidden Fields

Unsafe example:

```html
<input type="hidden" name="role" value="admin">
```

A user can modify this value.

Always check permissions on the server using trusted authentication and authorization data.

## 28. Rule 3 — Use `encodeURL()` for Session URL Tracking

For a URL-based session-tracking scenario:

```java
String url = response.encodeURL(
        request.getContextPath() + "/dashboard"
);
```

For a redirect destination:

```java
String url = response.encodeRedirectURL(
        request.getContextPath() + "/dashboard"
);

response.sendRedirect(url);
```

Do not manually construct a URL containing a session identifier.

## 29. Rule 4 — Use Server-Side Storage for Trusted State

If a student has logged in, the server should check the user's session or another trusted authentication mechanism.

Do not treat a username, role, price, payment status, or authorization decision supplied through a query parameter or hidden field as trustworthy.

---

# Part G: Interview Questions

## 30. Beginner Questions

**Q1. What is URL rewriting?**

It is a technique for passing information through a URL. Servlet URL encoding can also be used to support session tracking when cookies are unavailable.

**Q2. What is a hidden form field?**

It is an HTML input of type `hidden`. Its value is included when the containing form is submitted, even though it is not shown as a normal input.

**Q3. Can a user modify a hidden field?**

Yes. Hidden fields are not secure storage.

**Q4. What is `response.encodeURL()`?**

It allows the Servlet container to encode a URL for session tracking when needed.

**Q5. Why should we avoid putting sensitive data in URLs?**

URLs may be recorded in browser history, server logs, copied links, or referrer information.

## 31. Intermediate Questions

**Q6. How is query-parameter URL rewriting different from Servlet URL session tracking?**

Query parameters carry application data, such as `username=Rahul`. Servlet URL session tracking uses methods such as `encodeURL()` so the container can include a session identifier when necessary.

**Q7. Does a hidden field preserve data when a user clicks any link?**

No. The value is submitted only when its containing form is submitted, unless application code explicitly transfers it elsewhere.

**Q8. Can URL rewriting replace `HttpSession`?**

Not generally. It can support session tracking, but exposing session identifiers in URLs is risky. `HttpSession` is a server-side storage abstraction, while URL rewriting is one possible way to carry a session identifier between requests.

**Q9. What happens if a user changes `username=Rahul` to `username=Admin` in the URL?**

The server receives the changed parameter. The application must not treat it as proof of identity or permission.

**Q10. Which technique is generally suitable for authenticated user state?**

A server-side session with a protected session identifier, combined with server-side authorization checks, is a common approach for traditional Servlet applications.

---

# Final Summary

- **URL rewriting:** Passes data through URLs; `response.encodeURL()` supports container-managed session tracking.
- **Hidden form fields:** Carry values between form submissions.
- **Cookies:** Store small client-side values and commonly carry a session identifier.
- **`HttpSession`:** Stores server-side data associated with a user's session.

The key principle is that **URL parameters and hidden fields are client-controlled input, not trusted session state**. Validate all submitted values and make authorization decisions on the server.
