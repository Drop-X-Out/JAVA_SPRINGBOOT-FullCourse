# Redirecting HTTP Requests Using `sendRedirect()` and Solving Double Posting with PRG

## 1. Learning Objectives

After completing this lesson, students will understand:

1. What redirection means in a web application.
2. How `HttpServletResponse.sendRedirect()` works.
3. The difference between `forward()` and `sendRedirect()`.
4. Why refreshing a form submission can cause duplicate submissions.
5. What the **Post/Redirect/Get (PRG)** pattern is.
6. How to implement PRG using Java Servlets.
7. How to test redirection and prevent accidental duplicate form submissions.

---

# Part 1: Understanding Redirection

## 2. What Is Redirection?

Suppose a student opens this URL:

```text
http://localhost:8080/servlet-demo/login
```

After successfully logging in, we want the browser to open:

```text
http://localhost:8080/servlet-demo/dashboard
```

The server can ask the browser to navigate to another URL. This process is called **redirection**.

In Java Servlets, we can redirect the browser using:

```java
response.sendRedirect("dashboard");
```

Or, more explicitly:

```java
response.sendRedirect(request.getContextPath() + "/dashboard");
```

Here:

- `response` represents the HTTP response sent to the browser.
- `sendRedirect()` tells the browser to make a new request to another URL.
- `request.getContextPath()` returns the application's context path, such as `/servlet-demo`.
- `"/dashboard"` is the destination path within the application.

> **Important:** `sendRedirect()` is a method of `HttpServletResponse`, not `HttpServletRequest`.

---

## 3. How Does `sendRedirect()` Work?

Consider this code:

```java
response.sendRedirect(
    request.getContextPath() + "/dashboard"
);
```

Assume the application context path is `/servlet-demo`.

The destination becomes:

```text
/servlet-demo/dashboard
```

The flow is:

```text
Browser
   |
   | 1. GET /servlet-demo/login
   v
LoginServlet
   |
   | 2. Sends redirect response
   |    HTTP status: 302
   |    Location: /servlet-demo/dashboard
   v
Browser
   |
   | 3. Makes a NEW GET request
   |    GET /servlet-demo/dashboard
   v
DashboardServlet
   |
   | 4. Sends dashboard response
   v
Browser displays dashboard
```

The browser makes a new request because the server responds with a redirect status and a `Location` header.

A typical redirect response looks like this:

```http
HTTP/1.1 302 Found
Location: /servlet-demo/dashboard
```

The browser reads the `Location` header and navigates to the specified URL.

### Important points

- A redirect involves a response from the server followed by a new request from the browser.
- The browser's address bar normally changes to the destination URL.
- The new request is separate from the original request.
- Request attributes from the original request are not automatically carried over.
- For a normal `302` redirect, browsers typically follow the redirect using a `GET` request, even when the original request used `POST`.

---

# Part 2: Understanding the Difference Between `forward()` and `sendRedirect()`

## 4. `forward()` vs `sendRedirect()`

Both can lead to another resource, but they work differently.

| Feature | `forward()` | `sendRedirect()` |
|---|---|---|
| Performed by | Servlet container | Browser follows the redirect |
| Number of browser requests | One | Normally two requests |
| New browser request | No | Yes |
| Address bar changes | No | Yes |
| Request attributes | Remain available in the forwarded request | Not automatically carried over |
| Can redirect to an external website? | No, only resources handled by the same web application/container | Yes |
| Typical use | Forwarding to a JSP or server-side resource | Navigating to another URL |

### Example of `forward()`

```java
request.getRequestDispatcher("/dashboard")
       .forward(request, response);
```

The container forwards the existing request to the resource mapped to `/dashboard`. The browser does not make a second request.

### Example of `sendRedirect()`

```java
response.sendRedirect(
    request.getContextPath() + "/dashboard"
);
```

The browser receives the redirect response and requests `/dashboard` separately.

### When should we use each?

Use `forward()` when you want the server to process or render another resource as part of the same request.

Use `sendRedirect()` when you want the browser to navigate to another URL, especially after successfully processing a form submission.

---

# Part 3: Understanding the Double Posting Problem

## 5. What Is Double Posting?

Imagine we build a student registration form.

