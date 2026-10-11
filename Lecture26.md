# ServletConfig — From Zero to Deep Understanding

## 1. First understand the problem

Suppose we create a servlet:

```java
@WebServlet("/login")
public class LoginServlet extends HttpServlet {

    protected void doPost(HttpServletRequest request,
                          HttpServletResponse response) {

        // login logic
    }
}
```

Now imagine this servlet needs some **configuration information**.

For example:

```text
Admin email = admin@pwioi.com
Maximum login attempts = 3
Company name = PW IOI
Database URL = jdbc:mysql://localhost:3306/student
```

We *could* directly write:

```java
String adminEmail = "admin@pwioi.com";
```

But that's not a good approach.

Why?

Because configuration should ideally be **separate from business logic**.

So Servlet provides an object called:

# `ServletConfig`

Think of it as:

> **ServletConfig = configuration information specifically belonging to one servlet.**

---

# 2. Simple real-life example

Imagine a college has 10 teachers.

Each teacher gets their own ID card.

```text
Teacher A → ID Card A
Teacher B → ID Card B
Teacher C → ID Card C
```

The information on Teacher A's ID card is primarily related to Teacher A.

Similarly:

```text
LoginServlet → LoginServlet's ServletConfig
PaymentServlet → PaymentServlet's ServletConfig
StudentServlet → StudentServlet's ServletConfig
```

Each servlet can have its **own configuration**.

---

# 3. Where does ServletConfig come from?

When the web container starts your servlet, the container creates the servlet object.

For example:

```text
Tomcat
   ↓
Creates LoginServlet object
   ↓
Creates ServletConfig
   ↓
Associates ServletConfig with LoginServlet
   ↓
Calls init()
```

Conceptually:

```text
Tomcat
   |
   | creates
   ↓
LoginServlet
   |
   | has
   ↓
ServletConfig
```

The servlet can access its configuration through:

```java
getServletConfig()
```

---

# 4. Who creates ServletConfig?

**The servlet container creates it.**

For example:

```text
Tomcat
```

Tomcat is responsible for managing your servlet.

You generally don't manually create:

```java
new ServletConfig();
```

In fact, `ServletConfig` is an interface, so you don't normally instantiate it directly.

---

# 5. ServletConfig is an interface

The Servlet API provides:

```java
public interface ServletConfig
```

It provides methods that allow a servlet to access its configuration.

Important methods include:

```java
getServletName()
getServletContext()
getInitParameter()
getInitParameterNames()
```

The most important one for learning ServletConfig is:

```java
getInitParameter()
```

---

# 6. What is an init parameter?

Suppose we want to give our servlet this configuration:

```text
adminEmail = admin@pwioi.com
```

We can define it in `web.xml`:

```xml
<servlet>
    <servlet-name>LoginServlet</servlet-name>

    <servlet-class>com.example.LoginServlet</servlet-class>

    <init-param>
        <param-name>adminEmail</param-name>
        <param-value>admin@pwioi.com</param-value>
    </init-param>
</servlet>
```

Here:

```xml
<param-name>adminEmail</param-name>
```

is the **name**.

And:

```xml
<param-value>admin@pwioi.com</param-value>
```

is the **value**.

Think:

```text
adminEmail
    ↓
admin@pwioi.com
```

This is called a **Servlet initialization parameter**.

---

# 7. Why is it called "initialization" parameter?

Because this information is provided when the servlet is initialized.

The lifecycle is roughly:

```text
Tomcat starts
     ↓
Servlet configuration is read
     ↓
Servlet object created
     ↓
Servlet initialized
     ↓
init()
     ↓
Servlet ready to receive requests
```

So:

```text
init parameter
      ↓
parameter available during servlet initialization
```

---

# 8. Complete example

Let's create:

```text
LoginServlet
```

### `web.xml`

```xml
<web-app>

    <servlet>

        <servlet-name>LoginServlet</servlet-name>

        <servlet-class>
            com.example.LoginServlet
        </servlet-class>

        <init-param>

            <param-name>
                adminEmail
            </param-name>

            <param-value>
                admin@pwioi.com
            </param-value>

        </init-param>

    </servlet>

    <servlet-mapping>

        <servlet-name>
            LoginServlet
        </servlet-name>

        <url-pattern>
            /login
        </url-pattern>

    </servlet-mapping>

</web-app>
```

And Java:

