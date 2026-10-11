# RequestDispatcher in Servlet

`RequestDispatcher` is used when **one servlet wants to involve another resource** in handling the same request.

The resource can be:

- another Servlet
- JSP
- HTML resource

The two important methods are:

```java
requestDispatcher.forward(request, response);
```

and

```java
requestDispatcher.include(request, response);
```

The easiest way to understand them:

> **`forward()` = “You handle the request from here.”**  
> **`include()` = “Add your output to my output.”**

---

# 1. First understand the problem

Suppose we have:

```text
Browser
   |
   |  /login
   ↓
LoginServlet
```

The user enters:

```text
username = John
password = 1234
```

`LoginServlet` checks the login.

After successful login, we want to show:

```text
Welcome John
Dashboard
Profile
Logout
```

We could write all HTML inside `LoginServlet`.

But that's a bad idea:

```java
out.println("<html>");
out.println("<body>");
out.println("<h1>Welcome John</h1>");
out.println("</body>");
out.println("</html>");
```

Instead, we can have:

```text
LoginServlet
      |
      | forward()
      ↓
DashboardServlet / dashboard.jsp
```

This is where **RequestDispatcher** becomes useful.

---

# 2. What exactly is RequestDispatcher?

`RequestDispatcher` is an object provided by the Servlet API.

It allows a servlet to communicate with another resource **inside the server**.

Basic syntax:

```java
RequestDispatcher rd =
        request.getRequestDispatcher("/dashboard");

rd.forward(request, response);
```

or:

```java
RequestDispatcher rd =
        request.getRequestDispatcher("/header");

rd.include(request, response);
```

---

# 3. Understand the name

Break the word:

```text
Request + Dispatcher
```

### Request

The browser sent a request:

```text
GET /login
```

### Dispatcher

Dispatcher means:

> Decide where this request should be handled next.

So:

```text
Browser
   |
   | request
   ↓
LoginServlet
   |
   | RequestDispatcher
   ↓
DashboardServlet
```

---

# 4. How does it know where to go?

This part is extremely important.

Suppose:

```java
RequestDispatcher rd =
        request.getRequestDispatcher("/dashboard");
```

Here:

```text
/dashboard
```

is the **path of the target resource**.

For example:

```java
@WebServlet("/login")
public class LoginServlet extends HttpServlet {
}
```

and:

```java
@WebServlet("/dashboard")
public class DashboardServlet extends HttpServlet {
}
```

Then:

```java
request.getRequestDispatcher("/dashboard");
```

means:

> Find the resource mapped to `/dashboard`.

So the flow becomes:

```text
Browser
   |
   | /login
   ↓
LoginServlet
   |
   | RequestDispatcher
   | /dashboard
   ↓
DashboardServlet
```

---

# 5. Why not simply use a new URL?

Students often ask:

> Sir, why don't we just redirect?

This is where **forward vs redirect** becomes important.

There are two different concepts:

```text
forward()
redirect()
```

### Forward

```text
Browser
   |
   | /login
   ↓
LoginServlet
   |
   | forward()
   ↓
DashboardServlet
```

The browser doesn't make another request.

### Redirect

```text
Browser
   |
   | /login
   ↓
LoginServlet
   |
   | redirect("/dashboard")
   ↓
Browser
   |
   | NEW REQUEST
   | /dashboard
   ↓
DashboardServlet
```

So:

> **Forward = server-side movement**  
> **Redirect = browser-side new request**

We'll come back to this after understanding `forward()`.

---

# PART A — `forward()`

# 6. What does `forward()` mean?

Suppose:

```text
LoginServlet
```

receives a request.

It checks:

```text
Username correct?
Password correct?
```

If login is successful:

```text
LoginServlet
      |
      | forward()
      ↓
DashboardServlet
```

The LoginServlet says:

> "I have finished my part. From now on, DashboardServlet should handle this request."

That's **forwarding**.

---

# 7. Basic syntax

```java
RequestDispatcher rd =
        request.getRequestDispatcher("/dashboard");

rd.forward(request, response);
```