A student enters:

```text
Name: Rahul
Course: Java
```

The student clicks **Register**.

The browser sends:

```http
POST /servlet-demo/register
```

The server processes the registration and displays:

```text
Registration successful!
```

Now suppose the student refreshes the page.

The browser may display a confirmation asking whether to resubmit the form. If the student confirms, the browser sends the same POST request again.

The registration code executes again.

This can create:

- Duplicate student registrations.
- Duplicate orders.
- Duplicate payment attempts.
- Duplicate comments.
- Duplicate records in a database.

This problem is commonly called **double posting** or duplicate form submission.

### Important clarification

Refreshing a page does not always duplicate a POST automatically. Browsers often ask the user to confirm resubmission. The problem occurs when the POST request is repeated, whether through refresh, repeated clicks, a retry, or another client action.

---

## 6. Understanding the Problem Without PRG

Consider this simplified servlet:

```java
package servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/register")
public class RegisterServlet extends HttpServlet {

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        String name = request.getParameter("name");
        String course = request.getParameter("course");

        // Imagine saving the student into a database here.
        System.out.println("Saving student: " + name);

        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();

        out.println("<h1>Registration Successful!</h1>");
        out.println("<p>Name: " + name + "</p>");
        out.println("<p>Course: " + course + "</p>");
    }
}
```

The servlet processes the POST request and directly returns the result.

The flow is:

```text
Browser submits the form
        |
        v
POST /register
        |
        v
RegisterServlet.doPost()
        |
        v
Save student
        |
        v
Display success page
        |
        v
User refreshes the page
        |
        v
Browser may resubmit POST
        |
        v
Student may be saved again
```

The problem is that the page displayed in the browser is the response to the original POST request.

**We need a way to show the result without leaving the browser on the POST response.**

That is where PRG comes in.

---

# Part 4: Understanding the PRG Pattern

## 7. What Is PRG?

**PRG stands for Post/Redirect/Get.**

It is a web development pattern used after successfully processing a form submission.

It has three stages:

1. **POST:** The browser submits the form.
2. **REDIRECT:** The server processes the form and redirects the browser.
3. **GET:** The browser makes a new GET request to retrieve the result page.

The flow becomes:

```text
             Browser
                |
                | 1. POST /register
                v
          RegisterServlet
                |
                | 2. Process the form
                |    Save student
                |
                | 3. Send redirect
                |    HTTP 303 See Other
                |    Location: /result
                v
             Browser
                |
                | 4. GET /result
                v
          ResultServlet
                |
                | 5. Return success page
                v
             Browser
```

After the redirect, the browser's current page is the result of a **GET** request.

When the user refreshes that page, the browser repeats the GET request instead of resubmitting the original POST.

### The three steps in simple language

| Stage | What happens? | Example |
|---|---|---|
| POST | Submit the form | `/register` |
| Redirect | Server tells the browser to navigate elsewhere | `/result` |
| GET | Browser fetches the result page | `/result` |

### Which HTTP status should we use?

For a POST followed by a GET, HTTP `303 See Other` clearly instructs the client to retrieve the destination using GET.

In Servlet code:

```java
response.setStatus(HttpServletResponse.SC_SEE_OTHER);

response.setHeader(
    "Location",
    request.getContextPath() + "/result"
);
```

Alternatively, the common shortcut is:

```java
response.sendRedirect(
    request.getContextPath() + "/result"
);
```

`sendRedirect()` normally sends a `302 Found` response. Browsers typically change the follow-up request to GET, making it suitable for the common PRG flow.

For an explicit PRG implementation, we will use **303 See Other**.

---

# Part 5: Complete Maven Project Using PRG

## 8. Project Requirements

We will create a simple student registration application.

When the user submits the form:

1. The browser sends a POST request.
2. The servlet validates the input.
3. The servlet simulates saving the student.
4. The servlet stores a temporary success message in the HTTP session.
5. The servlet redirects to a result URL using HTTP 303.
6. The browser sends a GET request to the result servlet.
7. The result servlet displays the message.
8. Refreshing the result page does not repeat the original POST.

For now, we will simulate saving the student using an in-memory list. Database integration can be added later.

## 9. Complete File Structure