```java
package com.example;

import jakarta.servlet.ServletConfig;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

public class LoginServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                          HttpServletResponse response)
            throws ServletException, IOException {

        ServletConfig config = getServletConfig();

        String adminEmail =
                config.getInitParameter("adminEmail");

        response.getWriter().println(
                "Admin Email: " + adminEmail
        );
    }
}
```

When we visit:

```text
/login
```

we get:

```text
Admin Email: admin@pwioi.com
```

---

# 9. Let's understand this line deeply

```java
ServletConfig config = getServletConfig();
```

Break it down.

### `ServletConfig`

This is the interface/type.

```java
ServletConfig
```

### `config`

This is our variable.

```java
config
```

### `getServletConfig()`

This method is inherited from the Servlet API.

It gives us the configuration associated with **this servlet**.

So:

```java
ServletConfig config = getServletConfig();
```

means:

> "Give me the configuration object associated with this servlet and store it in `config`."

---

# 10. Then we call `getInitParameter()`

```java
String adminEmail =
        config.getInitParameter("adminEmail");
```

Think of it like asking:

```text
ServletConfig
      |
      | "Give me the value of adminEmail"
      ↓
admin@pwioi.com
```

The string:

```java
"adminEmail"
```

must match:

```xml
<param-name>adminEmail</param-name>
```

Then:

```java
getInitParameter("adminEmail")
```

returns:

```text
admin@pwioi.com
```

---

# 11. What if the parameter doesn't exist?

Suppose we write:

```java
String email =
        config.getInitParameter("xyz");
```

but there is no:

```xml
<param-name>xyz</param-name>
```

Then:

```java
getInitParameter("xyz")
```

returns:

```java
null
```

It doesn't automatically create the parameter.

---

# 12. Multiple parameters

We can have multiple configuration values.

```xml
<init-param>
    <param-name>adminEmail</param-name>
    <param-value>admin@pwioi.com</param-value>
</init-param>

<init-param>
    <param-name>maxAttempts</param-name>
    <param-value>3</param-value>
</init-param>

<init-param>
    <param-name>companyName</param-name>
    <param-value>PW IOI</param-value>
</init-param>
```

Then:

```java
ServletConfig config = getServletConfig();

String email =
        config.getInitParameter("adminEmail");

String attempts =
        config.getInitParameter("maxAttempts");

String company =
        config.getInitParameter("companyName");
```

Notice something important:

`getInitParameter()` returns a **String**.

Even:

```xml
<param-value>3</param-value>
```

comes back as:

```java
"3"
```

not:

```java
3
```

So if we need an integer:

```java
int maxAttempts =
        Integer.parseInt(
                config.getInitParameter("maxAttempts")
        );
```

---

# 13. Why use ServletConfig instead of hardcoding?

Without configuration:

```java
String adminEmail = "admin@pwioi.com";
```

If the email changes:

```text
admin@pwioi.com
        ↓
admin@example.com
```

we have to modify Java code.

With configuration:

```xml
<param-value>admin@example.com</param-value>
```

our Java code remains:

```java
String adminEmail =
        config.getInitParameter("adminEmail");
```

This gives us **separation between configuration and application logic**.

---

# 14. ServletConfig can be used with annotations too

Modern Servlet applications don't necessarily require `web.xml`.

We can use:

```java
@WebServlet(
    value = "/login",
    initParams = {
        @WebInitParam(
            name = "adminEmail",
            value = "admin@pwioi.com"
        )
    }
)
public class LoginServlet extends HttpServlet {
    
}
```

Imports:

```java
import jakarta.servlet.annotation.WebInitParam;
import jakarta.servlet.annotation.WebServlet;
```

Then:

```java
@Override
protected void doGet(HttpServletRequest request,
                      HttpServletResponse response)
        throws ServletException, IOException {

    ServletConfig config = getServletConfig();

    String email =
            config.getInitParameter("adminEmail");

    response.getWriter().println(email);
}
```

So there are two common ways:

### XML

```text
web.xml
   ↓
<init-param>
```

### Annotation

```text
@WebServlet
   ↓
@WebInitParam
```

Both provide servlet-specific initialization parameters.

---

# 15. Multiple servlets = different configurations

This is the **most important concept**.

Suppose we have:

```text
LoginServlet
PaymentServlet
StudentServlet
```

We can have:

```text
LoginServlet
   ↓
ServletConfig
   ↓
adminEmail = admin@pwioi.com
```

and:

```text
PaymentServlet
   ↓
ServletConfig
   ↓
currency = INR
```

and:

```text
StudentServlet
   ↓
ServletConfig
   ↓
pageSize = 20
```

