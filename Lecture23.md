# Configuring Error Pages in Servlet — From Beginner to Deep Understanding

## 1. First: What is an Error Page?

Suppose a user opens:

```text
http://localhost:8080/MyApp/login
```

and your Servlet has a problem.

For example:

```java
int a = 10 / 0;
```

This causes an exception:

```text
ArithmeticException: / by zero
```

Without proper error handling, the server may show a technical error page containing information that the **normal user doesn't need to see**.

For example:

```text
HTTP Status 500 – Internal Server Error

java.lang.ArithmeticException: / by zero
    at com.example.LoginServlet.doGet(...)
    ...
```

That's not a good user experience.

Instead, we can configure our application to show:

```text
Something went wrong!

Please try again later.
```

This is called an **error page**.

---

# 2. Why Do We Need Error Pages?

Imagine you're building a real website.

A user encounters:

```text
404
```

What does that mean?

It generally means:

> The requested resource could not be found.

For example:

```text
http://localhost:8080/MyApp/hello
```

But suppose `/hello` doesn't exist.

The server may generate a 404 error.

Instead of showing a boring technical page, we can create:

```text
404 - Page Not Found

Sorry, the page you're looking for doesn't exist.
```

Similarly:

### 500

Server-side error.

```text
500 - Internal Server Error
```

### 404

Resource doesn't exist.

```text
404 - Page Not Found
```

### 403

Access is forbidden.

```text
403 - Access Denied
```

So error-page configuration allows us to provide **custom responses for errors**.

---

# 3. Important Terminology

Before writing code, understand these four terms.

### 1. Error

Something went wrong while processing the request.

Example:

```java
int x = 10 / 0;
```

---

### 2. Exception

An exception is an event that occurs during program execution that disrupts normal execution.

Example:

```java
ArithmeticException
NullPointerException
IOException
```

---

### 3. HTTP Status Code

The server sends a status code to the browser.

Examples:

```text
200 → Success
404 → Not Found
500 → Internal Server Error
403 → Forbidden
```

---

### 4. Error Page

The page displayed to the user when an error occurs.

Example:

```text
error.jsp
```

---

# 4. How Servlet Error Handling Works

The basic flow is:

```text
Browser
   |
   | Request
   ↓
Servlet
   |
   | Something goes wrong
   ↓
Server detects error
   |
   ↓
web.xml configuration
   |
   ↓
Custom Error Page
   |
   ↓
Browser
```

The most important thing here is:

> **web.xml can tell the server which page should be displayed for a particular error.**

---

# 5. What is `web.xml`?

If you've never used it before, don't worry.

`web.xml` is called the:

> **Deployment Descriptor**

It contains configuration information for a web application.

For example, we can configure:

- Servlets
- URL mappings
- Welcome pages
- Error pages
- Session configuration
- Filters
- Security configuration

A typical project may contain:

```text
MyWebApplication
│
├── src
│   └── ...
│
└── WebContent
    ├── index.html
    └── WEB-INF
        └── web.xml
```

Or in Maven-style projects:

```text
MyWebApplication
│
├── src
│   └── main
│       ├── java
│       └── webapp
│           └── WEB-INF
│               └── web.xml
```

---

# 6. First Create an Error Page

Let's create:

```text
error.jsp
```

Example:

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Error</title>
</head>
<body>

    <h1>Something Went Wrong!</h1>

    <p>Please try again later.</p>

</body>
</html>
```

Very simple.

The browser will display:

```text
Something Went Wrong!

Please try again later.
```

---

# 7. Now Configure `web.xml`

We can configure an error page using:

```xml
<error-page>
    
</error-page>
```

Let's write:

```xml
<error-page>
    <error-code>500</error-code>
    <location>/error.jsp</location>
</error-page>
```

Now let's understand **every single line**.

---

# 8. Line-by-Line Explanation

### Line 1

```xml
<error-page>
```

This tells the server:

> "I am going to configure an error page."

`error-page` is an XML configuration element.

---

### Line 2

```xml
<error-code>500</error-code>
```

This says:

> "When HTTP status code 500 occurs, use this configuration."

Remember:

```text
500 = Internal Server Error
```

So:

```xml
<error-code>500</error-code>
```

means:

```text
If error code = 500
        ↓