Create a Maven Web Application in IntelliJ IDEA.

```text
servlet-demo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │   └── servlet/
        │       ├── RegisterServlet.java
        │       └── ResultServlet.java
        │
        └── webapp/
            ├── index.html
            │
            └── WEB-INF/
                └── web.xml
```

We will keep both Java classes in the same package:

```java
package servlet;
```

---

## 10. Maven Configuration — `pom.xml`

File location:

```text
servlet-demo/pom.xml
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

### Explanation

**`<packaging>war</packaging>`**

WAR stands for Web Application Archive. It packages Java web application files for deployment to a servlet container.

**`jakarta.servlet-api`**

This dependency provides the servlet interfaces and classes required for compilation.

**`<scope>provided</scope>`**

Tomcat provides the Servlet API at runtime, so Maven should not package another copy of that API inside the application's WAR.

**Java 17**

We use Java 17 for compilation. Tomcat 10.1 supports Jakarta Servlet 6.0 and requires Java 11 or later.

---

## 11. Create the Registration Form — `index.html`

File location:

```text
src/main/webapp/index.html
```

Complete code:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Student Registration</title>
</head>

<body>

    <h1>Student Registration Form</h1>

    <form action="register" method="post">

        <div>
            <label for="name">Student Name:</label>

            <input
                type="text"
                id="name"
                name="name"
                required
            >
        </div>

        <br>

        <div>
            <label for="course">Course:</label>

            <select id="course" name="course" required>
                <option value="">Select a course</option>
                <option value="Java">Java</option>
                <option value="Python">Python</option>
                <option value="Web Development">Web Development</option>
            </select>
        </div>

        <br>

        <button type="submit">Register Student</button>

    </form>

</body>

</html>
```

### Explanation

**`action="register"`**

The browser submits the form to the application's `/register` endpoint, relative to the current page URL.

**`method="post"`**

The browser sends the submitted form fields in the request body using HTTP POST.

**`name="name"` and `name="course"`**

The `name` attributes identify the parameters that the servlet reads.

For example:

```java
String name = request.getParameter("name");
String course = request.getParameter("course");
```

The HTML `required` attribute performs basic browser-side validation. The servlet must still validate the input because clients can bypass browser validation.

---

## 12. Registration Servlet — `RegisterServlet.java`

File location:

```text
src/main/java/servlet/RegisterServlet.java
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
import java.util.ArrayList;
import java.util.List;

@WebServlet("/register")
public class RegisterServlet extends HttpServlet {

    /*
     * Temporary in-memory storage for demonstration.
     *
     * This is shared by requests handled by this servlet instance.
     * It is NOT persistent database storage.
     */
    private final List<String> registeredStudents =
            new ArrayList<>();

    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        // Read the submitted form fields.
        String name = request.getParameter("name");
        String course = request.getParameter("course");

        // Validate the input on the server.
        if (name == null || name.isBlank()
                || course == null || course.isBlank()) {

            response.sendError(
                    HttpServletResponse.SC_BAD_REQUEST,
                    "Student name and course are required."
            );

            return;
        }

        // Simulate saving the student.
        String student = name.trim() + " - " + course;

        registeredStudents.add(student);

        System.out.println("Student registered: " + student);

        // Obtain or create the user's HTTP session.
        HttpSession session = request.getSession();

        // Store a one-time message for the result page.
        session.setAttribute(
                "successMessage",
                "Student registered successfully: " + student
        );

        /*
         * PRG STEP 2:
         * Redirect the browser to the result URL.
         *
         * HTTP 303 tells the client to retrieve the result
         * using GET, rather than repeating the POST.
         */
        response.setStatus(HttpServletResponse.SC_SEE_OTHER);

        response.setHeader(
                "Location",
                request.getContextPath() + "/result"
        );
    }
}
```

### Understand the code step by step

#### Step 1: Map the servlet

```java
@WebServlet("/register")
```

This annotation maps the URL `/register` to `RegisterServlet`.

For example, if the context path is `/servlet-demo`, the full endpoint is:

```text
http://localhost:8080/servlet-demo/register
```

#### Step 2: Handle POST requests

```java
@Override
protected void doPost(
        HttpServletRequest request,
        HttpServletResponse response
)
```

