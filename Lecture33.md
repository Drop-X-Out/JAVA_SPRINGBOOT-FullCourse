- `@{}`
- `th:href`
- `th:action`
- `th:object`
- `th:field`
- `th:if`
- `th:unless`
- `th:switch`
- `th:each`
- `th:value`
- hidden fields
- `${}` and `*{}`

We'll make a **Student Registration Form** and keep it beginner-friendly.

## 1. Folder structure

```text
Servlet4/
│
├── pom.xml
│
└── src/
    └── main/
        │
        ├── java/
        │   └── com/
        │       └── example/
        │           ├── Student.java
        │           └── StudentServlet.java
        │
        └── webapp/
            └── WEB-INF/
                └── templates/
                    ├── student-form.html
                    └── students.html
```

---

# 2. `pom.xml`

Remove `thymeleaf-spring6` because we are using **Servlet, not Spring**.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>Servlet4</artifactId>

    <packaging>war</packaging>

    <version>1.0-SNAPSHOT</version>

    <name>Servlet4 Maven Webapp</name>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>

        <!-- Servlet API -->
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>6.1.0</version>
            <scope>provided</scope>
        </dependency>

        <!-- Thymeleaf -->
        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf</artifactId>
            <version>3.1.3.RELEASE</version>
        </dependency>

    </dependencies>

    <build>

        <finalName>Servlet4</finalName>

    </build>

</project>
```

---

# 3. Create `Student.java`

Location:

```text
src/main/java/com/example/Student.java
```

```java
package com.example;

public class Student {

    private int id;

    private String name;

    private String email;

    private int age;

    private String course;

    private String gender;

    private String city;

    private boolean active;

    public Student() {
    }

    public Student(
            int id,
            String name,
            String email,
            int age,
            String course,
            String gender,
            String city,
            boolean active
    ) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.age = age;
        this.course = course;
        this.gender = gender;
        this.city = city;
        this.active = active;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getCourse() {
        return course;
    }

    public void setCourse(String course) {
        this.course = course;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public boolean isActive() {
        return active;
    }

    public void setActive(boolean active) {
        this.active = active;
    }
}
```

---

# 4. Create `StudentServlet.java`

Location:

```text
src/main/java/com/example/StudentServlet.java
```

Complete code:

```java
package com.example;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.WebContext;
import org.thymeleaf.templateresolver.WebApplicationTemplateResolver;
import org.thymeleaf.web.IWebExchange;
import org.thymeleaf.web.servlet.JakartaServletWebApplication;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

@WebServlet("/student")
public class StudentServlet extends HttpServlet {

    private TemplateEngine templateEngine;

    private List<Student> students = new ArrayList<>();

    @Override
    public void init() throws ServletException {

        /*
         * Create Thymeleaf application
         */
        JakartaServletWebApplication application =
                JakartaServletWebApplication.buildApplication(
                        getServletContext()
                );

        /*
         * Create Template Resolver
         */
        WebApplicationTemplateResolver resolver =
                new WebApplicationTemplateResolver(
                        application
                );

        /*
         * Template location
         */
        resolver.setPrefix(
                "/WEB-INF/templates/"
        );

        resolver.setSuffix(
                ".html"
        );

        resolver.setTemplateMode(
                "HTML"
        );

        resolver.setCacheable(false);

        /*
         * Create Template Engine
         */
        templateEngine = new TemplateEngine();

        /*
         * Connect resolver
         */
        templateEngine.setTemplateResolver(
                resolver
        );


        /*
         * Sample students
         */
        students.add(
                new Student(
                        1,
                        "Rahul",
                        "rahul@gmail.com",
                        21,
                        "Java",
                        "Male",
                        "Lucknow",
                        true
                )
        );

        students.add(
                new Student(
                        2,
                        "Priya",
                        "priya@gmail.com",
                        22,
                        "Python",
                        "Female",
                        "Delhi",
                        true
                )
        );
    }


    /*
     * GET /student
     *
     * Show form
     */
    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        /*
         * Create empty student
         */
        Student student = new Student();

        /*
         * Put student into context
         */
        WebContext context = createContext(
                request,
                response
        );

        context.setVariable(
                "student",
                student
        );

        context.setVariable(
                "students",
                students
        );

        context.setVariable(
                "message",
                null
        );

        /*
         * Process HTML
         */
        templateEngine.process(
                "student-form",
                context,
                response.getWriter()
        );
    }