Two things are passed:

```java
request
response
```

Why?

Because the next resource needs access to the same request and response.

---

# 8. Why pass `request`?

Suppose LoginServlet gets:

```text
username = John
```

We want DashboardServlet to know the username.

We can put it into the request:

```java
request.setAttribute("username", "John");
```

Then forward:

```java
RequestDispatcher rd =
        request.getRequestDispatcher("/dashboard");

rd.forward(request, response);
```

DashboardServlet can retrieve it:

```java
String username =
        (String) request.getAttribute("username");
```

So:

```text
LoginServlet
     |
     | request.setAttribute()
     ↓
Request
     |
     | forward()
     ↓
DashboardServlet
     |
     | request.getAttribute()
     ↓
"John"
```

This is one of the most important uses of `forward()`.

---

# 9. Complete example — Forward

## LoginServlet

```java
import jakarta.servlet.RequestDispatcher;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

import java.io.IOException;

@WebServlet("/login")
public class LoginServlet extends HttpServlet {

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        String username = request.getParameter("username");
        String password = request.getParameter("password");

        if (username.equals("John")
                && password.equals("1234")) {

            request.setAttribute("username", username);

            RequestDispatcher rd =
                    request.getRequestDispatcher("/dashboard");

            rd.forward(request, response);

        } else {

            response.getWriter()
                    .println("Invalid username or password");
        }
    }
}
```

---

## DashboardServlet

```java
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

import java.io.IOException;

@WebServlet("/dashboard")
public class DashboardServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        String username =
                (String) request.getAttribute("username");

        response.setContentType("text/html");

        response.getWriter().println(
                "<h1>Welcome " + username + "</h1>"
        );
    }
}
```

---

# 10. Understand the complete flow

Browser submits:

```text
POST /login
```

↓

Tomcat finds:

```java
@WebServlet("/login")
```

↓

`LoginServlet` executes.

It receives:

```java
username = John
password = 1234
```

↓

Credentials are correct.

↓

We store:

```java
request.setAttribute("username", username);
```

Now the request contains:

```text
Request
------------------
username = John
```

↓

Then:

```java
rd.forward(request, response);
```

↓

Tomcat sends the same request to:

```text
/dashboard
```

↓

`DashboardServlet` receives the same request.

↓

It executes:

```java
request.getAttribute("username");
```

↓

Gets:

```text
John
```

↓

Browser receives:

```text
Welcome John
```

---

# 11. VERY IMPORTANT: URL does NOT change

This is one of the biggest differences between forward and redirect.

Suppose browser initially requests:

```text
http://localhost:8080/MyApp/login
```

Then:

```java
rd.forward(request, response);
```

takes it internally to:

```text
/dashboard
```

But the browser's address bar can still show:

```text
http://localhost:8080/MyApp/login
```

Because the browser was **not asked to make another request**.

Think:

```text
Browser
   |
   | /login
   ↓
Tomcat
   |
   | internally forward
   ↓
DashboardServlet
```

The browser doesn't know about this internal forwarding as a new navigation.

---

# 12. Forward diagram

```text
             ONE REQUEST
Browser ───────────────────────→ LoginServlet
                                  |
                                  |
                                  | forward()
                                  ↓
                            DashboardServlet
                                  |
                                  ↓
                               Response
                                  |
                                  ↓
                               Browser
```

Only **one client request**.

---

# 13. What happens to the request object?

The same request is passed.

```java
rd.forward(request, response);
```

So request attributes can travel:

```text
LoginServlet
     |
     | setAttribute()
     ↓
 Request Object
     |
     | forward
     ↓
DashboardServlet
     |
     | getAttribute()
     ↓
 value
```

Example:

```java
request.setAttribute("name", "John");
```

Then:

```java
request.getAttribute("name");
```

returns:

```text
John
```

---

# 14. What about request parameters?

Request parameters also remain available.

Suppose browser sends:

```text
/login?username=John
```

LoginServlet:

```java
String username =
        request.getParameter("username");
```

Then forward:

```java
rd.forward(request, response);
```