So:

```text
LoginServlet
     ↓
its ServletConfig

PaymentServlet
     ↓
its ServletConfig

StudentServlet
     ↓
its ServletConfig
```

This is why we say:

> **ServletConfig is servlet-specific.**

---

# 16. Very important distinction

Don't think:

```text
ServletConfig = one configuration for the entire application
```

Instead:

```text
Servlet A → Config A

Servlet B → Config B

Servlet C → Config C
```

Each servlet can have its own initialization parameters.

---

# 17. What about `init()`?

Now we reach an important advanced point.

A servlet can read its configuration during initialization.

For example:

```java
@Override
public void init(ServletConfig config)
        throws ServletException {

    String email =
            config.getInitParameter("adminEmail");

    System.out.println(email);
}
```

The container calls:

```java
init(config)
```

during servlet initialization.

Conceptually:

```text
Tomcat
   ↓
Creates servlet
   ↓
Creates ServletConfig
   ↓
Passes config to init()
   ↓
init(config)
   ↓
Servlet ready
```

---

# 18. But there is an important beginner trap

You might write:

```java
@Override
public void init(ServletConfig config)
        throws ServletException {

    System.out.println(
        config.getInitParameter("adminEmail")
    );
}
```

That's valid.

But if you override `init(ServletConfig)` directly, you should understand the servlet lifecycle carefully.

`HttpServlet` ultimately relies on the inherited initialization behavior. A common safe pattern when overriding it is:

```java
@Override
public void init(ServletConfig config)
        throws ServletException {

    super.init(config);

    String email =
            config.getInitParameter("adminEmail");

    System.out.println(email);
}
```

Or, often simpler, use the no-argument `init()`:

```java
@Override
public void init() throws ServletException {

    ServletConfig config = getServletConfig();

    String email =
            config.getInitParameter("adminEmail");

    System.out.println(email);
}
```


---

# 19. ServletConfig methods

There are four major methods to remember.

### 1. `getServletName()`

```java
String name = config.getServletName();
```

Returns the configured servlet name.

For example:

```text
LoginServlet
```

---

### 2. `getInitParameter()`

```java
String value =
        config.getInitParameter("adminEmail");
```

Gets one initialization parameter.

---

### 3. `getInitParameterNames()`

```java
Enumeration<String> names =
        config.getInitParameterNames();
```

Gets all initialization parameter names.

For example:

```text
adminEmail
maxAttempts
companyName
```

We can iterate:

```java
Enumeration<String> names =
        config.getInitParameterNames();

while (names.hasMoreElements()) {

    String name = names.nextElement();

    String value =
            config.getInitParameter(name);

    System.out.println(
            name + " = " + value
    );
}
```

---

### 4. `getServletContext()`

This one is extremely important because it connects us to the next topic.

```java
ServletContext context =
        config.getServletContext();
```

This means:

> "From this servlet's configuration, give me the ServletContext of the web application."

And now we start moving from:

```text
ServletConfig
```

to:

```text
ServletContext
```

---

# 20. One picture to remember

Think about your application:

```text
                    TOMCAT
                      |
          ---------------------------
          |            |            |
          ↓            ↓            ↓
     LoginServlet  PaymentServlet StudentServlet
          |            |            |
          ↓            ↓            ↓
     Config A       Config B       Config C
          |            |            |
          ↓            ↓            ↓
     ServletConfig ServletConfig ServletConfig
```

But all of these servlets belong to the same web application:

```text
             ONE WEB APPLICATION
                     |
                     ↓
              ServletContext
             /       |       \
            /        |        \
       Login      Payment    Student
        Config     Config     Config
```

This leads to the fundamental difference:

```text
ServletConfig
      ↓
ONE servlet

ServletContext
      ↓
ENTIRE web application
```

---

# 21. The easiest analogy for students

Imagine a **college**.

```text
College
   |
   |--- Teacher A
   |      ↓
   |   Teacher A's personal information
   |
   |--- Teacher B
   |      ↓
   |   Teacher B's personal information
   |
   |--- Teacher C
          ↓
       Teacher C's personal information
```

Teacher-specific information:

```text
ServletConfig
```

College-wide information:

```text
ServletContext
```

So:

```text
ServletConfig
= "What is special/configured for ME?"

ServletContext
= "What is available/shared for the whole APPLICATION?"
```

---

# 22. One very important rule

If you have:

```text
LoginServlet
```

and its configuration contains:

```text
adminEmail = admin@pwioi.com
```

