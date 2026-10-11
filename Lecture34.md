## Thymeleaf Fragments — `th:fragment`, `th:replace`, `th:insert`

Fragments are used when you have **HTML code that you want to reuse on multiple pages**.

For example, suppose every page has the same:

- Header
- Navbar
- Footer

Instead of writing the same HTML again and again, create it **once** and reuse it.

---

# 1. `th:fragment`

`th:fragment` is used to **create a reusable piece of HTML**.

### `fragments.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<body>

    <header th:fragment="header">
        <h1>My Website</h1>
        <nav>
            <a href="#">Home</a>
            <a href="#">About</a>
            <a href="#">Contact</a>
        </nav>
    </header>

</body>
</html>
```

Here:

```html
<header th:fragment="header">
```

means:

> Create a fragment named `header`.

So `header` can now be reused on other pages.

---

# 2. `th:replace`

`th:replace` is used to **replace the current element with the fragment**.

Suppose we have:

```html
<div th:replace="~{fragments :: header}">
    Old Content
</div>
```

Thymeleaf finds:

```html
th:fragment="header"
```

from `fragments.html`.

The result is effectively:

```html
<header>
    <h1>My Website</h1>
    <nav>
        <a href="#">Home</a>
        <a href="#">About</a>
        <a href="#">Contact</a>
    </nav>
</header>
```

The original `<div>` is **replaced**.

### Easy memory

> `replace` → **Replace my element**

---

# 3. `th:insert`

`th:insert` is used to **insert the fragment inside the current element**.

Example:

```html
<div th:insert="~{fragments :: header}">
    Old Content
</div>
```

The `<div>` remains, and the fragment is inserted **inside** it.

Conceptually:

```html
<div>

    <header>
        <h1>My Website</h1>
        <nav>
            <a href="#">Home</a>
            <a href="#">About</a>
            <a href="#">Contact</a>
        </nav>
    </header>

</div>
```

### Easy memory

> `insert` → **Keep my element and put the fragment inside it**

---

# `replace` vs `insert`

This is the most important difference.

### `th:replace`

```html
<div th:replace="~{fragments :: header}">
</div>
```

The `<div>` **goes away**.

```text
<div>
   ↓
REPLACED
   ↓
<header>...</header>
```

### `th:insert`

```html
<div th:insert="~{fragments :: header}">
</div>
```

The `<div>` **stays**.

```text
<div>
   ↓
<header>...</header>
</div>
```

---

# Complete Beginner Example

### Folder structure

```text
WEB-INF/
└── templates/
    ├── fragments.html
    └── home.html
```

### `fragments.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<body>

<header th:fragment="header">

    <h1>PW IOI</h1>

    <nav>
        <a href="#">Home</a>
        <a href="#">Courses</a>
        <a href="#">Contact</a>
    </nav>

</header>


<footer th:fragment="footer">

    <p>© 2026 PW IOI</p>

</footer>

</body>
</html>
```

We created **two fragments**:

```text
header
footer
```

---

### `home.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<head>
    <title>Home</title>
</head>

<body>

    <!-- Header -->
    <div th:replace="~{fragments :: header}"></div>

    <h2>Welcome to Home Page</h2>

    <p>This is the home page.</p>

    <!-- Footer -->
    <div th:replace="~{fragments :: footer}"></div>

</body>

</html>
```

Thymeleaf sees:

```html
<div th:replace="~{fragments :: header}"></div>
```

and looks inside:

```text
fragments.html
       ↓
th:fragment="header"
```

Then it replaces the `<div>` with that fragment.

---

# What does `~{fragments :: header}` mean?

Break it into pieces:

```text
~{fragments :: header}
 │       │       │
 │       │       └── Fragment name
 │       └────────── Template name
 └────────────────── Thymeleaf fragment expression
```

So:

```html
~{fragments :: header}
```

means:

> Find the `header` fragment inside `fragments.html`.

---

## One very easy way to remember

```text
th:fragment → CREATE
th:replace  → REPLACE
th:insert   → INSERT
```

### Flow

```text
fragments.html

     th:fragment
          ↓
      CREATE
          ↓
   ┌─────────────┐
   │   header    │
   └─────────────┘
          ↓
   Other HTML page
          ↓
   ┌─────────────┐
   │ th:replace  │ → replaces element
   └─────────────┘

   OR

   ┌─────────────┐
   │  th:insert  │ → keeps element
   └─────────────┘
```
# Thymeleaf Utility Objects: Strings (`#strings`)

Thymeleaf provides the `#strings` utility object to perform common string operations, such as converting text to uppercase, converting text to lowercase, finding the length of a string, and checking whether a string contains specific text.

## 1. `#strings.toUpperCase(name)`

Converts a string into uppercase letters.

```html
<p th:text="${#strings.toUpperCase(name)}"></p>
```

**Java:**
```java
String name = "John";
context.setVariable("name", name);
```

**Output:**
```text
John
```

## 2. `#strings.toLowerCase(name)`

Converts a string into lowercase letters.

```html
<p th:text="${#strings.toLowerCase(name)}"></p>
```

**Output:**
```text
John
```

## 3. `#strings.length(name)`

Returns the number of characters in a string.

```html
<p th:text="${#strings.length(name)}"></p>
```

**Output:**
```text
5
```

For `"John"`, the length is `5` because the string contains five characters.

## 4. `#strings.contains(name, 'it')`

Checks whether a string contains the specified text. It returns `true` if the text is found and `false` otherwise.

```html
<p th:text="${#strings.contains(name, 'it')}"></p>
```

**Java:**
```java
String name = "Smith";
context.setVariable("name", name);
```

**Output:**
```text
true
```

This is because `"Smith"` contains `"it"`.

### Complete Example

**Java Servlet:**
```java
String name = "Smith";

context.setVariable("name", name);
```

**Thymeleaf HTML:**
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>String Utility Example</title>
</head>
<body>

    <h2>Original Name: <span th:text="${name}"></span></h2>

    <p>Uppercase:
        <span th:text="${#strings.toUpperCase(name)}"></span>
    </p>

    <p>Lowercase:
        <span th:text="${#strings.toLowerCase(name)}"></span>
    </p>

    <p>Length:
        <span th:text="${#strings.length(name)}"></span>
    </p>

    <p>Contains 'it':
        <span th:text="${#strings.contains(name, 'it')}"></span>
    </p>

</body>
</html>
```

**Output:**
```text
Original Name: Smith
Uppercase: SMITH
Lowercase: smith
Length: 5
Contains 'it': true
```

## Quick Revision

| Method | Purpose |
|---|---|
| `#strings.toUpperCase(name)` | Converts text to uppercase |
| `#strings.toLowerCase(name)` | Converts text to lowercase |
| `#strings.length(name)` | Returns the string length |
| `#strings.contains(name, 'it')` | Checks whether text contains a substring |

**Remember:** `#strings` is a Thymeleaf utility object that makes string operations easy directly inside HTML templates.
# Thymeleaf Utility Objects: Lists (`#lists`)

Thymeleaf provides the `#lists` utility object to perform common operations on Java lists, such as checking the list size, checking whether the list is empty, and checking whether a list contains a particular element.

## 1. `#lists.size(employees)`

Returns the total number of elements in a list.

**Java Servlet:**
```java
List<String> employees = List.of("John", "Rahul", "Aman");

context.setVariable("employees", employees);
```

**Thymeleaf HTML:**
```html
<p>
    Total Employees:
    <span th:text="${#lists.size(employees)}">0</span>
</p>
```

**Output:**
```text
Total Employees: 3
```

Explanation: The list contains three employees, so the size is `3`.

## 2. `#lists.isEmpty(employees)`

Checks whether a list is empty. Returns `true` if the list has no elements; otherwise, returns `false`.

**Java Servlet:**
```java
List<String> employees = List.of();

context.setVariable("employees", employees);
```

**Thymeleaf HTML:**
```html
<p th:if="${#lists.isEmpty(employees)}">
    No employees found.
</p>

<p th:unless="${#lists.isEmpty(employees)}">
    Employees are available.
</p>
```

**Output:**
```text
No employees found.
```

Explanation: The list has no elements, so `#lists.isEmpty(employees)` returns `true`.

## 3. `#lists.contains(employees, obj)`

Checks whether a list contains a particular element. Returns `true` if the element is found; otherwise, returns `false`.

**Java Servlet:**
```java
List<String> employees = List.of("John", "Rahul", "Aman");

context.setVariable("employees", employees);
context.setVariable("name", "John");
```

**Thymeleaf HTML:**
```html
<p th:if="${#lists.contains(employees, name)}">
    Employee found!
</p>

<p th:unless="${#lists.contains(employees, name)}">
    Employee not found!
</p>
```

**Output:**
```text
Employee found!
```

Explanation: The list contains `"John"`, which matches the value stored in `name`, so the method returns `true`.

Here, `obj` means the object or value you want to search for in the list.

## Complete Example

**Java Servlet:**
```java
List<String> employees = List.of("John", "Rahul", "Aman");

context.setVariable("employees", employees);
context.setVariable("name", "John");
```

**Thymeleaf HTML:**
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>List Utility Example</title>
</head>
<body>

    <h2>Employee List Operations</h2>

    <p>
        Total Employees:
        <span th:text="${#lists.size(employees)}">0</span>
    </p>

    <p th:if="${#lists.isEmpty(employees)}">
        No employees found.
    </p>

    <p th:unless="${#lists.isEmpty(employees)}">
        Employees are available.
    </p>

    <p th:if="${#lists.contains(employees, name)}">
        <span th:text="${name}"></span> was found.
    </p>

    <p th:unless="${#lists.contains(employees, name)}">
        <span th:text="${name}"></span> was not found.
    </p>

    <h3>All Employees</h3>

    <ul>
        <li th:each="employee : ${employees}"
            th:text="${employee}">Employee</li>
    </ul>

</body>
</html>
```

**Output:**
```text
Employee List Operations

Total Employees: 3

Employees are available.

John was found.

All Employees
- John
- Rahul
- Aman
```

## Quick Revision

| Method | Purpose |
|---|---|
| `#lists.size(employees)` | Returns the number of elements |
| `#lists.isEmpty(employees)` | Checks whether the list is empty |
| `#lists.contains(employees, obj)` | Checks whether the list contains an element |

**Remember:** `#lists` is a Thymeleaf utility object used to perform list operations directly inside HTML templates.

# Thymeleaf Utility Objects: Numbers (`#numbers`)

Thymeleaf provides the `#numbers` utility object to format numbers, such as ages, salaries, prices, and percentages, directly inside HTML templates.

## 1. `#numbers.formatInteger(age, 2)`

Formats an integer with a specified minimum number of integer digits.

**Java Servlet:**
```java
int age = 25;

context.setVariable("age", age);
```

**Thymeleaf HTML:**
```html
<p>
    Age:
    <span th:text="${#numbers.formatInteger(age, 2)}">25</span>
</p>
```

**Output:**
```text
Age: 25
```

### Example with a single-digit age

```java
int age = 5;

context.setVariable("age", age);
```

```html
<p th:text="${#numbers.formatInteger(age, 2)}"></p>
```

**Output:**
```text
05
```

**Explanation:** The second argument, `2`, specifies the minimum number of integer digits. If the number has fewer than two digits, Thymeleaf adds leading zeros.

## 2. `#numbers.formatDecimal(salary, 1, 2)`

Formats a decimal number with a specified minimum number of integer digits and a fixed number of decimal digits.

**Java Servlet:**
```java
double salary = 45678.5678;

context.setVariable("salary", salary);
```

**Thymeleaf HTML:**
```html
<p>
    Salary:
    <span th:text="${#numbers.formatDecimal(salary, 1, 2)}">
        45678.57
    </span>
</p>
```

**Output:**
```text
Salary: 45,678.57
```

**Explanation:**

```text
#numbers.formatDecimal(salary, 1, 2)
                                ↑  ↑
                                │  └── Number of decimal digits
                                └───── Minimum integer digits
```

- `1`: At least one digit before the decimal point.
- `2`: Exactly two digits after the decimal point.
- Thymeleaf rounds the number to two decimal places.
- The default decimal format uses grouping separators for large numbers.

### Another Example

**Java:**
```java
double salary = 25000.5;

context.setVariable("salary", salary);
```

**HTML:**
```html
<p th:text="${#numbers.formatDecimal(salary, 1, 2)}"></p>
```

**Output:**
```text
25,000.50
```

## Complete Example

**Java Servlet:**
```java
int age = 5;
double salary = 45678.5678;

context.setVariable("age", age);
context.setVariable("salary", salary);
```

**Thymeleaf HTML:**
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Number Utility Example</title>
</head>
<body>

    <h2>Number Formatting</h2>

    <p>
        Age:
        <span th:text="${#numbers.formatInteger(age, 2)}">05</span>
    </p>

    <p>
        Salary:
        <span th:text="${#numbers.formatDecimal(salary, 1, 2)}">
            45,678.57
        </span>
    </p>

</body>
</html>
```

**Output:**
```text
Number Formatting

Age: 05
Salary: 45,678.57
```

## Quick Revision

| Method | Purpose |
|---|---|
| `#numbers.formatInteger(age, 2)` | Formats an integer with at least two digits |
| `#numbers.formatDecimal(salary, 1, 2)` | Formats a number with at least one integer digit and exactly two decimal digits |

**Remember:** Use `#numbers.formatInteger()` for integer formatting and `#numbers.formatDecimal()` when you need decimal-place formatting.

# Thymeleaf Utility Objects: Temporal (Date and Time)

Thymeleaf provides the `#temporals` utility object to format and display Java date and time objects, such as `LocalDate`, `LocalTime`, and `LocalDateTime`.

## 1. `#temporals.format(localDate, 'dd/MM/yyyy')`

Formats a date into the specified pattern.

**Java Servlet:**

```java
import java.time.LocalDate;

LocalDate localDate = LocalDate.of(2026, 10, 12);

context.setVariable("localDate", localDate);
```

**Thymeleaf HTML:**

```html
<p>
    Date:
    <span th:text="${#temporals.format(localDate, 'dd/MM/yyyy')}">
        12/10/2026
    </span>
</p>
```

**Output:**

```text
Date: 12/10/2026
```

### Understanding the Date Pattern

| Pattern | Meaning | Example |
|---|---|---|
| `dd` | Day (2 digits) | `12` |
| `MM` | Month (2 digits) | `10` |
| `yyyy` | Year (4 digits) | `2026` |
| `dd/MM/yyyy` | Day/Month/Year | `12/10/2026` |
| `yyyy-MM-dd` | Year-Month-Day | `2026-10-12` |
| `dd MMM yyyy` | Day, abbreviated month, year | `12 Oct 2026` |

**Important:** Use uppercase `MM` for the month. Lowercase `mm` represents minutes.

## 2. Complete Example with Current Date

**Java Servlet:**

```java
import java.time.LocalDate;

LocalDate localDate = LocalDate.now();

context.setVariable("localDate", localDate);
```

**Thymeleaf HTML:**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Date Formatting</title>
</head>
<body>

    <h2>Date and Time Formatting</h2>

    <p>
        Indian Date Format:
        <span th:text="${#temporals.format(localDate, 'dd/MM/yyyy')}">
            12/10/2026
        </span>
    </p>

    <p>
        ISO Date Format:
        <span th:text="${#temporals.format(localDate, 'yyyy-MM-dd')}">
            2026-10-12
        </span>
    </p>

    <p>
        Readable Date:
        <span th:text="${#temporals.format(localDate, 'dd MMM yyyy')}">
            12 Oct 2026
        </span>
    </p>

</body>
</html>
```

## 3. Formatting Date and Time Together

Java:

```java
import java.time.LocalDateTime;

LocalDateTime dateTime = LocalDateTime.of(2026, 10, 12, 14, 30);

context.setVariable("dateTime", dateTime);
```

Thymeleaf:

```html
<p th:text="${#temporals.format(dateTime, 'dd/MM/yyyy HH:mm:ss')}"></p>
```

Output:

```text
12/10/2026 14:30:00
```

Here, `HH` represents the hour in 24-hour format, `mm` represents minutes, and `ss` represents seconds.

## Quick Revision

| Expression | Purpose |
|---|---|
| `#temporals.format(localDate, 'dd/MM/yyyy')` | Formats a date |
| `#temporals.format(dateTime, 'dd/MM/yyyy HH:mm:ss')` | Formats a date and time |
| `#temporals.format(localDate, 'yyyy-MM-dd')` | Formats a date in year-month-day order |

**Remember:** `#temporals` formats Java date/time objects for display in Thymeleaf templates. It changes how the date appears, not the original date value.