DashboardServlet can also do:

```java
String username =
        request.getParameter("username");
```

because it's still the same request.

---

# PART B — `include()`

Now let's understand `include()`.

`include()` has a completely different purpose.

---

# 15. What does `include()` mean?

Imagine a website has:

```text
--------------------------------
          HEADER
--------------------------------

          MAIN CONTENT

--------------------------------
          FOOTER
--------------------------------
```

Instead of writing the header in every servlet:

```java
out.println("<header>...</header>");
```

we can have a separate resource:

```text
HeaderServlet
```

Then another servlet can say:

> "Give me whatever HeaderServlet produces and include that output in my response."

That's:

```java
rd.include(request, response);
```

---

# 16. Simple diagram

```text
MainServlet
    |
    | include()
    ↓
HeaderServlet
    |
    | output
    ↓
MainServlet response
```

The important difference:

### `forward()`

```text
"Now YOU handle the response."
```

### `include()`

```text
"Give me your output; I'll include it in my response."
```

---

# 17. Simple include example

Suppose:

```java
@WebServlet("/header")
public class HeaderServlet extends HttpServlet {

    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws IOException {

        response.getWriter().println(
                "<h1>My Website</h1>"
        );
    }
}
```

Now:

```java
@WebServlet("/home")
public class HomeServlet extends HttpServlet {

    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws IOException {

        response.setContentType("text/html");

        RequestDispatcher rd =
                request.getRequestDispatcher("/header");

        rd.include(request, response);

        response.getWriter().println(
                "<h2>Welcome to Home Page</h2>"
        );
    }
}
```

Output:

```text
My Website

Welcome to Home Page
```

---

# 18. What actually happened?

Browser:

```text
GET /home
```

↓

`HomeServlet` receives request.

↓

HomeServlet executes:

```java
rd.include(request, response);
```

↓

Tomcat executes:

```text
HeaderServlet
```

↓

HeaderServlet produces:

```html
<h1>My Website</h1>
```

↓

That output gets included in the response.

↓

Control returns to:

```text
HomeServlet
```

↓

HomeServlet continues:

```java
response.getWriter().println(
    "<h2>Welcome to Home Page</h2>"
);
```

Final response:

```html
<h1>My Website</h1>
<h2>Welcome to Home Page</h2>
```

---

# 19. This is the key difference

Look carefully.

### Forward

```java
rd.forward(request, response);

System.out.println("Hello");
```

The code after `forward()` normally does **not continue as normal response generation**, because control has been transferred to the target resource.

Conceptually:

```text
LoginServlet
     |
     | forward()
     ↓
DashboardServlet
     |
     ↓
DONE
```

---

### Include

```java
rd.include(request, response);

System.out.println("Hello");
```

Execution comes back to the original servlet.

```text
HomeServlet
     |
     | include()
     ↓
HeaderServlet
     |
     ↓
BACK
     |
     ↓
HomeServlet continues
```

That's the core difference.

---

# 20. Real-world analogy

Imagine a restaurant.

## Forward

You enter a restaurant and tell waiter:

> "I want to order."

Waiter says:

> "Go to Counter B. They will handle your order from now."

You go to Counter B.

The first waiter is done.

That's:

```text
forward()
```

---

## Include

You order your food.

Waiter says:

> "I'll also ask the chef to add a special side dish."

Chef prepares the side dish.

The waiter adds it to your order.

Then the waiter continues handling your order.

That's:

```text
include()
```

---

# 21. `forward()` vs `include()`

| Feature | `forward()` | `include()` |
|---|---|---|
| Purpose | Transfer handling | Add output |
| Original resource | Gives control away | Continues execution |
| Target resource | Handles response | Contributes output |
| Original servlet continues | Normally no | Yes |
| Same request | Yes | Yes |
| Same response | Yes | Yes |
| Common use | Login → Dashboard | Header/Footer |
| URL changes | No | No |

---

# 22. Very important: Both are server-side

Both:

```java
forward()
```

and:

```java
include()
```

are handled by the server.

They are **not redirects**.

Think:

```text
                Server
          ┌─────────────────┐
Browser → │ LoginServlet    │
          │       ↓         │
          │   forward()     │
          │       ↓         │
          │ DashboardServlet│
          └─────────────────┘
```

The browser doesn't initiate another request.

---

# 23. `sendRedirect()` vs `forward()`

Students should understand this comparison clearly.

### Forward

```java
RequestDispatcher rd =
        request.getRequestDispatcher("/dashboard");

rd.forward(request, response);
```

Flow:

```text
Browser
   |
   | Request #1
   ↓
LoginServlet
   |
   | forward
   ↓
DashboardServlet
   |
   ↓
Browser
```

### Redirect

```java
response.sendRedirect("dashboard");
```

Flow:

```text
Browser
   |
   | Request #1
   ↓
LoginServlet
   |
   | redirect
   ↓
Browser
   |
   | Request #2
   ↓
DashboardServlet
   |
   ↓
Browser
```

So:

```text
forward   → 1 request
redirect  → 2 requests
```

---

# 24. Why is `forward()` useful for login?

Suppose:

```java
if (validLogin) {

    request.setAttribute(
        "username",
        username
    );

    RequestDispatcher rd =
        request.getRequestDispatcher("/dashboard");

    rd.forward(request, response);
}
```

Dashboard can access:

```java
request.getAttribute("username");
```

Very useful when you want to pass data from one server-side component to another.

---

# 25. Why is `include()` useful?

Suppose we have:

```text
header
footer
navbar
menu
common message
```

Instead of repeating code:

```text
HomeServlet
AboutServlet
ContactServlet
ProfileServlet
```

each can include:

```text
Header
Footer
```

Conceptually:

```text
HomeServlet
   |
   ├── include Header
   |
   ├── Home content
   |
   └── include Footer
```

Result:

```text
------------------
HEADER
------------------
HOME CONTENT
------------------
FOOTER
------------------
```

---

# 26. Include multiple resources

You can even do:

```java
RequestDispatcher header =
        request.getRequestDispatcher("/header");

header.include(request, response);

response.getWriter().println(
        "<h1>Home Page</h1>"
);

RequestDispatcher footer =
        request.getRequestDispatcher("/footer");

footer.include(request, response);
```

Flow:

```text
HomeServlet
    |
    +---- include Header
    |
    +---- Home Content
    |
    +---- include Footer
```

---

# 27. Request attributes with include

The request is also available to the included resource.

For example:

```java
request.setAttribute(
        "username",
        "John"
);

RequestDispatcher rd =
        request.getRequestDispatcher("/header");

rd.include(request, response);
```

HeaderServlet:

```java
String username =
        (String) request.getAttribute("username");
```

So:

```text
Original Request
       |
       | attribute
       ↓
   HeaderServlet
```

---

# 28. `getRequestDispatcher()` has two common forms

### Using request

```java
RequestDispatcher rd =
        request.getRequestDispatcher("/dashboard");
```

This is very common.

### Using ServletContext

```java
RequestDispatcher rd =
        getServletContext()
        .getRequestDispatcher("/dashboard");
```

For beginners, focus first on:

```java
request.getRequestDispatcher(...)
```

---

# 29. `forward()` with JSP

This becomes very important in real applications.

Servlet:

```java
request.setAttribute(
        "username",
        "John"
);

RequestDispatcher rd =
        request.getRequestDispatcher("/dashboard.jsp");

rd.forward(request, response);
```

Then JSP can use that data.

Conceptually:

```text
Browser
   ↓
LoginServlet
   ↓
setAttribute()
   ↓
forward()
   ↓
dashboard.jsp
   ↓
HTML
   ↓
Browser
```

This is one reason Servlets and JSP were commonly used together.

---

# 30. Very important beginner distinction

Don't confuse:

```java
request.setAttribute()
```

with:

```java
request.getParameter()
```

### Parameter

Comes from the client request:

```java
request.getParameter("username");
```

Example:

```text
/login?username=John
```

### Attribute

Added by server-side code:

```java
request.setAttribute(
    "username",
    "John"
);
```