show configured error page
```

---

### Line 3

```xml
<location>/error.jsp</location>
```

This tells the server:

> "The page that should be displayed is `/error.jsp`."

So the server knows:

```text
500
 ↓
error.jsp
```

---

### Line 4

```xml
</error-page>
```

This closes the configuration.

---

# 9. Complete `web.xml`

For a modern Servlet project, you might have:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee
         https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
         version="6.0">

    <error-page>
        <error-code>500</error-code>
        <location>/error.jsp</location>
    </error-page>

</web-app>
```

**Note:** If you're learning with an older Tomcat/Servlet version, your `web.xml` may use the older `javax`-era schema. The error-page concept remains the same.

---

# 10. Now Let's Create a Servlet That Produces an Error

Suppose we have:

```java
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/test")
public class TestServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws ServletException, IOException {

        int a = 10;
        int b = 0;

        int result = a / b;

        response.getWriter().println(result);
    }
}
```

Now look carefully.

This line:

```java
int result = a / b;
```

becomes:

```java
10 / 0
```

which causes:

```text
ArithmeticException
```

---

# 11. What Happens Internally?

The user requests:

```text
/test
```

The server sends the request to:

```text
TestServlet
```

The Servlet executes:

```java
int a = 10;
int b = 0;
```

Then:

```java
int result = a / b;
```

Java throws:

```text
ArithmeticException
```

The request processing fails.

The container generates an HTTP 500 error.

Because we configured:

```xml
<error-page>
    <error-code>500</error-code>
    <location>/error.jsp</location>
</error-page>
```

the container can route the error to:

```text
error.jsp
```

So the user sees your custom page.

---

# 12. The Complete Flow

Remember this diagram:

```text
              Browser
                 |
                 | GET /test
                 ↓
           TestServlet
                 |
                 |
          10 / 0 happens
                 |
                 ↓
       ArithmeticException
                 |
                 ↓
          HTTP 500 Error
                 |
                 ↓
             web.xml
                 |
                 ↓
          error-code = 500
                 |
                 ↓
            error.jsp
                 |
                 ↓
              Browser
```

This is the **core concept**.

---

# 13. Configuring 404 Error

Now let's handle another very common error.

```text
404
```

means:

> Requested resource was not found.

We can configure:

```xml
<error-page>
    <error-code>404</error-code>
    <location>/404.jsp</location>
</error-page>
```

Now create:

```text
404.jsp
```

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Page Not Found</title>
</head>
<body>

    <h1>404 - Page Not Found</h1>

    <p>Sorry, the page you requested could not be found.</p>

</body>
</html>
```

Now:

```text
Wrong URL
    ↓
404
    ↓
404.jsp
```

---

# 14. Configure Multiple Error Pages

You can configure several error pages.

For example:

```xml
<error-page>
    <error-code>404</error-code>
    <location>/404.jsp</location>
</error-page>

<error-page>
    <error-code>500</error-code>
    <location>/500.jsp</location>
</error-page>

<error-page>
    <error-code>403</error-code>
    <location>/403.jsp</location>
</error-page>
```

Now:

```text
404 → 404.jsp

500 → 500.jsp

403 → 403.jsp
```

---

# 15. What About Exceptions?

This is where Servlet error configuration becomes more interesting.

Instead of specifying an HTTP status code:

```xml
<error-code>500</error-code>
```

we can specify an exception:

```xml
<exception-type>java.lang.ArithmeticException</exception-type>
```

Example:

```xml
<error-page>

    <exception-type>
        java.lang.ArithmeticException
    </exception-type>

    <location>/arithmetic-error.jsp</location>

</error-page>
```

This means:

> If an `ArithmeticException` occurs, forward the request to `arithmetic-error.jsp`.

---

# 16. Why Use Exception Type?

Suppose you have:

```java
int x = 10 / 0;
```

Java generates:

```text
ArithmeticException
```

The server can recognize that exception.

Then:

```text
ArithmeticException
        ↓