that parameter is associated with **LoginServlet's configuration**.

It doesn't automatically mean:

```text
PaymentServlet
```

can do:

```java
paymentServletConfig.getInitParameter("adminEmail");
```

and get LoginServlet's value.

Each servlet has its own configuration.

That's the key idea students should understand **before learning ServletContext**.

---

# 23. Final mental model

Remember this hierarchy:

```text
                    TOMCAT
                      |
                WEB APPLICATION
                      |
                ServletContext
                      |
        -----------------------------
        |             |             |
        ↓             ↓             ↓
   LoginServlet  PaymentServlet  StudentServlet
        |             |             |
        ↓             ↓             ↓
   ServletConfig ServletConfig ServletConfig
        |             |             |
        ↓             ↓             ↓
     init params   init params   init params
```

Yes. The important thing is: **`@WebInitParam` does not itself enforce “maximum 3 attempts.”** It only gives the servlet a configuration value. **Your Java code must use that value to implement the rule.**

Let's build a complete, beginner-friendly example.

## Goal

We want:

```text
/login
   ↓
User enters username + password
   ↓
Wrong password?
   ↓
Attempt counter increases
   ↓
After 3 wrong attempts
   ↓
Login blocked
```

And instead of hardcoding:

```java
int maxAttempts = 3;
```

we'll configure it using:

```java
@WebInitParam(
    name = "maxAttempts",
    value = "3"
)
```

---

# 1. Complete `LoginServlet`

```java
package com.example;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebInitParam;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

import java.io.IOException;

@WebServlet(
        value = "/login",
        initParams = {
                @WebInitParam(
                        name = "maxAttempts",
                        value = "3"
                )
        }
)
public class LoginServlet extends HttpServlet {

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        // --------------------------------
        // 1. Get username and password
        // --------------------------------

        String username =
                request.getParameter("username");

        String password =
                request.getParameter("password");


        // --------------------------------
        // 2. Get ServletConfig
        // --------------------------------

        String maxAttemptsString =
                getServletConfig()
                        .getInitParameter("maxAttempts");


        // --------------------------------
        // 3. Convert String to int
        // --------------------------------

        int maxAttempts =
                Integer.parseInt(maxAttemptsString);


        // --------------------------------
        // 4. Get session
        // --------------------------------

        HttpSession session =
                request.getSession();


        // --------------------------------
        // 5. Get current attempts
        // --------------------------------

        Integer attempts =
                (Integer) session.getAttribute("attempts");


        // --------------------------------
        // 6. If no attempts yet
        // --------------------------------

        if (attempts == null) {
            attempts = 0;
        }


        // --------------------------------
        // 7. Check login
        // --------------------------------

        if ("John".equals(username)
                && "1234".equals(password)) {

            // Correct login

            session.removeAttribute("attempts");

            response.getWriter().println(
                    "Login Successful!"
            );

        } else {

            // Wrong login

            attempts++;

            session.setAttribute(
                    "attempts",
                    attempts
            );


            // --------------------------------
            // 8. Check maximum attempts
            // --------------------------------

            if (attempts >= maxAttempts) {

                response.getWriter().println(
                        "Account blocked. Maximum login attempts reached."
                );

            } else {

                int remaining =
                        maxAttempts - attempts;

                response.getWriter().println(
                        "Wrong username or password."
                        + "<br>"
                        + "Attempts used: "
                        + attempts
                        + "<br>"
                        + "Attempts remaining: "
                        + remaining
                );
            }
        }
    }
}
```

Now let's understand **exactly what happens internally**.

---

# 2. Our configuration

At the top:

```java
@WebInitParam(
    name = "maxAttempts",
    value = "3"
)
```

This says:

```text
Parameter name
      ↓
maxAttempts

Parameter value
      ↓
3
```

But remember:

**The `3` is configuration only.**

It does NOT automatically count anything.

---

# 3. Getting `maxAttempts`

We write:

```java
String maxAttemptsString =
        getServletConfig()
                .getInitParameter("maxAttempts");
```

Tomcat has the configuration:

```text
ServletConfig

maxAttempts → "3"
```

So:

```java
getInitParameter("maxAttempts")
```

returns:

```text
"3"
```

Notice the quotes conceptually.

It's a `String`.

---

# 4. Why convert it to `int`?

We need to perform mathematics:

```text
3 - 1
3 - 2
3 - 3
```

So:

```java
int maxAttempts =
        Integer.parseInt(maxAttemptsString);
```