    /*
     * POST /student
     *
     * Receive form data
     */
    @Override
    protected void doPost(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        /*
         * Get form values
         */
        int id = Integer.parseInt(
                request.getParameter("id")
        );

        String name =
                request.getParameter("name");

        String email =
                request.getParameter("email");

        int age = Integer.parseInt(
                request.getParameter("age")
        );

        String course =
                request.getParameter("course");

        String gender =
                request.getParameter("gender");

        String city =
                request.getParameter("city");

        boolean active =
                request.getParameter("active") != null;


        /*
         * Create Student object
         */
        Student student = new Student(
                id,
                name,
                email,
                age,
                course,
                gender,
                city,
                active
        );


        /*
         * Store student
         */
        students.add(student);


        /*
         * Create context
         */
        WebContext context = createContext(
                request,
                response
        );

        context.setVariable(
                "student",
                student
        );

        context.setVariable(
                "students",
                students
        );

        context.setVariable(
                "message",
                "Student registered successfully!"
        );


        /*
         * Show page
         */
        templateEngine.process(
                "student-form",
                context,
                response.getWriter()
        );
    }


    /*
     * Helper method
     */
    private WebContext createContext(
            HttpServletRequest request,
            HttpServletResponse response
    ) {

        JakartaServletWebApplication application =
                JakartaServletWebApplication.buildApplication(
                        getServletContext()
                );

        IWebExchange exchange =
                application.buildExchange(
                        request,
                        response
                );

        response.setContentType(
                "text/html;charset=UTF-8"
        );

        return new WebContext(exchange);
    }
}
```

---

# 5. Create `student-form.html`

Location:

```text
src/main/webapp/WEB-INF/templates/student-form.html
```

Here we'll use **almost every Thymeleaf feature you asked about**.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<head>

    <meta charset="UTF-8">

    <title>Student Registration</title>

</head>

<body>

<h1>Student Registration</h1>


<!-- ================================= -->
<!-- th:if -->
<!-- ================================= -->

<p th:if="${message != null}"
   th:text="${message}">

    Message

</p>


<!-- ================================= -->
<!-- th:unless -->
<!-- ================================= -->

<p th:unless="${message != null}">

    Please fill the form below.

</p>


<!-- ================================= -->
<!-- th:action + th:object -->
<!-- ================================= -->

<form th:action="@{/student}"
      th:object="${student}"
      method="post">


    <!-- ================================= -->
    <!-- HIDDEN FIELD -->
    <!-- ================================= -->

    <input type="hidden"
           th:field="*{id}">


    <!-- ================================= -->
    <!-- NAME -->
    <!-- ================================= -->

    <label>Name:</label>

    <input type="text"
           th:field="*{name}"
           placeholder="Enter your name">

    <br><br>


    <!-- ================================= -->
    <!-- EMAIL -->
    <!-- ================================= -->

    <label>Email:</label>

    <input type="email"
           th:field="*{email}"
           placeholder="Enter your email">

    <br><br>


    <!-- ================================= -->
    <!-- AGE -->
    <!-- ================================= -->

    <label>Age:</label>

    <input type="number"
           th:field="*{age}">

    <br><br>


    <!-- ================================= -->
    <!-- COURSE -->
    <!-- ================================= -->

    <label>Course:</label>

    <select th:field="*{course}">

        <option value="">
            Select Course
        </option>

        <option value="Java">
            Java
        </option>

        <option value="Python">
            Python
        </option>

        <option value="Web Development">
            Web Development
        </option>

    </select>

    <br><br>


    <!-- ================================= -->
    <!-- GENDER -->
    <!-- ================================= -->

    <label>Gender:</label>

    <input type="radio"
           th:field="*{gender}"
           value="Male">

    Male


    <input type="radio"
           th:field="*{gender}"
           value="Female">

    Female

    <br><br>


    <!-- ================================= -->
    <!-- CITY -->
    <!-- ================================= -->

    <label>City:</label>

    <select th:field="*{city}">

        <option value="">
            Select City
        </option>

        <option value="Lucknow">
            Lucknow
        </option>

        <option value="Delhi">
            Delhi
        </option>

        <option value="Mumbai">
            Mumbai
        </option>

    </select>

    <br><br>


    <!-- ================================= -->
    <!-- CHECKBOX -->
    <!-- ================================= -->

    <label>

        <input type="checkbox"
               th:field="*{active}">

        Active Student

    </label>

    <br><br>


    <button type="submit">
        Register Student
    </button>

</form>


<hr>


<!-- ================================= -->
<!-- th:href -->
<!-- ================================= -->

<a th:href="@{/student}">
    Refresh Student Form
</a>


<hr>


<h2>Registered Students</h2>


<!-- ================================= -->
<!-- th:each -->
<!-- ================================= -->

<table border="1">

    <thead>

        <tr>

            <th>ID</th>

            <th>Name</th>

            <th>Email</th>

            <th>Age</th>

            <th>Course</th>

            <th>Gender</th>

            <th>City</th>

            <th>Status</th>

        </tr>

    </thead>


    <tbody>

        <tr th:each="s : ${students}">


            <td th:text="${s.id}">
                ID
            </td>


            <td th:text="${s.name}">
                Name
            </td>


            <td th:text="${s.email}">
                Email
            </td>


            <td th:text="${s.age}">
                Age
            </td>


            <td th:text="${s.course}">
                Course
            </td>


            <td th:text="${s.gender}">
                Gender
            </td>


            <td th:text="${s.city}">
                City
            </td>


            <!-- ================================= -->
            <!-- th:if + th:unless -->
            <!-- ================================= -->

            <td>

                <span th:if="${s.active}">
                    Active
                </span>

                <span th:unless="${s.active}">
                    Inactive
                </span>

            </td>

        </tr>

    </tbody>

</table>


<hr>


<!-- ================================= -->
<!-- th:switch + th:case -->
<!-- ================================= -->

<h2>Course Information</h2>


<div th:switch="${student.course}">

    <p th:case="'Java'">

        You selected Java.
        Great for backend development.

    </p>


    <p th:case="'Python'">

        You selected Python.
        Great for programming and data science.

    </p>


    <p th:case="'Web Development'">

        You selected Web Development.

    </p>


    <p th:case="*">

        Please select a course.

    </p>

</div>


</body>

</html>
```