The browser submits the form using POST, so the servlet container invokes `doPost()`.

#### Step 3: Read the form values

```java
String name = request.getParameter("name");
String course = request.getParameter("course");
```

If the student enters Rahul and Java, the variables contain:

```text
name = "Rahul"
course = "Java"
```

#### Step 4: Validate on the server

```java
if (name == null || name.isBlank()
        || course == null || course.isBlank()) {
```

This checks for missing or blank values.

`return;` stops execution when the request is invalid. The `sendError()` call sends a 400 Bad Request response.

#### Step 5: Simulate saving the student

```java
registeredStudents.add(student);
```

This is only an in-memory demonstration. In a real application, the data would normally be saved to a database.

**Important:** This list is not durable storage, and this example does not make registration idempotent. If the same POST is deliberately repeated, it can still add the student again. PRG prevents the normal refresh-after-success problem, but it is not a complete duplicate-submission protection mechanism.

#### Step 6: Store a message in the session

```java
HttpSession session = request.getSession();

session.setAttribute(
        "successMessage",
        "Student registered successfully: " + student
);
```

We need the message to survive the redirect.

The original POST request and the result GET request are separate requests. Therefore, an ordinary request attribute would not automatically be available in the result servlet.

An HTTP session lets us temporarily store the message across those requests.

#### Step 7: Send a 303 redirect

```java
response.setStatus(HttpServletResponse.SC_SEE_OTHER);
```

This sets the HTTP response status to `303 See Other`.

Then:

```java
response.setHeader(
        "Location",
        request.getContextPath() + "/result"
);
```

This tells the browser where to go next.

If the context path is `/servlet-demo`, the response contains:

```http
HTTP/1.1 303 See Other
Location: /servlet-demo/result
```

The browser then sends:

```http
GET /servlet-demo/result
```

Notice that the browser does not need to repeat the original registration POST to display the success page.

---

## 13. Result Servlet — `ResultServlet.java`

File location:

```text
src/main/java/servlet/ResultServlet.java
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
import java.io.PrintWriter;

@WebServlet("/result")
public class ResultServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        response.setContentType("text/html;charset=UTF-8");

        HttpSession session = request.getSession(false);

        String successMessage = null;

        if (session != null) {
            Object message = session.getAttribute("successMessage");

            if (message instanceof String) {
                successMessage = (String) message;
            }

            // Remove the message after reading it.
            session.removeAttribute("successMessage");
        }

        PrintWriter out = response.getWriter();

        out.println("<!DOCTYPE html>");
        out.println("<html lang='en'>");

        out.println("<head>");
        out.println("<meta charset='UTF-8'>");
        out.println("<title>Registration Result</title>");
        out.println("</head>");

        out.println("<body>");

        out.println("<h1>Registration Result</h1>");

        if (successMessage != null) {

            out.println("<p>" + escapeHtml(successMessage) + "</p>");

        } else {

            out.println("<p>No new registration message is available.</p>");

        }

        out.println("<a href='" + request.getContextPath()
                + "/'>Register another student</a>");

        out.println("</body>");
        out.println("</html>");
    }

    /*
     * Escape special HTML characters before placing user-influenced
     * text inside the HTML response.
     */
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

### Explanation

#### Why do we use `doGet()`?

The registration servlet responds to the form's POST request.

After redirection, the browser makes a GET request to `/result`.

Therefore, the result servlet handles the request in:

```java
protected void doGet(...)
```

#### Why use `request.getSession(false)`?

```java
HttpSession session = request.getSession(false);
```

This returns the existing session if one exists. It does not create a new session when none exists.

The result page can still display a fallback message if the user visits `/result` directly.

#### Why remove the message?

```java
session.removeAttribute("successMessage");
```

This implements a simple **flash message**: a message stored temporarily, read on the next page request, and then removed.

Without removing it, refreshing the result page could show the old message again. Removing the message makes the example easier to understand, although session handling across multiple tabs or concurrent requests requires additional care in production applications.

#### Why escape the output?

```java
escapeHtml(successMessage)
```

The student name comes from user input. Escaping HTML special characters helps prevent user input from being interpreted as HTML markup or executable script.

---

## 14. Optional `web.xml`

File location:

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

    <display-name>Servlet Demo</display-name>

</web-app>
```