Then:

```java
request.getAttribute("username");
```

Think:

```text
Parameter
Client → Server

Attribute
Server → Server
```

And `RequestDispatcher` makes server-side transfer/include particularly useful.

---

# 31. One complete mini-project

Let's build:

```text
/login
/dashboard
/header
/footer
```

## LoginServlet

```java
@WebServlet("/login")
public class LoginServlet extends HttpServlet {

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        String username =
                request.getParameter("username");

        String password =
                request.getParameter("password");

        if (username.equals("John")
                && password.equals("1234")) {

            request.setAttribute(
                    "username",
                    username
            );

            RequestDispatcher rd =
                    request.getRequestDispatcher(
                            "/dashboard"
                    );

            rd.forward(request, response);

        } else {

            response.getWriter().println(
                    "Invalid Login"
            );
        }
    }
}
```

---

## DashboardServlet

```java
@WebServlet("/dashboard")
public class DashboardServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws IOException {

        response.setContentType("text/html");

        RequestDispatcher header =
                request.getRequestDispatcher(
                        "/header"
                );

        header.include(request, response);

        String username =
                (String) request.getAttribute(
                        "username"
                );

        response.getWriter().println(
                "<h1>Welcome "
                + username
                + "</h1>"
        );

        RequestDispatcher footer =
                request.getRequestDispatcher(
                        "/footer"
                );

        footer.include(request, response);
    }
}
```

---

## HeaderServlet

```java
@WebServlet("/header")
public class HeaderServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws IOException {

        response.getWriter().println(
                "<header>"
                + "<h2>My Website</h2>"
                + "<hr>"
                + "</header>"
        );
    }
}
```

---

## FooterServlet

```java
@WebServlet("/footer")
public class FooterServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws IOException {

        response.getWriter().println(
                "<hr>"
                + "<footer>"
                + "Copyright 2026"
                + "</footer>"
        );
    }
}
```

---

# 32. Full flow of this project

```text
                 Browser
                    |
                    | POST /login
                    ↓
              LoginServlet
                    |
             check username
             check password
                    |
                    | setAttribute()
                    |
                    | forward()
                    ↓
             DashboardServlet
                    |
                    | include()
                    ↓
               HeaderServlet
                    |
                    | output
                    ↓
             DashboardServlet
                    |
                    | dashboard output
                    |
                    | include()
                    ↓
               FooterServlet
                    |
                    | output
                    ↓
                 Browser
```

This single example teaches both methods beautifully.

---

# 33. The one-line memory trick

Tell students to remember:

```text
forward()  → "You take over."
include()  → "You contribute."
```

Or:

```text
FORWARD
F = Full responsibility moves Forward
```

```text
INCLUDE
I = Include your output Inside my response
```

---

# 34. Interview questions students should know

### Q1. What is RequestDispatcher?

It is a Servlet API interface used to dispatch a request to another server-side resource using methods such as `forward()` and `include()`.

### Q2. What is `forward()`?

It transfers request processing to another resource on the server.

### Q3. What is `include()`?

It includes the output generated by another resource in the current response.

### Q4. Does URL change with forward?

No.

### Q5. Does URL change with include?

No.

### Q6. Does forward create a new request?

No.

### Q7. Does include create a new request?

No.

### Q8. Can request attributes be passed with forward?

Yes.

### Q9. Common use of forward?

```text
Servlet → JSP
LoginServlet → Dashboard
Controller → View
```

### Q10. Common use of include?

```text
Header
Footer
Navbar
Common content
```

---

# 35. Final mental model

If students remember only this diagram, they can understand the whole topic:

```text
                 REQUEST
                    ↓
              LoginServlet
                    |
             ┌──────┴──────┐
             │             │
          forward()      include()
             │             │
             ↓             ↓
       Dashboard       Header
       takes over      gives output
             │             │
             ↓             ↓
          Response      Back to
                         original
                         servlet
```

### In one sentence:

**`forward()` changes who handles the request; `include()` adds another resource's output while the original resource continues.**
