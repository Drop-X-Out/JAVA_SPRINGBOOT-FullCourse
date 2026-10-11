# Servlet Technology — Introduction, Tomcat Setup & First Web Application

## 1. Where Are We in Our Learning Journey?

Before starting Servlet technology, let's revise what we have learned so far.

We have already learned how Java communicates with a database using JDBC and how Hibernate makes database operations easier through ORM. We have also learned Maven for managing dependencies and building Java projects.

Now we are moving to a new part of web development: **connecting the frontend to the server.**

### What have we learned so far?

| Technology | What we have learned |
|---|---|
| JDBC | Connecting Java applications to a database and executing SQL queries |
| Hibernate | Working with database records using Java objects and ORM |
| Maven | Managing project dependencies, project structure, and builds |
| HTML and CSS | Creating the structure and design of a frontend |

### Understanding the two parts of our application

We can divide our learning into two parts.

**Part 1: Database and Server**

We have already studied how Java code communicates with a database using JDBC and Hibernate.

```text
Java Application
       |
       v
JDBC / Hibernate
       |
       v
    Database
```

**Part 2: Frontend and Server**

Now we will learn how a browser sends a request to a Java server and receives a response.

```text
Frontend (HTML)
       |
       | HTTP Request
       v
Java Server (Servlet)
       |
       | HTTP Response
       v
Frontend (Browser)
```

**Part 3: Connecting Everything Together — Future Lectures**

After understanding both parts separately, we will combine them.

```text
Frontend
   |
   v
Servlet (Server)
   |
   v
Hibernate / JDBC
   |
   v
Database
```

### Why are we learning in this order?

Imagine building a student registration application.

First, we need to understand how Java saves student information in a database. We have studied that using JDBC and Hibernate.

Now, we need to understand how a student can submit information through an HTML form and send it to Java.

Once we understand both processes, we can connect the frontend, server, and database to build a complete application.

**Remember:** We are not connecting the frontend to the database directly. We are learning the frontend-to-server connection first, and we will integrate the database in future lectures.

---

## 2. What Is Servlet Technology?

A **Servlet** is a Java class that runs inside a servlet container and handles requests from clients, such as web browsers.

Servlets are commonly used to build the server-side part of Java web applications.

For example, suppose we create a login page using HTML.

The user enters a username and password and clicks the Login button. The browser sends the request to the server, and the server-side Java code processes it.

A servlet can receive that request and generate a response.

### Example of a real-world application

Consider a student registration application.

1. A student opens a registration page.
2. The student enters their details.
3. The browser sends the request to the server.
4. The servlet receives the request.
5. The servlet processes the information.
6. The server sends a response back to the browser.

### Deep one-liner

> A Servlet is a server-side Java component that receives HTTP requests, executes application logic, and generates HTTP responses.

### Why do we need Servlets?

HTML is used to create the frontend, but HTML alone cannot execute Java backend code.

Similarly, a Java class that runs as a normal console application does not automatically become accessible through an HTTP URL.

We need a web runtime that can receive browser requests and invoke our Java code. Servlets provide the request-handling component, and a servlet container such as Tomcat manages their execution.

---

## 3. What Is a Server?

A server is a system or software that provides services or resources to clients.

For example, when you open a website in a browser, your browser acts as a client and sends a request to a server.

The server processes the request and returns a response.

### Can our own computer act as a server?

Yes. During development, we can use our own computer as a local server.

For example, when we run a Java web application using Tomcat, our computer can receive requests at an address such as:

```text
http://localhost:8080/
```

Here:

- `localhost` refers to the same computer on which the server is running.
- `8080` is the port used by our Tomcat instance in this example.

This setup allows us to develop and test web applications without deploying them to a publicly accessible server.

### Deep one-liner

> A server receives requests from clients, processes them according to its responsibilities, and returns the appropriate response.

---

## 4. What Is Apache Tomcat?

Apache Tomcat is an open-source Java web server and servlet container.

It provides the runtime environment needed to execute servlet-based Java web applications.

We can write servlet code in Java, but we need a compatible servlet container to run that code as a web application.

That is why we use Tomcat.

### Why do we need Tomcat?

Tomcat helps us:

- Run Java servlet applications.
- Receive HTTP requests from browsers.
- Identify the requested web application.
- Invoke the appropriate servlet.
- Send HTTP responses back to the browser.
- Manage the servlet lifecycle.