We use `@WebServlet` annotations to map both servlets, so we do not need servlet mappings in this XML file.

---

# Part 6: Run the Application in IntelliJ IDEA

## 15. Configure Smart Tomcat

1. Open the project in IntelliJ IDEA.
2. Ensure Maven dependencies have loaded successfully.
3. Install or enable the **Smart Tomcat** plugin if it is not already available.
4. Install Tomcat 10.1 locally.
5. Create a Smart Tomcat run configuration.
6. Select your Tomcat installation directory.
7. Select the web application or module directory according to the plugin configuration.
8. Set the application context path to `/servlet-demo`, if the configuration exposes that option.
9. Select the project module and run configuration.
10. Start Tomcat.

The exact field names can differ between Smart Tomcat plugin versions. Check the run configuration to ensure the application is deployed with the intended context path.

### Build using Maven

From the project root:

```bash
mvn clean package
```

This builds the WAR file:

```text
target/servlet-demo.war
```

If you need an exploded WAR for deployment or inspection:

```bash
mvn clean package war:exploded
```

The exploded web application is typically generated under:

```text
target/servlet-demo/
```

Smart Tomcat can use its configured deployment directory or module settings to run the application. You do not need to manually copy the WAR into Tomcat's `webapps` directory when using the configured IDE workflow.

---

# Part 7: Test the Application

## 16. Open the Registration Form

Assuming Tomcat runs on port `8080` and the context path is `/servlet-demo`, open:

```text
http://localhost:8080/servlet-demo/
```

Enter:

```text
Student Name: Rahul
Course: Java
```

Click **Register Student**.

### Expected sequence

The browser submits:

```http
POST /servlet-demo/register
```

The servlet processes the form and responds with:

```http
HTTP/1.1 303 See Other
Location: /servlet-demo/result
```

The browser follows the redirect:

```http
GET /servlet-demo/result
```

The result page displays:

```text
Registration Result

Student registered successfully: Rahul - Java
```

The address bar should now contain:

```text
http://localhost:8080/servlet-demo/result
```

## 17. Test Refreshing the Result Page

After registration:

1. Confirm that the browser is on `/result`.
2. Refresh the page.
3. Observe that the browser does not ask to resubmit the registration POST.
4. The one-time message may be gone because the result servlet removed it from the session.

Check the IntelliJ console. You should see one registration message for the single submission:

```text
Student registered: Rahul - Java
```

Refreshing `/result` makes another GET request; it does not run `RegisterServlet.doPost()` again.

**This is the key benefit of PRG.**

---

# Part 8: What PRG Does and Does Not Guarantee

## 18. Does PRG Completely Prevent Duplicate Registrations?

No. PRG prevents the common problem in which a user refreshes the successful POST response and accidentally resubmits the form.

It does not guarantee that a registration can happen only once.

For example, the user could:

- Double-click the submit button.
- Submit the form twice before the first response arrives.
- Retry a failed request.
- Open the form in two tabs and submit both.
- Send the same POST request using a script or API client.

Each request could reach the registration servlet.

### Production-level protections

For important operations, combine PRG with additional safeguards:

1. **Server-side validation:** Reject invalid input even if browser validation is bypassed.
2. **Database constraints:** For example, a unique constraint on a student registration number or email when appropriate.
3. **Idempotency keys:** Associate retries with the same unique operation key, particularly for payments and APIs.
4. **Transaction management:** Ensure related database operations succeed or fail consistently.
5. **Duplicate-click prevention:** Disable the submit button after the first click as a user-experience improvement, not as the only protection.

For a real registration application, a database unique constraint is generally more reliable than checking a Java list before inserting a record.

---

# Part 9: Common Mistakes

## 19. Mistake 1 — Writing the Success Page Directly in `doPost()`

Problem:

```java
protected void doPost(
        HttpServletRequest request,
        HttpServletResponse response
) throws IOException {

    // Save the student...

    response.getWriter().println("Registration successful!");
}
```

This leaves the browser displaying the POST response. Refreshing may cause the browser to ask to resubmit the form.

**Solution:** After successful processing, redirect to a GET endpoint.