Now:

```text
"3"
 ↓
3
```

We can mathematically work with it.

---

# 5. But where do we store the user's attempt count?

This is where **session scope** comes in.

We create/get the user's session:

```java
HttpSession session =
        request.getSession();
```

Imagine the browser belongs to John.

Tomcat creates a session:

```text
Session
   |
   └── attempts
```

Initially:

```text
attempts = null
```

because the user hasn't attempted login yet.

---

# 6. Get previous attempt count

We do:

```java
Integer attempts =
        (Integer) session.getAttribute("attempts");
```

We're asking:

> "Does this user's session already contain an `attempts` value?"

If there isn't one:

```java
if (attempts == null) {
    attempts = 0;
}
```

So initially:

```text
attempts = 0
```

---

# 7. User enters wrong password

Suppose:

```text
Username: John
Password: abc
```

But correct password is:

```text
1234
```

Our condition:

```java
if ("John".equals(username)
        && "1234".equals(password))
```

becomes:

```text
"John" == "John"       ✅
"1234" == "abc"          ❌
```

Therefore:

```text
Login failed
```

---

# 8. Increase attempts

We execute:

```java
attempts++;
```

Before:

```text
attempts = 0
```

After:

```text
attempts = 1
```

Then we store it:

```java
session.setAttribute(
    "attempts",
    attempts
);
```

Session now contains:

```text
SESSION
   |
   └── attempts = 1
```

---

# 9. Check maximum

Our configuration says:

```text
maxAttempts = 3
```

Current attempt:

```text
attempts = 1
```

We check:

```java
if (attempts >= maxAttempts)
```

Which means:

```text
1 >= 3
```

False.

So:

```text
Attempts remaining = 3 - 1
                    = 2
```

Response:

```text
Wrong username or password.

Attempts used: 1
Attempts remaining: 2
```

---

# 10. User tries again

Second wrong attempt.

Before:

```text
Session:
attempts = 1
```

We retrieve:

```java
Integer attempts =
        (Integer) session.getAttribute("attempts");
```

Therefore:

```text
attempts = 1
```

Then:

```java
attempts++;
```

becomes:

```text
attempts = 2
```

Store:

```java
session.setAttribute("attempts", 2);
```

Now:

```text
Session:
attempts = 2
```

Check:

```text
2 >= 3
```

False.

Remaining:

```text
3 - 2 = 1
```

---

# 11. Third wrong attempt

User enters wrong password again.

Current:

```text
attempts = 2
```

Then:

```java
attempts++;
```

becomes:

```text
attempts = 3
```

Session:

```text
SESSION
   |
   └── attempts = 3
```

Then:

```java
if (attempts >= maxAttempts)
```

becomes:

```text
3 >= 3
```

True.

So:

```text
Account blocked.
Maximum login attempts reached.
```

---

# 12. The complete flow

This is what I would draw on the board for students:

```text
@WebInitParam
      |
      ↓
maxAttempts = "3"
      |
      ↓
ServletConfig
      |
      ↓
getInitParameter("maxAttempts")
      |
      ↓
Integer.parseInt()
      |
      ↓
maxAttempts = 3
```

Separately:

```text
User
 ↓
Login attempt
 ↓
HttpSession
 ↓
attempts
 ↓
0 → 1 → 2 → 3
```

Then these two meet:

```text
                  ServletConfig
                       |
                 maxAttempts = 3
                       |
                       ↓
                    COMPARE
                       ↑
                       |
                 attempts = 1/2/3
                       |
                    Session
```

That's the actual relationship.

---

# 13. Why are we using Session here?

This is an important teaching point.

`maxAttempts` and `attempts` are **two completely different things**.

### `maxAttempts`

Comes from:

```java
@WebInitParam
```

It is configuration for the servlet.

```text
maxAttempts = 3
```

### `attempts`

Comes from:

```java
HttpSession
```

It represents the current user's login-attempt state.

```text
attempts = 0
1
2
3
```

So:

```text
@WebInitParam
       ↓
"What is the allowed maximum?"

HttpSession
       ↓
"How many attempts has THIS USER made?"
```

---

# 14. Why not put `attempts` in ServletConfig?

Because ServletConfig belongs to the servlet, not an individual user.

Imagine 100 users:

```text
User A → 1 attempt
User B → 2 attempts
User C → 0 attempts
User D → 3 attempts
```

We need separate values.

If we put:

```text
attempts = 3
```

inside servlet-wide configuration, everybody would effectively share the same value.