### How does Tomcat fit into our application?

```text
Browser
   |
   | HTTP Request
   v
Tomcat
   |
   v
Servlet
   |
   | HTTP Response
   v
Browser
```

At this stage, we are only learning how to run a web application using Tomcat. We will write servlet code and connect the frontend in the upcoming lessons.

### Deep one-liner

> Tomcat is the runtime that hosts our web application and manages the execution of servlet components.

---

## 5. Download and Install Apache Tomcat

For this lesson, we will use **Apache Tomcat 10.1 with JDK 17**.

Tomcat 10.1 supports Jakarta Servlet 6.0 and requires Java 11 or later. JDK 17 is a suitable choice for our setup.

### Step 1: Download Tomcat

Open the official website:

https://tomcat.apache.org/

Navigate to the Tomcat 10.1 download page:

https://tomcat.apache.org/download-10.cgi

Under the binary distributions, download the ZIP archive for Windows.

For example, choose the 64-bit Windows ZIP distribution if it matches your system.

### Step 2: Extract the downloaded file

After downloading:

1. Locate the ZIP file in your Downloads folder.
2. Extract it to a convenient location.
3. Open the extracted Tomcat directory.

You should see a structure similar to this:

```text
apache-tomcat-10.1/
│
├── bin/
├── conf/
├── lib/
├── logs/
├── temp/
├── webapps/
└── work/
```

### Step 3: Understand the important folders

| Folder | Purpose |
|---|---|
| `bin` | Contains scripts to start and stop Tomcat |
| `conf` | Contains server configuration files |
| `lib` | Contains libraries used by Tomcat |
| `logs` | Stores server logs |
| `webapps` | Common deployment location for web applications |
| `work` | Stores generated working files |

The most important folders for today's practical are `bin` and `webapps`.

### Step 4: Check Java installation

Open Command Prompt and run:

```bash
java -version
```

You should see information about the installed Java version.

If Java is not installed or the command is not recognized, install a compatible JDK and configure your Java environment.

### Step 5: Start Tomcat

Open the extracted Tomcat folder and navigate to `bin`.

On Windows, execute:

```bat
startup.bat
```

Alternatively, run the following command from the Tomcat installation directory:

```bat
bin\startup.bat
```

A Tomcat console or process should start, depending on the environment.

### Step 6: Open Tomcat in the browser

Open:

http://localhost:8080/

If Tomcat has started successfully and port 8080 is available, you should see the Tomcat welcome page.

### Step 7: Stop Tomcat

When you want to stop the server, execute:

```bat
bin\shutdown.bat
```

### Important distinction

The Tomcat welcome page confirms that the Tomcat server is running. It does not yet mean that our own Maven web application has been created or deployed.

In the next section, we will create our own web application.

---

## 6. What Is Maven Web Application Packaging?

We have already studied Maven, so we will reuse that knowledge.

Maven is a build automation and dependency management tool for Java projects.

It helps us maintain a standard project structure, download dependencies, compile Java code, and package our application.

For a servlet-based web application, Maven commonly generates a **WAR file**.

WAR stands for **Web Application Archive**.

A WAR file packages the web application's resources and compiled classes into a deployable archive.

### JAR vs. WAR

| JAR | WAR |
|---|---|
| Java Archive | Web Application Archive |
| Commonly used for libraries and standalone Java applications | Used to package web applications |
| Can contain compiled Java classes and resources | Can contain web resources, compiled classes, and web application configuration |
| Commonly used with Java application launchers | Commonly deployed to a servlet container such as Tomcat |

### Deep one-liner

> Maven manages the project and its build, while WAR packaging prepares a web application for deployment to a compatible servlet container.

---

## 7. Create a Maven Web Application in IntelliJ IDEA

Now we will create our first web application.

We are not going to build a complete servlet program yet. Our goal is to create the project, understand its structure, configure Tomcat, and open the application's root page.

### Step 1: Open IntelliJ IDEA

Open IntelliJ IDEA and choose the option to create a new project.

Depending on your IntelliJ IDEA version, the exact labels may differ.

### Step 2: Select the Maven project setup

Choose the Maven build system for the project.

Configure the project with a name such as:

```text
servlet-demo
```

Choose a compatible JDK, such as JDK 17.