## 20. Mistake 2 — Using `forward()` Instead of Redirecting

```java
request.getRequestDispatcher("/result")
       .forward(request, response);
```

This forwards the original POST request internally. It does not create a new browser GET request.

Consequently, the browser is still displaying the response to the POST operation.

**Solution:** Use `sendRedirect()` or an explicit 303 response for PRG.

## 21. Mistake 3 — Forgetting the Context Path

Less robust:

```java
response.sendRedirect("/result");
```

A leading slash in this URL points to the server root, not necessarily to the current application. If the application context path is `/servlet-demo`, the redirect may go to the wrong place.

Safer:

```java
response.sendRedirect(
        request.getContextPath() + "/result"
);
```

For an explicit 303:

```java
response.setStatus(HttpServletResponse.SC_SEE_OTHER);

response.setHeader(
        "Location",
        request.getContextPath() + "/result"
);
```

## 22. Mistake 4 — Trying to Read a Request Attribute After Redirect

For example:

```java
request.setAttribute("message", "Success");

response.sendRedirect(
        request.getContextPath() + "/result"
);
```

The new GET request is a different request. The request attribute is not automatically transferred.

**Solution:** Use an HTTP session for a simple flash message, or use another appropriate mechanism such as a database-backed result identifier.

## 23. Mistake 5 — Expecting PRG to Prevent All Duplicate Requests

PRG does not make the POST operation itself idempotent. Two POST requests can still execute the registration logic twice.

**Solution:** Use PRG for browser refresh behavior and add server-side duplicate protection for important operations.

---

# Part 10: Quick Revision

## 24. Important Methods and Constants

| Method or constant | Purpose |
|---|---|
| `response.sendRedirect(url)` | Sends a redirect response, normally using status 302 |
| `response.setStatus(HttpServletResponse.SC_SEE_OTHER)` | Sets HTTP status 303 |
| `response.setHeader("Location", url)` | Sets the destination URL |
| `request.getContextPath()` | Returns the application's context path |
| `request.getParameter(name)` | Reads a submitted form parameter |
| `request.getSession()` | Returns the current session or creates one |
| `request.getSession(false)` | Returns the existing session or `null` |
| `session.setAttribute(name, value)` | Stores data in the session |
| `session.getAttribute(name)` | Reads data from the session |
| `session.removeAttribute(name)` | Removes data from the session |

## 25. Interview Questions

**Q1. What is `sendRedirect()`?**

It is a method of `HttpServletResponse` that instructs the browser to navigate to another URL by sending a redirect response.

**Q2. What is the difference between `forward()` and `sendRedirect()`?**

`forward()` internally forwards the same request to another resource. `sendRedirect()` tells the browser to make a new request.

**Q3. What does PRG stand for?**

Post/Redirect/Get.

**Q4. Why do we use PRG?**

To avoid the common browser refresh behavior that can resubmit a form's POST request after the response has been displayed.

**Q5. Which HTTP status code is useful for explicitly implementing PRG?**

`303 See Other`, because it directs the client to retrieve the result using GET.

**Q6. Does `sendRedirect()` normally return status 303?**

No. The standard Servlet API's `sendRedirect()` normally uses status `302 Found`. For an explicit 303, set the status to `HttpServletResponse.SC_SEE_OTHER` and set the `Location` header.

**Q7. Can a redirect preserve request attributes?**

No. The browser makes a new request, so request attributes from the original request are not automatically preserved.

**Q8. Why did we use `HttpSession` in this example?**

To carry a temporary success message from the POST request to the GET request after redirection.

**Q9. Does PRG guarantee that a form will only be submitted once?**

No. It helps prevent accidental resubmission caused by refreshing the result page. Server-side idempotency and database constraints may still be necessary.

**Q10. What happens when the user refreshes the result page?**

The browser repeats the GET request. It does not automatically repeat the original registration POST.

---

# Final Summary

The PRG pattern changes the flow of a form submission from:

```text
POST → Display Result
```

to:

```text
POST → Redirect → GET → Display Result
```

Use `sendRedirect()` when you want the browser to navigate to another URL. For an explicit PRG implementation, return **303 See Other** after successfully processing the POST request.