That's not what we want.

Session gives us:

```text
User A Session → attempts = 1

User B Session → attempts = 2

User C Session → attempts = 0

User D Session → attempts = 3
```

This is why **Session Scope** is appropriate for this simple demonstration.

---

# 15. One important correction: "blocked" isn't permanent here

Our example says:

```text
Account blocked
```

But technically, this code only blocks further login attempts **according to the session's attempt count**.

If the session expires or a new session is created, the count can disappear.

For a real production login system, account lockout needs server-side persistent state, usually associated with the account, plus security controls such as rate limiting and careful handling of authentication failures.

So for teaching Servlet concepts:

```text
@WebInitParam
       +
ServletConfig
       +
HttpSession
```

is an excellent example.

But don't present it as a production-grade account-locking system.

---

# 16. Even simpler version for first-time learners

Before showing the complete code, I'd actually teach this tiny version:

```java
@WebServlet(
    value = "/login",
    initParams = {
        @WebInitParam(
            name = "maxAttempts",
            value = "3"
        )
    }
)
public class LoginServlet extends HttpServlet {

    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response) {

        String value =
                getServletConfig()
                        .getInitParameter("maxAttempts");

        int maxAttempts =
                Integer.parseInt(value);

        response.getWriter().println(
                "Maximum attempts: "
                + maxAttempts
        );
    }
}
```

Output:

```text
Maximum attempts: 3
```

# ServletContext — From Absolute Basics to Deep Understanding

Now that `ServletConfig` is clear, **ServletContext becomes much easier**.

The single most important idea is:

> **ServletConfig belongs to one servlet. ServletContext belongs to the entire web application.**

---

## 1. First understand the problem

Imagine our application has:

```text
MyWebApp
│
├── LoginServlet
├── StudentServlet
├── PaymentServlet
└── AdminServlet
```

Suppose we want some information to be available to **all these servlets**:

```text
Company Name = PW IOI
Application Name = Student Portal
Maintenance Mode = false
```

We don't want to configure this separately for every servlet.

Instead, we need something shared by the **whole application**.

That is:

```java
ServletContext
```

---

# 2. Real-life analogy

Imagine a college.

```text
                    COLLEGE
                       │
             ┌─────────┼─────────┐
             ↓         ↓         ↓
          Teacher A  Teacher B  Teacher C
```

Each teacher can have personal information:

```text
Teacher A → personal configuration
Teacher B → personal configuration
```

That's like:

```text
ServletConfig
```

But the college has information shared by everyone:

```text
College Name
College Address
College Website
Academic Year
```

That's like:

```text
ServletContext
```

So:

```text
ServletConfig
      ↓
one servlet

ServletContext
      ↓
entire web application
```

---

# 3. Who creates ServletContext?

Again, **the Servlet container** creates it.

For example:

```text
Tomcat
   ↓
starts your web application
   ↓
creates ServletContext
   ↓
creates/manages servlets
```

Conceptually:

```text
                 Tomcat
                   │
                   ↓
             Web Application
                   │
                   ↓
            ServletContext
             /     |     \
            /      |      \
           ↓       ↓       ↓
       Login    Student   Payment
       Servlet  Servlet   Servlet
```

There is generally **one ServletContext per web application within a container**.

---

# 4. How do we get ServletContext?

There are several ways.

The easiest:

```java
ServletContext context =
        getServletContext();
```

`getServletContext()` is available to the servlet through the Servlet API.

Now:

```java
context
```

represents the application's context.

---

# 5. Very simple example

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws IOException {

        ServletContext context =
                getServletContext();

        response.getWriter().println(
                "ServletContext obtained!"
        );
    }
}
```

Here:

```java
ServletContext context =
        getServletContext();
```

means:

> "Give me the ServletContext associated with my web application."

---

# 6. ServletContext can store application-wide data

This is one of its most important features.

Remember our Session example:

```java
session.setAttribute("attempts", attempts);
```

That data belonged to **one user's session**.

ServletContext also has:

```java
context.setAttribute()
```

For example:

```java
context.setAttribute(
    "collegeName",
    "PW IOI"
);
```

Now the application context contains:

```text
ServletContext
┌─────────────────────────┐
│ collegeName → PW IOI    │
└─────────────────────────┘
```

Any servlet in the same application can access that attribute.

---

# 7. Another servlet can retrieve it

Suppose `LoginServlet` does:

```java
ServletContext context =
        getServletContext();