If your IntelliJ IDEA version provides a Web Application or Maven Web Application template, select it. If it does not, create a Maven project and add the web application structure manually.

**Important:** A plain Maven quickstart project is usually intended for a standard Java application. For our current lesson, we want a web application that can be deployed to Tomcat.

### Step 3: Understand the project configuration

The project will have a `pom.xml` file.

A minimal Maven web application configuration looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>servlet-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>war</packaging>

</project>
```

### Line-by-line explanation of `pom.xml`

**1. XML declaration**

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

This specifies the XML version and character encoding used in the configuration file.

**2. Project root element**

```xml
<project>
```

The `project` element is the root element of a Maven project configuration.

In the complete file, it also contains the Maven namespace and schema attributes that help identify the document format.

**3. Model version**

```xml
<modelVersion>4.0.0</modelVersion>
```

This specifies the POM model version Maven uses to interpret the project configuration.

**4. Group ID**

```xml
<groupId>com.example</groupId>
```

The `groupId` identifies the group or organization associated with the project.

**5. Artifact ID**

```xml
<artifactId>servlet-demo</artifactId>
```

The `artifactId` is the name of our project artifact.

**6. Version**

```xml
<version>1.0-SNAPSHOT</version>
```

This identifies the current project version. `SNAPSHOT` conventionally indicates a version that is still under development.

**7. Packaging**

```xml
<packaging>war</packaging>
```

This is particularly important for our lesson.

It tells Maven to package the application as a WAR file rather than using the default JAR packaging.

For example, the build will generate an archive similar to:

```text
target/servlet-demo.war
```

The actual filename can depend on the project's artifact ID, version, and build configuration.

---

## 8. Understand the Maven Web Application File Structure

A typical Maven web application follows this structure:

```text
servlet-demo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │
        ├── resources/
        │
        └── webapp/
```

Let's understand each part.

### `pom.xml`

This file contains Maven project configuration, dependencies, and build settings.

### `src/main/java/`

This directory is used for Java source code.

In future lessons, we will create servlet classes here.

For example:

```text
src/main/java/com/example/HelloServlet.java
```

### `src/main/resources/`

This directory is used for application resources that need to be available on the Java classpath.

### `src/main/webapp/`

This directory contains the web application's web resources.

For example:

```text
src/main/webapp/
    ├── index.html
    ├── css/
    ├── js/
    └── images/
```

The HTML files we create for the frontend will be placed here.

### Why is this structure important?

Maven follows a standard directory structure. It knows where to find Java source code, resources, and web application files during the build.

When Maven packages the application as a WAR, it collects the relevant files into the archive.

**Remember:** `src/main/webapp` is the source directory. The WAR file in `target` is the packaged application.

---

## 9. Our First Run: No Servlet Code Yet

At this stage, we are not writing servlet logic.

We only want to verify that:

1. Our Maven web project exists.
2. Our web application structure is correct.
3. Tomcat can run the project.
4. We can open the application's root URL in the browser.

### Does an empty webapp automatically show a page?

Not necessarily.

If the web application does not contain a welcome page and no welcome resource is configured, opening its root URL may result in a 404 response.

To make this first practical easy to verify, create a minimal `index.html` file.

Place it here:

```text
src/main/webapp/index.html
```

Add the following simple HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Servlet Demo</title>
</head>
<body>
    <h1>Welcome to Our First Web Application</h1>
    <p>Tomcat is running our Maven web project.</p>
</body>
</html>
```

This is only a static HTML page. We have not written any servlet code yet.

The purpose is to verify that our web application can be served by Tomcat.

### What happens when we open the page?

```text
Browser
   |
   | Request for /
   v
Tomcat
   |
   v
Web Application
   |
   v
index.html
   |
   | HTML Response
   v
Browser
```

Tomcat serves the static HTML page. The browser displays the content.

This is our first successful step in running a Java web application.

---

## 10. Why Do We Need to Build a WAR File?

After creating the Maven web application, we need to package it so that Tomcat can deploy it.

Maven provides a command for this.

Open the terminal in the project root and run:

```bash
mvn clean package
```

Let's understand the command.

### `mvn`

This invokes Maven.

### `clean`

This removes the previous build output, usually from the `target` directory.

### `package`

This runs the build lifecycle through the packaging phase. Maven compiles the project, processes the web resources, and packages the application according to its configuration.