---

# 6. Where is `th:value`?

We have already used `th:field`, which internally generates the appropriate `value` attributes.

But since you specifically want students to understand **`th:value`**, let's add a separate example.

Add this before the form:

```html
<h2>Simple Dynamic Value</h2>

<input type="text"
       th:value="${student.name}"
       placeholder="Student Name">
```

If:

```java
student.setName("John");
```

Thymeleaf generates approximately:

```html
<input type="text"
       value="John"
       placeholder="Student Name">
```

So now you can directly compare:

```html
th:value="${student.name}"
```

versus:

```html
th:field="*{name}"
```

### `th:value`

```text
Set the value
```

### `th:field`

```text
Bind the input to the form object's property
```

---

# 7. What happens when you open the application?

Open:

```text
http://localhost:8080/Servlet4/student
```

The Servlet executes:

```java
doGet()
```

Then:

```java
Student student = new Student();
```

Then:

```java
context.setVariable(
    "student",
    student
);
```

Now Thymeleaf receives:

```text
student
students
message
```

Then Thymeleaf renders:

```text
student-form.html
```

---

# 8. The most important part: `th:object`

Look at:

```html
<form th:action="@{/student}"
      th:object="${student}"
      method="post">
```

We are saying:

```text
FORM
 │
 └── student object
```

Then:

```html
<input th:field="*{name}">
```

means:

```text
student.name
```

And:

```html
<input th:field="*{email}">
```

means:

```text
student.email
```

And:

```html
<input th:field="*{age}">
```

means:

```text
student.age
```

This is why `*{}` is called a **selection expression** — it works relative to the object established by `th:object`.

---

# 9. What happens when Submit is clicked?

Suppose the user enters:

```text
Name: John
Email: John@gmail.com
Age: 25
Course: Java
Gender: Male
City: Lucknow
Active: ✓
```

The browser sends:

```text
POST /Servlet4/student
```

with form data approximately:

```text
id=0
name=John
email=John@gmail.com
age=25
course=Java
gender=Male
city=Lucknow
active=true
```

Then the Servlet receives:

```java
request.getParameter("name");
```

```java
request.getParameter("email");
```

etc.

Then we create:

```java
Student student = new Student(
        id,
        name,
        email,
        age,
        course,
        gender,
        city,
        active
);
```

And add it to:

```java
students.add(student);
```

Then `th:each` displays all students.

---

# 10. How every Thymeleaf feature fits together

This is the part I'd emphasize to your students:

```text
                    THYMELEAF
                        │
        ┌───────────────┼────────────────┐
        │               │                │
       DATA            URL              FORM
        │               │                │
      ${}              @{}          th:object
        │               │                │
    th:text          th:href         th:field
    th:value         th:action
        │
        │
   ┌────┴─────┐
   │          │
 th:if    th:unless
   │
 th:switch
   │
 th:case
   │
 th:each
```

### Quick revision table

| Syntax | Meaning |
|---|---|
| `${name}` | Get a variable |
| `@{/student}` | Build a URL |
| `th:text` | Put text into an element |
| `th:value` | Set an element's value |
| `th:href` | Set an `<a>` URL |
| `th:action` | Set form submission URL |
| `th:object` | Bind a form to an object |
| `th:field` | Bind an input to an object property |
| `*{name}` | Access property of current object |
| `th:if` | Render if condition is true |
| `th:unless` | Render if condition is false |
| `th:switch` | Choose among cases |
| `th:case` | Individual switch case |
| `th:each` | Loop through a collection |
| `type="hidden"` | Send data without displaying it |

The **core form pattern** to remember is:

```html
<form th:action="@{/student}"
      th:object="${student}"
      method="post">

    <input th:field="*{name}">

    <input th:field="*{email}">

    <input th:field="*{age}">

    <button type="submit">
        Save
    </button>

</form>
```