arithmetic-error.jsp
```

This allows you to have different pages for different exception types.

---

# 17. Example: `NullPointerException`

Suppose:

```java
String name = null;

System.out.println(name.length());
```

This causes:

```text
NullPointerException
```

We could configure:

```xml
<error-page>

    <exception-type>
        java.lang.NullPointerException
    </exception-type>

    <location>/null-error.jsp</location>

</error-page>
```

Now:

```text
NullPointerException
        ↓
null-error.jsp
```

---

# 18. Exception Configuration vs Error-Code Configuration

This is extremely important.

### Error code

```xml
<error-page>
    <error-code>404</error-code>
    <location>/404.jsp</location>
</error-page>
```

We're saying:

> "If HTTP status 404 occurs, use this page."

---

### Exception

```xml
<error-page>
    <exception-type>
        java.lang.ArithmeticException
    </exception-type>

    <location>/arithmetic-error.jsp</location>
</error-page>
```

We're saying:

> "If this particular Java exception occurs, use this page."

---

# 19. You Can Configure Both

For example:

```xml
<web-app ...>

    <error-page>

        <error-code>404</error-code>

        <location>/404.jsp</location>

    </error-page>


    <error-page>

        <error-code>500</error-code>

        <location>/500.jsp</location>

    </error-page>


    <error-page>

        <exception-type>
            java.lang.ArithmeticException
        </exception-type>

        <location>/arithmetic-error.jsp</location>

    </error-page>

</web-app>
```

---

# 20. What is `<location>` Exactly?

This is another important concept.

Suppose your application looks like:

```text
MyApp
│
├── index.jsp
│
├── error.jsp
│
└── WEB-INF
    └── web.xml
```

Then:

```xml
<location>/error.jsp</location>
```

means:

> Start from the root of the web application and locate `error.jsp`.

The `/` here is **application context root**, not necessarily your computer's filesystem root.

So don't write:

```xml
<location>C:\Users\John\Desktop\error.jsp</location>
```

❌ Wrong.

Instead:

```xml
<location>/error.jsp</location>
```

✅ Correct.

---

# 21. Can the Error Page Be JSP?

Yes.

For example:

```xml
<location>/error.jsp</location>
```

---

# 22. Can It Be HTML?

Yes.

For example:

```xml
<location>/error.html</location>
```

---

# 23. Can It Be a Servlet?

Yes.

For example:

```xml
<location>/errorHandler</location>
```

If `/errorHandler` is mapped to a Servlet, the Servlet can generate the response.

For example:

```java
@WebServlet("/errorHandler")
public class ErrorHandlerServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {

        response.setContentType("text/html");

        response.getWriter().println(
            "<h1>Something went wrong!</h1>"
        );
    }
}
```

Then:

```xml
<error-page>
    <error-code>500</error-code>
    <location>/errorHandler</location>
</error-page>
```

---

# 24. Important: Error Page Is Usually Server-Side Forwarded

When an error occurs, the container can dispatch the request to the configured error resource.

Conceptually:

```text
Client
   |
   | /test
   ↓
TestServlet
   |
   | Error
   ↓
Servlet Container
   |
   | Error dispatch
   ↓