If the build succeeds, we should see a WAR file similar to:

```text
target/servlet-demo.war
```

### Deep one-liner

> The Maven package phase builds the project and creates a WAR archive that can be deployed to a compatible servlet container.

### Manual deployment using Tomcat

Without Smart Tomcat, we can deploy the application manually.

1. Run `mvn clean package`.
2. Open the project's `target` directory.
3. Locate `servlet-demo.war`.
4. Copy it into the Tomcat installation's `webapps` directory.
5. Start or restart Tomcat as required.
6. Open the deployed application's URL.

For this project, the URL will commonly be:

```text
http://localhost:8080/servlet-demo/
```

The context path usually follows the WAR filename, unless the deployment has been configured differently.

Because our application contains `index.html`, Tomcat can serve it as the application's welcome page.

### Why can this process feel repetitive?

Every time we change the project, we may need to rebuild the WAR and copy it into the deployment directory.

That is manageable, but it can slow down development.

To simplify this workflow, we can use IntelliJ IDEA with Smart Tomcat.

---

## 11. What Is Smart Tomcat?

Smart Tomcat is an IntelliJ IDEA plugin that helps configure and launch a web application using an installed Tomcat server.

Instead of manually copying the WAR file into Tomcat's `webapps` directory for every development run, we can configure the plugin to launch the application from the project or its configured deployment directory.

### Why are we using Smart Tomcat?

For our current practical, it helps us:

- Configure the Tomcat installation in IntelliJ IDEA.
- Launch the web application from the IDE.
- View the server logs in the IDE.
- Open the application using the configured URL and context path.
- Avoid repeating manual deployment steps during development.

**Important:** Smart Tomcat does not replace Maven or Tomcat. Maven still builds the project, and Tomcat still runs the web application. The plugin makes the development workflow more convenient.

### Deep one-liner

> Smart Tomcat integrates Tomcat launching into IntelliJ IDEA so we can run and test our web application more conveniently during development.

---

## 12. Install Smart Tomcat in IntelliJ IDEA

### Step 1: Open Settings

In IntelliJ IDEA:

1. Open **File → Settings** on Windows.
2. Select **Plugins**.
3. Open the Marketplace tab.

### Step 2: Search for Smart Tomcat

Search for:

```text
Smart Tomcat
```

You can also find the plugin here:

https://plugins.jetbrains.com/plugin/9492-smart-tomcat

Install the plugin and restart IntelliJ IDEA if prompted.

### Step 3: Configure the Tomcat server

After installation:

1. Open the Run/Debug Configurations menu.
2. Add a new Smart Tomcat configuration.
3. Select the Tomcat installation directory you downloaded earlier.
4. Select the project or deployment directory according to the plugin's configuration options.
5. Configure the context path if required.
6. Save the configuration.

The exact fields can vary between plugin versions.

### Step 4: Run the project

Click the Run button in IntelliJ IDEA.

The IDE should start Tomcat using the configuration we created.

Check the Run console for startup messages and the application URL.

### Step 5: Open the application URL

Open the URL shown by the configured server or run console.

For example, if the configured port is `8080` and the context path is `/servlet-demo`, the URL will be:

```text
http://localhost:8080/servlet-demo/
```

If the context path is empty, the application may instead be available at the root URL:

```text
http://localhost:8080/
```

The actual URL depends on the Smart Tomcat configuration.

### Step 6: Verify the page

You should see:

**Welcome to Our First Web Application**

This confirms that the application is being served through Tomcat.

If the page does not appear, verify the deployment directory, context path, port, and presence of `index.html`.

---

### Important difference

| Field | Example | Meaning |
|---|---|---|
| Project Group ID | `com.example` | Identifies our project group |
| Project Artifact ID | `servlet-demo` | Identifies our application |
| Archetype Group ID | `org.apache.maven.archetypes` | Identifies the template provider |
| Archetype Artifact ID | `maven-archetype-webapp` | Identifies the web application template |

If IntelliJ provides an archetype selection screen, select `maven-archetype-webapp` and configure the project coordinates.

If the archetype is unavailable in your IDE, create a Maven project and configure the WAR packaging and web application structure as shown below.

---

## 6. Understand the Project Folder Structure

After generating the project, we should have a structure similar to this:

```text
servlet-demo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │   └── com/
        │       └── example/
        │           └── HelloServlet.java
        │
        ├── resources/
        │
        └── webapp/
            ├── index.html
            │
            └── WEB-INF/
                └── web.xml
```

The generated files can vary depending on the archetype and IDE. We will ensure the required files and directories exist.

### `pom.xml`

This file contains the Maven project configuration, dependencies, and build settings.

### `src/main/java/`

This is where we write our Java classes.

Our servlet will be placed here:

```text
src/main/java/com/example/HelloServlet.java
```

### `src/main/resources/`

This directory stores resources that need to be available on the Java classpath.

### `src/main/webapp/`

This directory contains our web application's resources, such as HTML, CSS, JavaScript, and images.

For example:

```text
src/main/webapp/
├── index.html
├── css/
├── js/
└── images/
```

### `WEB-INF/`

This directory is used for web application configuration and protected application resources.

Files placed inside `WEB-INF` cannot ordinarily be requested directly through a browser URL.

### `web.xml`

This is the traditional deployment descriptor for a Java web application.

For our first servlet, we will use the `@WebServlet` annotation, so we do not need to configure the servlet mapping in `web.xml`.

**Deep one-liner:** Maven organizes the source code and web resources, while Tomcat deploys and executes the packaged web application.

---

## 7. Configure the `pom.xml`

Open the project's `pom.xml` and use the following configuration.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
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

### Line-by-line explanation

**1. Model version**

```xml
<modelVersion>4.0.0</modelVersion>
```

This identifies the POM model version Maven uses to interpret the project configuration.

**2. Group ID**

```xml
<groupId>com.example</groupId>
```

This identifies the group to which our application belongs.

**3. Artifact ID**

```xml
<artifactId>servlet-demo</artifactId>
```

This identifies our application artifact.

**4. Version**

```xml
<version>1.0-SNAPSHOT</version>
```

This identifies the current development version of our project.

**5. Packaging**

```xml
<packaging>war</packaging>
```

This tells Maven to create a Web Application Archive instead of the default JAR packaging.

**6. Java version**

```xml
<maven.compiler.release>17</maven.compiler.release>
```

This configures compilation for Java 17.

**7. Servlet API dependency**

```xml
<groupId>jakarta.servlet</groupId>
<artifactId>jakarta.servlet-api</artifactId>
<version>6.0.0</version>
```

This provides the servlet interfaces required to compile our Java code.

**8. Dependency scope**

```xml
<scope>provided</scope>
```

This tells Maven that the servlet API is supplied by the runtime environment. Tomcat provides the API when running the application, so Maven should not package another copy of it in the WAR.

**9. Final filename**

```xml
<finalName>servlet-demo</finalName>
```

This configures Maven to generate:

```text
target/servlet-demo.war
```

After editing the file, reload the Maven project in IntelliJ IDEA.

---

## 8. Write Our First Servlet Program

Now we will create our first Java servlet.

Create the following file:

```text
src/main/java/com/example/HelloServlet.java
```

### Complete code

```java
package com.example;

import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws IOException {

        response.setContentType("text/html;charset=UTF-8");

        response.getWriter().println(
            "<h1>Hello from my first Servlet!</h1>"
        );
    }
}
```

Let's understand every important line.

### Line 1: Package declaration

```java
package com.example;
```

The package identifies the namespace containing our class.

Because the class belongs to `com.example`, its source file should be inside the corresponding package directory:

```text
src/main/java/com/example/
```

### Lines 3–6: Imports

```java
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
```

Each import provides access to a Java type.

- `WebServlet` lets us map a URL to the servlet.
- `HttpServlet` is the base class for our HTTP servlet.
- `HttpServletRequest` represents the incoming HTTP request.
- `HttpServletResponse` represents the outgoing HTTP response.
- `IOException` represents an input/output error that can occur while writing the response.

These types come from the Jakarta Servlet API and the Java standard library.

### Line 8: URL mapping

```java
@WebServlet("/hello")
```

This is one of the most important lines.

It tells Tomcat that requests matching the servlet mapping `/hello` should be handled by `HelloServlet`.

For example, if our application is deployed under `/servlet-demo`, the URL becomes:

```text
http://localhost:8080/servlet-demo/hello
```

Notice that `/hello` is the servlet mapping, not the complete application URL.

### Line 9: Class declaration

