In Thymeleaf, `${...}` is an **expression syntax** used to access a value from the data sent by your Java code.

Think of it as:

> **“Give me the value of this variable.”**

### Example

Java:

```java
String name = "John";

context.setVariable("name", name);
```

Thymeleaf:

```html
<h1 th:text="${name}">
    Welcome
</h1>
```

Here:

```text
${name}
```

means:

```text
"Go to the Thymeleaf context and get the value stored under 'name'."
```

So:

```text
${name}
   ↓
John
```

And the final HTML becomes:

```html
<h1>John</h1>
```

### Why `${}`?

The `$` tells Thymeleaf:

> **I want to evaluate an expression.**

The `{ }` contains the expression:

```text
${name}
  ↑
 expression
```

You can also do calculations:

```html
<p th:text="${age + 1}"></p>
```

If:

```java
context.setVariable("age", 25);
```

then:

```text
${age + 1}
      ↓
26
```

You can access object properties too:

```html
<p th:text="${student.name}"></p>
```

So the basic pattern to teach students is:

```text
${variable}
     ↓
Get the value of the variable
```

and:

```text
th:text="${variable}"
        ↓
Put that value into the HTML element's text.
```
`th:value` means **“set the value of an HTML element using Thymeleaf.”**

The important difference is:

```text
th:text   → changes the text/content inside an element
th:value  → changes the value attribute of an element
```

### 1. `th:text`

```html
<h1 th:text="${name}">Name</h1>
```

If:

```java
context.setVariable("name", "John");
```

Thymeleaf produces:

```html
<h1>John</h1>
```

So `th:text` is commonly used for **displaying data**.

---

### 2. `th:value`

Most commonly, you'll use it with form inputs:

```html
<input type="text" th:value="${name}">
```

If:

```java
context.setVariable("name", "John");
```

Thymeleaf produces:

```html
<input type="text" value="John">
```

So the browser shows:

```text
┌──────────────────┐
│ John            │
└──────────────────┘
```

The value is **inside the input**, not normal HTML text.

---

### Easy way to remember

```text
th:text
   ↓
"What should I DISPLAY?"

th:value
   ↓
"What should be the VALUE of this element?"
```

### Real example

Suppose you're editing a student's information.

```html
<h2 th:text="${student.name}">
    Student Name
</h2>

<input type="text" th:value="${student.name}">
```

If the student's name is `John`:

```html
<h2>John</h2>

<input type="text" value="John">
```

**Important:** `${...}` is the value/expression, while `th:text` and `th:value` tell Thymeleaf **where/how to put that value**.
In Thymeleaf, you can use **`th:if`** to conditionally display an HTML element.

Think of it as:

> **`th:if` = show this element only if the condition is true.**

### Basic example

Java:

```java
int age = 25;

context.setVariable("age", age);
```

HTML:

```html
<p th:if="${age >= 18}">
    You are an adult.
</p>
```

Because:

```text
25 >= 18
```

is `true`, Thymeleaf sends:

```html
<p>You are an adult.</p>
```

---

### If the condition is false

```java
int age = 15;
```

Then:

```html
<p th:if="${age >= 18}">
    You are an adult.
</p>
```

The `<p>` element is **not included in the final HTML**.

So:

```text
age = 25 → show
age = 15 → don't show
```

### With a boolean

Java:

```java
boolean loggedIn = true;

context.setVariable("loggedIn", loggedIn);
```

HTML:

```html
<h2 th:if="${loggedIn}">
    Welcome, John!
</h2>
```

If `loggedIn` is `true` → displayed.

If `loggedIn` is `false` → not displayed.

### `th:if` vs Java `if`

Java:

```java
if (age >= 18) {
    // Java logic
}
```

Thymeleaf:

```html
<p th:if="${age >= 18}">
    You are an adult.
</p>
```

The **Java `if` controls server-side Java execution**, while **`th:if` controls whether an HTML element appears in the generated HTML**.

A simple way to remember:

```text
th:text   → What text?
th:value  → What value?
th:if     → Should I show it?
```
In Thymeleaf, **`th:unless`** is the opposite of **`th:if`**.

Think:

> **`th:unless` = show this element unless the condition is true.**

Or more simply:

```text
th:if      → show IF condition is true
th:unless  → show IF condition is false
```

### Example

Java:

```java
int age = 15;

context.setVariable("age", age);
```

Thymeleaf:

```html
<p th:unless="${age >= 18}">
    You are a minor.
</p>
```

The condition:

```text
age >= 18
15 >= 18
false
```

Because the condition is **false**, `th:unless` displays the paragraph:

```text
You are a minor.
```

---

### Compare `th:if` and `th:unless`

```html
<p th:if="${age >= 18}">
    You are an adult.
</p>

<p th:unless="${age >= 18}">
    You are a minor.
</p>
```

If:

```text
age = 25
```

Output:

```text
You are an adult.
```

If:

```text
age = 15
```

Output:

```text
You are a minor.
```

### Easy memory trick

```text
IF
 ↓
Condition TRUE → Show


UNLESS
 ↓
Condition TRUE → Hide
Condition FALSE → Show
```

So this:

```html
<p th:unless="${loggedIn}">
    Please login first.
</p>
```

means:

> **Show "Please login first" unless the user is logged in.**

If `loggedIn = true` → hidden.

If `loggedIn = false` → shown.

In Thymeleaf, **`th:switch` and `th:case`** are used when you want to choose **one option from multiple possibilities**.

Think of it like Java's `switch`.

```text
Java switch       → switch + case
Thymeleaf         → th:switch + th:case
```

### Basic example

Java:

```java
String role = "admin";

context.setVariable("role", role);
```

HTML:

```html
<div th:switch="${role}">

    <p th:case="'admin'">
        Welcome Admin
    </p>

    <p th:case="'student'">
        Welcome Student
    </p>

    <p th:case="'teacher'">
        Welcome Teacher
    </p>

    <p th:case="*">
        Unknown Role
    </p>

</div>
```

### How it works

Thymeleaf evaluates:

```html
th:switch="${role}"
```

If:

```text
role = "admin"
```

then:

```text
admin → Welcome Admin
```

If:

```text
role = "student"
```

then:

```text
student → Welcome Student
```

If none of the cases match:

```html
<p th:case="*">
    Unknown Role
</p>
```

`*` means **default case**.

---

### Compare with Java

Java:

```java
switch (role) {

    case "admin":
        System.out.println("Welcome Admin");
        break;

    case "student":
        System.out.println("Welcome Student");
        break;

    case "teacher":
        System.out.println("Welcome Teacher");
        break;

    default:
        System.out.println("Unknown Role");
}
```

Thymeleaf:

```html
<div th:switch="${role}">

    <p th:case="'admin'">Welcome Admin</p>

    <p th:case="'student'">Welcome Student</p>

    <p th:case="'teacher'">Welcome Teacher</p>

    <p th:case="*">Unknown Role</p>

</div>
```

### Easy memory

```text
th:if
    ↓
one condition


th:unless
    ↓
opposite condition


th:switch
    ↓
multiple possible conditions
        ↓
    th:case
```

So for Thymeleaf:

```text
th:switch → choose
th:case   → option
th:case="*" → default
```

In Thymeleaf, **`th:each`** is used to **loop through a collection** and generate HTML for each item.

Think of it as:

> **`th:each` = repeat this HTML for every item.**

It is similar to Java's `for-each` loop.

### 1. Java

Suppose you have:

```java
List<String> students = List.of(
    "John",
    "Rahul",
    "Aman"
);

context.setVariable("students", students);
```

### 2. Thymeleaf

```html
<ul>

    <li th:each="student : ${students}"
        th:text="${student}">
    </li>

</ul>
```

Thymeleaf sees:

```text
students
   ↓
["John", "Rahul", "Aman"]
   ↓
th:each
   ↓
repeat <li>
```

The final HTML becomes conceptually:

```html
<ul>

    <li>John</li>

    <li>Rahul</li>

    <li>Aman</li>

</ul>
```

### Understand this syntax

```html
th:each="student : ${students}"
```

Break it down:

```text
student
   ↓
current item

:
   ↓
"from"

${students}
   ↓
collection/list
```

So:

```text
student : ${students}
```

means:

> **Take each item from the `students` list and temporarily call it `student`.**

---

### Real example with Student objects

Java:

```java
context.setVariable("students", students);
```

Suppose each student has:

```text
name
age
```

HTML:

```html
<table>

    <tr th:each="student : ${students}">

        <td th:text="${student.name}"></td>

        <td th:text="${student.age}"></td>

    </tr>

</table>
```

If you have:

```text
John   25
Rahul   22
Aman    21
```

Thymeleaf generates:

```text
John   25
Rahul   22
Aman    21
```

### Compare with Java

Java:

```java
for (String student : students) {

    System.out.println(student);

}
```

Thymeleaf:

```html
<li th:each="student : ${students}"
    th:text="${student}">
</li>
```

So remember:

```text
th:text    → put a value into HTML
th:value   → set an HTML value
th:if      → show if true
th:unless  → show if false
th:switch  → choose between cases
th:each    → repeat for every item
```