error.jsp
```

The important idea is:

> The container handles the error and dispatches to the configured error resource.

---

# 25. Can the Error Page Know What Error Happened?

Yes.

Servlet provides special request attributes for error handling.

This is very useful.

For example:

```jsp
<%= request.getAttribute(
    "jakarta.servlet.error.status_code"
) %>
```

can provide the error status code in Jakarta Servlet environments.

There are several standard error attributes.

---

# 26. Important Error Attributes

The Servlet specification provides attributes such as:

```text
jakarta.servlet.error.status_code
```

Status code.

---

```text
jakarta.servlet.error.exception_type
```

Exception class.

---

```text
jakarta.servlet.error.message
```

Error message.

---

```text
jakarta.servlet.error.exception
```

Actual exception object.

---

```text
jakarta.servlet.error.request_uri
```

The URI that caused the error.

---

```text
jakarta.servlet.error.servlet_name
```

The Servlet that was processing the request.

---

# 27. Very Important Version Difference

If you're using an older Java EE / Servlet environment, you may see:

```text
javax.servlet.error.status_code
```

instead of:

```text
jakarta.servlet.error.status_code
```

This is because Servlet APIs moved from the `javax.*` namespace to `jakarta.*`.

So:

### Older Servlet versions

```text
javax.servlet.*
```

### Modern Jakarta Servlet versions

```text
jakarta.servlet.*
```

The exact namespace depends on your Tomcat/Servlet version.

---

# 28. Create a Better Error JSP

For modern Jakarta Servlet:

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Error</title>
</head>
<body>

    <h1>Something Went Wrong</h1>

    <p>
        Status Code:
        <%= request.getAttribute(
            "jakarta.servlet.error.status_code"
        ) %>
    </p>

    <p>
        Request:
        <%= request.getAttribute(
            "jakarta.servlet.error.request_uri"
        ) %>
    </p>

</body>
</html>
```

Now the page can display useful information.

---

# 29. Understanding `request.getAttribute()`

Let's understand this carefully.

You already know:

```java
request
```

represents the client's request.

And:

```java
getAttribute()
```

retrieves information stored inside the request.

For example:

```java
request.getAttribute("name");
```

means:

> "Give me the value associated with the attribute named `name`."

In an error dispatch, the Servlet container automatically places certain error information into the request.

So:

```jsp
request.getAttribute(
    "jakarta.servlet.error.status_code"
)
```

asks:

> "Give me the error status code associated with this error request."

---

# 30. A Real Mini Project

Let's create a complete example.

### Project

```text
ServletErrorDemo
│
├── src
│   └── main
│       ├── java
│       │   └── TestServlet.java
│       │
│       └── webapp
│           ├── error.jsp
│           ├── 404.jsp
│           └── WEB-INF
│               └── web.xml
```

---

# 31. `TestServlet.java`

```java
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/test")
public class TestServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws ServletException, IOException {

        int a = 10;
        int b = 0;

        int result = a / b;

        response.getWriter().println(result);
    }
}
```

---

# 32. `error.jsp`

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Server Error</title>
</head>

<body>

    <h1>500 - Internal Server Error</h1>

    <p>
        Sorry, something went wrong on the server.
    </p>

    <p>
        Please try again later.
    </p>

</body>
</html>
```

---

# 33. `404.jsp`

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Page Not Found</title>
</head>

<body>

    <h1>404 - Page Not Found</h1>

    <p>
        Sorry, the requested page does not exist.
    </p>

</body>
</html>
```

---

# 34. `web.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee
         https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
         version="6.0">

    <error-page>

        <error-code>404</error-code>

        <location>/404.jsp</location>

    </error-page>


    <error-page>

        <error-code>500</error-code>

        <location>/error.jsp</location>

    </error-page>

</web-app>
```

---

# 35. Test 404

Run the application.

Suppose your application is:

```text
http://localhost:8080/ServletErrorDemo
```

Try:

```text
http://localhost:8080/ServletErrorDemo/abc
```

There is no `/abc`.

Therefore:

```text
404
 ↓
404.jsp
```

---

# 36. Test 500

Now open:

```text
http://localhost:8080/ServletErrorDemo/test
```

Servlet executes:

```java
int a = 10;
int b = 0;

int result = a / b;
```

This generates:

```text
ArithmeticException
```

which results in a server error.

Then:

```text
500
 ↓
error.jsp
```

---

# 37. Exception-Based Configuration

Now let's specifically handle `ArithmeticException`.

```xml
<error-page>

    <exception-type>
        java.lang.ArithmeticException
    </exception-type>

    <location>/arithmetic-error.jsp</location>

</error-page>
```

Create:

```text
arithmetic-error.jsp
```

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Calculation Error</title>
</head>

<body>

    <h1>Calculation Error</h1>

    <p>
        A mathematical error occurred.
    </p>

</body>
</html>
```

Now the flow becomes:

```text
10 / 0
  ↓
ArithmeticException
  ↓
ArithmeticException configuration
  ↓
arithmetic-error.jsp
```

---

# 38. Three Main Ways to Configure Error Pages

This is the part I recommend memorizing.

## Method 1 — HTTP Status Code

```xml
<error-page>

    <error-code>404</error-code>

    <location>/404.jsp</location>

</error-page>
```

Meaning:

```text
404 → 404.jsp
```

---

## Method 2 — Exception Type

```xml
<error-page>

    <exception-type>
        java.lang.NullPointerException
    </exception-type>

    <location>/null-error.jsp</location>

</error-page>
```

Meaning:

```text
NullPointerException
        ↓
null-error.jsp
```

---

## Method 3 — General Error Handling

You can configure a general error mechanism using an exception type such as:

```text
java.lang.Throwable
```

For example:

```xml
<error-page>

    <exception-type>
        java.lang.Throwable
    </exception-type>

    <location>/general-error.jsp</location>

</error-page>
```

`Throwable` is the broad superclass for Java's exceptions and errors, so this acts as a broad fallback for throwable conditions.

---

# 39. Why Is `Throwable` Important?

Java's hierarchy roughly looks like:

```text
Throwable
│
├── Error
│
└── Exception
     │
     ├── RuntimeException
     │    ├── NullPointerException
     │    ├── ArithmeticException
     │    └── ...
     │
     └── Other Exceptions
```

Therefore:

```text
Throwable
```

is very broad.

It's useful when you want a general fallback error page.

---

# 40. What Should You Remember for Interviews?

If someone asks:

> **How do you configure error pages in Servlet?**

A strong answer is:

> Error pages in a Servlet-based web application can be configured using the deployment descriptor `web.xml`. We can map an HTTP error status code such as 404 or 500 to a specific resource using `<error-code>`, or map a Java exception such as `NullPointerException` to a specific resource using `<exception-type>`. When the configured error occurs, the Servlet container dispatches the request to the configured error resource.

---

# 41. Most Important `web.xml` Structure

Memorize this:

```xml
<error-page>

    <error-code>500</error-code>

    <location>/error.jsp</location>

</error-page>
```

And:

```xml
<error-page>

    <exception-type>
        java.lang.Exception
    </exception-type>

    <location>/error.jsp</location>

</error-page>
```

That's the foundation.

---

# 42. One Very Important Difference

Don't confuse:

```text
HTTP Error
```

with:

```text
Java Exception
```

For example:

### 404

```text
HTTP Status Code
```

not necessarily a Java exception.

Whereas:

```text
NullPointerException
```

is a:

```text
Java Exception
```

So:

```xml
<error-code>
```

is for HTTP status codes.

And:

```xml
<exception-type>
```

is for Java exception classes.

---

# 43. Final Mental Model

If you remember only one diagram from this lecture, remember this:

```text
                         REQUEST
                            |
                            ↓
                         SERVLET
                            |
                 ┌──────────┴──────────┐
                 |                     |
              SUCCESS                ERROR
                 |                     |
                 ↓                     ↓
             HTTP 200            Error occurs
                                       |
                         ┌─────────────┴─────────────┐
                         |                           |
                    HTTP ERROR                 JAVA EXCEPTION
                         |                           |
                    404 / 500              NullPointerException
                         |                  ArithmeticException
                         |                           |
                         ↓                           ↓
                    error-code              exception-type
                         |                           |
                         └─────────────┬─────────────┘
                                       ↓
                                  web.xml
                                       |
                                       ↓
                                Error Resource
                                       |
                                       ↓
                                   error.jsp
                                       |
                                       ↓
                                    Browser
```

### The three things to remember:

**1. `web.xml`** → configuration

**2. `<error-code>`** → HTTP status code

**3. `<exception-type>`** → Java exception

And both ultimately point to:

```xml
<location>/some-error-page.jsp</location>
```

That is the complete foundation of **Servlet Error Page Configuration**.