context.setAttribute(
        "collegeName",
        "PW IOI"
);
```

Now `StudentServlet` can do:

```java
ServletContext context =
        getServletContext();

String collegeName =
        (String) context.getAttribute("collegeName");
```

And:

```text
collegeName
     ↓
"PW IOI"
```

This is the key concept:

> **ServletContext allows servlets within the same web application to share application-level data.**

---

# 8. Session vs ServletContext

This is extremely important.

Suppose there are two users:

```text
User A
User B
```

If we use:

```java
session.setAttribute("username", "John");
```

that username belongs to **User A's session**.

User B has a different session.

```text
User A Session
└── username = John

User B Session
└── username = Rahul
```

But if we do:

```java
context.setAttribute(
    "collegeName",
    "PW IOI"
);
```

then:

```text
             ServletContext
                   │
          collegeName = PW IOI
             /     |     \
            ↓      ↓      ↓
         User A  User B  User C
```

Everyone can access it.

---

# 9. Three scopes together

Now you can see the hierarchy:

```text
REQUEST
   ↓
One HTTP request

SESSION
   ↓
One user's session

APPLICATION
   ↓
Entire web application
```

Servlet API gives us corresponding mechanisms:

```text
Request scope
     ↓
request.setAttribute()

Session scope
     ↓
session.setAttribute()

Application scope
     ↓
context.setAttribute()
```

This is a VERY important table for students:

| Scope | Object | Data belongs to |
|---|---|---|
| Request | `HttpServletRequest` | One request |
| Session | `HttpSession` | One user/session |
| Application | `ServletContext` | Entire application |

---

# 10. ServletContext initialization parameters

Just like `ServletConfig` can have initialization parameters, **ServletContext can have application-wide initialization parameters**.

For example, in `web.xml`:

```xml
<context-param>
    <param-name>companyName</param-name>
    <param-value>PW IOI</param-value>
</context-param>
```

This is different from:

```xml
<init-param>
```

Remember:

### ServletConfig

```xml
<init-param>
```

belongs to one servlet.

### ServletContext

```xml
<context-param>
```

belongs to the entire application.

---

# 11. Reading a context parameter

Java:

```java
ServletContext context =
        getServletContext();

String companyName =
        context.getInitParameter("companyName");
```

If `web.xml` contains:

```xml
<context-param>
    <param-name>companyName</param-name>
    <param-value>PW IOI</param-value>
</context-param>
```

then:

```java
context.getInitParameter("companyName")
```

returns:

```text
PW IOI
```

---

# 12. Compare this with `@WebInitParam`

This is where your previous topic connects directly.

### Servlet-specific

```java
@WebServlet(
    value = "/login",
    initParams = {
        @WebInitParam(
            name = "maxAttempts",
            value = "3"
        )
    }
)
```

Then:

```java
getServletConfig()
    .getInitParameter("maxAttempts");
```

---

### Application-wide

You can define:

```xml
<context-param>
    <param-name>companyName</param-name>
    <param-value>PW IOI</param-value>
</context-param>
```

Then:

```java
getServletContext()
    .getInitParameter("companyName");
```

Notice the difference:

```text
@WebInitParam
     ↓
ServletConfig
     ↓
one servlet


<context-param>
     ↓
ServletContext
     ↓
entire application
```

---

# 13. Why can't we use `@WebInitParam` for application-wide configuration?

Because:

```java
@WebInitParam
```

is designed for **servlet initialization parameters**.

For application-wide initialization parameters, traditionally you define:

```xml
<context-param>
```

in `web.xml`.

The distinction is:

```text
@WebInitParam
       ↓
Servlet initialization parameter


<context-param>
       ↓
Application context initialization parameter
```

---

# 14. ServletContext attributes

Don't confuse:

```java
context.setAttribute()
```

with:

```java
context.getInitParameter()
```

They serve different purposes.

### Initialization parameter

```java
context.getInitParameter("companyName");
```

Used for configuration.

### Attribute

```java
context.getAttribute("companyName");
```

Used for data stored in the application context.

For example:

```java
context.setAttribute(
    "totalStudents",
    500
);
```

Then:

```java
Integer total =
        (Integer) context.getAttribute(
                "totalStudents"
        );
```

---

# 15. Parameter vs Attribute

This distinction is worth teaching carefully.

### Parameter

Generally configuration:

```text
companyName → PW IOI
maxFileSize → 10MB
environment → production
```

Defined/configured by the application.

Access:

```java
getInitParameter()
```

---

### Attribute

Runtime data:

```text
onlineStudents → 120
totalVisitors → 5000
currentAnnouncement → "Exam tomorrow"
```

Stored during application execution.

Access:

```java
getAttribute()
```

So:

```text
Parameter
    ↓
