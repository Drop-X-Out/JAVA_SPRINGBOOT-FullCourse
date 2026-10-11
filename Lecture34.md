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