```java
public class HelloServlet extends HttpServlet {
```

We create a Java class named `HelloServlet`.

The keyword `extends` means that our class inherits from `HttpServlet`.

This allows the class to use the HTTP servlet framework and override methods such as `doGet()`.

### Line 11: Override annotation

```java
@Override
```

This indicates that the next method overrides a method inherited from the parent class.

Java checks that the method signature matches an inherited method.

### Lines 12–16: The `doGet()` method

```java
protected void doGet(
        HttpServletRequest request,
        HttpServletResponse response
) throws IOException {
```

This method handles HTTP GET requests for the servlet.

Tomcat invokes it when a GET request matches the servlet's URL mapping.

Let's understand the parameters.

**`HttpServletRequest request`**

This object provides access to information sent by the client, such as query parameters and request headers.

Later, we will use it to read data submitted through an HTML form.

**`HttpServletResponse response`**

This object allows the servlet to construct the response sent back to the browser.

**`throws IOException`**

Writing a response can fail because of input/output problems. This declaration allows the method to propagate an `IOException` to the servlet container.

### Line 18: Set the response content type

```java
response.setContentType("text/html;charset=UTF-8");
```

This tells the browser that the response contains HTML and uses UTF-8 character encoding.

The browser can then interpret the returned content as HTML.

### Lines 20–22: Send the response

```java
response.getWriter().println(
    "<h1>Hello from my first Servlet!</h1>"
);
```

`getWriter()` provides a writer for sending text in the HTTP response.

`println()` writes the HTML heading into that response.

The browser receives the response and renders the heading.

### Final closing braces

```java
    }
}
```

The first closing brace ends the `doGet()` method, and the second ends the `HelloServlet` class.

### Deep one-liner for the complete program

> `@WebServlet` maps the URL, `HttpServlet` provides HTTP handling, `doGet()` processes the GET request, and `HttpServletResponse` sends the generated content to the browser.

---

## 9. Understand Routes and URL Mapping

Now let's understand what a route means in this servlet example.

In our code, we wrote:

```java
@WebServlet("/hello")
```

This is a servlet URL mapping.

It tells Tomcat which servlet should handle requests for this path within the web application.

### Suppose our application is named `servlet-demo`

The full URL will be:

```text
http://localhost:8080/servlet-demo/hello
```

Let's break it into parts.

| URL part | Meaning |
|---|---|
| `http://` | HTTP protocol |
| `localhost` | The computer running Tomcat |
| `8080` | The server port in this example |
| `/servlet-demo` | The web application's context path |
| `/hello` | The servlet URL mapping |

### Important distinction

These are two different things:

```java
@WebServlet("/hello")
```

and

```text
/servlet-demo/hello
```

The first is the servlet mapping.

The second is the complete path inside the local server URL, including the application context path.

### How does Tomcat identify the servlet?

When the browser requests:

```text
http://localhost:8080/servlet-demo/hello
```

Tomcat identifies the deployed application using `/servlet-demo` and then matches `/hello` to the servlet mapping.

It invokes the servlet's `doGet()` method for the GET request.

### Request flow

```text
Browser
   |
   | GET /servlet-demo/hello
   v
Tomcat
   |
   | Identify application
   v
servlet-demo
   |
   | Match /hello
   v
HelloServlet
   |
   | Execute doGet()
   v
HTTP Response
   |
   v
Browser
```

**Deep one-liner:** The context path identifies the web application, while the servlet mapping identifies the servlet within that application.

---

## 10. Run the Application Using Maven

Before running the project, ensure that the following are configured:

- JDK 17
- Maven
- Tomcat 10.1
- The servlet API dependency in `pom.xml`
- The `HelloServlet.java` source file

### Step 1: Build the project

Open the terminal in the project root and execute:

```bash
mvn clean package
```

Maven compiles the Java code and packages the web application.

After a successful build, we should have:

```text
target/servlet-demo.war
```

### Step 2: Understand WAR packaging

A WAR file is the deployable archive containing the web application's compiled classes, resources, and relevant configuration.

Tomcat can deploy this archive and run the servlet.

### Step 3: Deploy manually if required

If you are not using Smart Tomcat:

1. Build the WAR file.
2. Copy `target/servlet-demo.war` into Tomcat's `webapps` directory.
3. Start Tomcat.
4. Open the URL in the browser.

For our configuration, the URL will generally be:

```text
http://localhost:8080/servlet-demo/hello
```

If the application is deployed with a different context path, use that path instead.

---

## 11. Run the Application Using Smart Tomcat

For classroom development, Smart Tomcat makes the process more convenient.

It allows IntelliJ IDEA to launch the application using the configured Tomcat installation.

### Step 1: Install Smart Tomcat

1. Open IntelliJ IDEA.
2. Go to **Settings → Plugins**.
3. Search for `Smart Tomcat`.
4. Install the plugin.
5. Restart IntelliJ IDEA if requested.

Plugin page:

https://plugins.jetbrains.com/plugin/9492-smart-tomcat

### Step 2: Create a run configuration

Open **Run → Edit Configurations**.

Add a Smart Tomcat configuration.

Configure the following, as supported by your plugin version:

- Tomcat installation directory.
- Project or deployment directory.
- Application context path.
- Server port, if configurable.

Make sure the deployment directory points to a location containing the compiled servlet classes and the web application resources.

### Step 3: Run the application

Click Run.

Tomcat should start, and the IDE should display its output in the Run console.

Use the configured URL to open the application.

For example:

```text
http://localhost:8080/servlet-demo/hello
```

### What if the plugin uses a different context path?

The URL depends on the configured context path. If the context path is `/`, the application may be accessible directly at the server root. If it is `/servlet-demo`, include that path before `/hello`.

Smart Tomcat configuration fields and deployment behavior can vary by plugin version, so always use the actual URL and configuration shown by your IDE.

---

## 12. Expected Output

Open the servlet URL in your browser.

For example:

```text
http://localhost:8080/servlet-demo/hello
```

You should see:

# Hello from my first Servlet!

This confirms that:

1. Tomcat is running.
2. Our web application has been deployed.
3. The URL mapping is working.
4. Tomcat is invoking our servlet.
5. The servlet is sending an HTTP response.
6. The browser is rendering the returned HTML.

### What did we actually build?

We built a Java web application that returns a response through a servlet.

We have not connected the database or Hibernate yet.

Our current application flow is:

```text
Browser
   |
   | HTTP GET Request
   v
Tomcat
   |
   v
HelloServlet
   |
   | HTTP Response
   v
Browser
```

---

## 13. Common Errors and Solutions

### Error 1: `package jakarta.servlet does not exist`

**Reason:** The servlet API dependency is missing, or Maven has not downloaded the dependency.

**Solution:**

- Verify the `jakarta.servlet-api` dependency.
- Reload the Maven project.
- Run `mvn clean package`.

### Error 2: `404 Not Found`

**Reason:** The application context path or servlet mapping may be incorrect, or the application may not have been deployed.

**Solution:**

- Check that Tomcat has deployed the application.
- Check the URL.
- Verify `@WebServlet("/hello")`.
- Inspect the Tomcat logs.

### Error 3: `Address already in use`

**Reason:** Another application is already using port 8080.

**Solution:** Stop the conflicting process or configure Tomcat to use another available port.

### Error 4: `javax.servlet` imports do not work

**Reason:** Older tutorials may use the Java EE `javax.servlet` namespace.

**Solution:** We are using Tomcat 10.1, so our code must use the `jakarta.servlet` namespace and compatible dependencies.

### Error 5: The project builds but the servlet does not run

**Reason:** The application may have been deployed incorrectly, or the configured deployment directory may not contain the compiled classes.

**Solution:**

- Confirm that `HelloServlet.class` is available in the deployed application's `WEB-INF/classes/com/example/` directory.
- Check the Smart Tomcat deployment configuration.
- Review the server logs.

---

## 14. Practical Assignments

After completing the example, students should try these tasks.

### Beginner

1. Change the response to `Welcome to Java Web Development`.
2. Change the URL mapping from `/hello` to `/welcome`.
3. Create a servlet mapped to `/about`.
4. Return an HTML heading and a paragraph.
5. Create two different servlet classes with different URL mappings.

### Intermediate

6. Create a servlet that displays the current server date and time.
7. Create a servlet that returns a simple HTML list.
8. Create a servlet that returns a different message from another servlet route.
9. Create a frontend form and submit a username to the servlet.
10. Read the submitted username using `request.getParameter()` and display a personalized greeting.

We will explore frontend form submission in the next lesson.

---