Configuration

Attribute
    ↓
Runtime/shared data
```

---

# 16. Complete example

Let's make two servlets.

### `LoginServlet`

```java
package com.example;

import jakarta.servlet.ServletContext;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

import java.io.IOException;

@WebServlet("/login")
public class LoginServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        ServletContext context =
                getServletContext();

        context.setAttribute(
                "collegeName",
                "PW IOI"
        );

        response.getWriter().println(
                "Application data stored."
        );
    }
}
```

Now the application context has:

```text
ServletContext
     |
     └── collegeName → PW IOI
```

---

# 17. `StudentServlet`

```java
package com.example;

import jakarta.servlet.ServletContext;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

import java.io.IOException;

@WebServlet("/student")
public class StudentServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        ServletContext context =
                getServletContext();

        String collegeName =
                (String) context.getAttribute(
                        "collegeName"
                );

        response.getWriter().println(
                "College: " + collegeName
        );
    }
}
```

If `/login` has already been visited, then `/student` can access:

```text
College: PW IOI
```

because both servlets share the same application context.

---

# 18. But there is a problem with that example

What happens if the user directly visits:

```text
/student
```

before:

```text
/login
```

Then:

```java
context.getAttribute("collegeName")
```

may return:

```text
null
```

because nobody has stored the attribute yet.

That's why application-wide initialization data is often configured at application startup rather than depending on one user visiting a servlet first.

---

# 19. ServletContext is not "user-specific"

This is perhaps the biggest thing to remember.

Don't do this:

```java
context.setAttribute(
    "username",
    "John"
);
```

for logged-in user information.

Why?

Because the context is shared.

Imagine:

```text
User A logs in
username = John
```

Then:

```java
context.setAttribute(
    "username",
    "John"
);
```

Then User B logs in:

```text
username = Rahul
```

and:

```java
context.setAttribute(
    "username",
    "Rahul"
);
```

Now the shared application value has changed.

You don't want to use application scope for individual users.

For that:

```text
User-specific
     ↓
HttpSession
```

---

# 20. ServletContext lifecycle

Think of the application lifecycle:

```text
Tomcat starts
      ↓
Web application starts
      ↓
ServletContext created
      ↓
Application runs
      ↓
Servlets use ServletContext
      ↓
Application stops
      ↓
ServletContext destroyed
```

So its lifetime is tied to the **web application**.

This is different from request scope.

A request:

```text
Browser → Server
```

might last milliseconds.

A session might last minutes/hours.

ServletContext lives for the lifetime of the web application.

---

# 21. Important methods of ServletContext

Some important methods:

```java
getInitParameter()
```

Get application initialization parameter.

```java
getInitParameterNames()
```

Get all application initialization parameter names.

```java
getAttribute()
```

Get an application attribute.

```java
setAttribute()
```

Store an application attribute.

```java
removeAttribute()
```

Remove an application attribute.

```java
getServletContextName()
```

Get the application's configured display name.

There are also methods for things such as application resources, logging, servlet registration, and dispatching requests.

---

# 22. The most important comparison

Put this on your classroom board:

```text
                CONFIGURATION

       ServletConfig        ServletContext
             │                    │
             ↓                    ↓
       One Servlet          Whole Application
             │                    │
             ↓                    ↓
       @WebInitParam        <context-param>
             │                    │
             ↓                    ↓
      getInitParameter()   getInitParameter()
```

And for runtime data:

```text
                ATTRIBUTES

Request                 Session              Application
   │                       │                     │
   ↓                       ↓                     ↓
request                session                context
   │                       │                     │
   ↓                       ↓                     ↓
setAttribute()         setAttribute()        setAttribute()
```

---

# 23. One-line memory trick

Tell students:

> **Config = Configuration of one servlet. Context = Common environment of the whole application.**

Or even simpler:

```text
ServletConfig
     ↓
"My configuration"

ServletContext
     ↓
"Our application's information"
```

And the complete picture becomes:

```text
                    WEB APPLICATION
                           │
                    ServletContext
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
     LoginServlet     StudentServlet   PaymentServlet
          │                │                │
          ↓                ↓                ↓
   ServletConfig     ServletConfig     ServletConfig
```

**One application → one shared context → many servlets → each servlet has its own config.**
### In one sentence:
