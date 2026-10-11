# Maven Tutorial for Beginners: Creating a WAR File and Deploying to Tomcat

---

## Table of Contents

1. What is Maven?
2. Why do we need Maven?
3. What is a WAR file?
4. What is Apache Tomcat?
5. Understanding the Maven project structure
6. Installing Java, Maven, and Tomcat
7. Creating a Maven web application
8. Understanding `pom.xml`
9. Writing a complete Servlet application
10. Understanding the Maven build lifecycle
11. Creating a WAR file
12. Deploying the WAR file to Tomcat
13. Accessing the application in a browser
14. Updating and redeploying the application
15. Common errors and solutions
16. Maven commands cheat sheet
17. Practice exercises
18. Interview questions

---

# 1. What Is Maven?

**Maven is a build automation and dependency management tool used primarily for Java projects.**

It helps developers manage project dependencies, compile Java code, run tests, package applications, and prepare applications for deployment.

Imagine that you are developing a Java Servlet project.

You need to:

- Compile your Java files.
- Add the Servlet API dependency.
- Organize resources and configuration files.
- Package your application into a WAR file.
- Prepare the application for deployment on Tomcat.

Without Maven, you might need to manage many of these tasks manually.

Maven automates much of this work.

### Real-world example

Suppose your project uses a Servlet API dependency.

Without Maven, you may need to download the required JAR file and configure your IDE or build path manually.

With Maven, you declare the dependency inside `pom.xml`, and Maven resolves it for you.

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.0.0</version>
    <scope>provided</scope>
</dependency>
```

Maven reads this configuration and downloads the dependency into its local repository when required.

**Important:** Maven does not replace Java or Tomcat. Each has a different responsibility.

| Technology | Responsibility |
|---|---|
| JDK | Compiles and runs Java code |
| Maven | Builds the project and manages dependencies |
| Servlet API | Provides classes and interfaces for Servlet development |
| Apache Tomcat | Runs the Servlet web application |
| WAR file | Packages the web application for deployment |

---

# 2. Why Do We Need Maven?

Consider a project containing the following files:

```text
MyApplication/
├── src/
│   └── Main.java
├── libraries/
└── configuration/
```

As the project grows, managing libraries, compilation, testing, and packaging manually becomes difficult.

Maven provides a standard solution.

### 2.1 Dependency management

Maven downloads libraries declared in `pom.xml`.

For example, a Servlet application can declare the Jakarta Servlet API instead of manually downloading its JAR file.

### 2.2 Standard project structure

Maven follows a standard directory structure.

```text
src/main/java       -> Java source files
src/main/resources  -> Application resources
src/main/webapp     -> Web files such as HTML and WEB-INF
src/test/java       -> Test source files
```

This structure makes projects easier to understand and maintain.

### 2.3 Compilation

Maven compiles Java source files into `.class` files.

For example:

```text
HelloServlet.java
       |
       | Maven compiles
       v
HelloServlet.class
```

### 2.4 Packaging

Maven can package a web application into a `.war` file.

```text
Java source code
       |
       v
Maven build
       |
       v
Compiled classes + web resources
       |
       v
hello-maven.war
```

### 2.5 Repeatable builds

Another developer can obtain the project, run the appropriate Maven command, and build the application without manually assembling its dependencies.

---

# 3. What Is a WAR File?

WAR stands for **Web Application Archive**.

A WAR file packages the files required by a Java web application into a single archive.

It can contain:

- Compiled Java classes.
- HTML, CSS, and JavaScript files.
- JSP files.
- Configuration files.
- Application libraries.
- Other web resources.

A WAR file uses a ZIP-compatible archive format.

For example:

```text
hello-maven.war
```

A simplified view of its contents might look like this:

```text
hello-maven.war
│
├── index.html
│
└── WEB-INF/
    ├── classes/
    │   └── com/
    │       └── example/
    │           └── HelloServlet.class
    │
    └── lib/
        └── application-library.jar
```

The exact contents depend on the project and its dependencies.

## JAR vs WAR

| JAR | WAR |
|---|---|
| Java Archive | Web Application Archive |
| Commonly used for Java libraries and applications | Used for Java web applications |
| May run independently when configured with an executable entry point | Typically deployed to a Servlet container |
| Usually contains Java classes and resources | Uses a web application directory structure |

**Remember:** A WAR file is not a server. It is a package that a compatible Servlet container, such as Tomcat, can deploy.

---

# 4. What Is Apache Tomcat?

Apache Tomcat is a Java Servlet container that can run Java web applications.

When you deploy a Servlet-based WAR file, Tomcat loads the application, creates the Servlet environment, and handles incoming HTTP requests.

For example:

```text
Browser
   |
   | HTTP request
   v
Apache Tomcat
   |
   v
Deployed web application
   |
   v
HelloServlet
   |
   v
HTTP response
   |
   v
Browser
```

Tomcat commonly uses port `8080` for HTTP traffic in a default installation.

Its `webapps` directory is a conventional deployment location.

Example:

```text
apache-tomcat/
├── bin/
├── conf/
├── logs/
├── temp/
├── webapps/
│   ├── ROOT/
│   ├── docs/
│   ├── examples/
│   └── hello-maven.war
└── work/
```

When Tomcat's Host configuration has automatic deployment enabled, placing a WAR file in `webapps` normally causes Tomcat to deploy it.

---

# 5. Understanding the Maven Project Structure

We will create a project named `hello-maven`.

The final project structure will be:

```text
hello-maven/
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
        └── webapp/
            └── index.html
```

Let's understand each part.

### `pom.xml`

The Project Object Model file contains project information, dependencies, build configuration, and plugins.

### `src/main/java`

Contains Java source code.

Example:

```text
src/main/java/com/example/HelloServlet.java
```

### `src/main/webapp`

Contains web resources, such as HTML, CSS, JavaScript, JSP files, and the `WEB-INF` directory.

### `target`

Maven creates this directory when building the project.

```text
target/
├── classes/
├── hello-maven/
└── hello-maven.war
```

The exact intermediate files can vary by Maven configuration.

**Important:** You do not need to create `target` manually. Maven generates it.

---

# 6. Installing Java, Maven, and Tomcat

We will use the following compatible setup for this tutorial:

- JDK 17
- Apache Maven 3.9.x
- Apache Tomcat 10.1.x
- Jakarta Servlet API 6.0
- Maven WAR Plugin

Tomcat 10.1 supports Jakarta Servlet 6.0 and requires Java 11 or later. JDK 17 is a suitable choice for this example.

Use official downloads:

- Java: https://jdk.java.net/17/
- Maven: https://maven.apache.org/download.cgi
- Tomcat: https://tomcat.apache.org/download-10.cgi

Follow the installation instructions for your operating system.

## 6.1 Verify Java

Open a terminal or Windows Command Prompt.

```bash
java -version
```

Then check the Java compiler:

```bash
javac -version
```

You should see a Java 17 version.

If the commands are not recognized, install the JDK and configure `PATH` and, where appropriate, `JAVA_HOME`.

## 6.2 Verify Maven

```bash
mvn -version
```

The output should display the Maven version and the Java version used by Maven.

If `mvn` is not recognized, extract Maven and add its `bin` directory to `PATH`.

For example, a Windows installation might use:

```text
C:\apache-maven\bin
```

The actual directory depends on where you extracted Maven.

## 6.3 Verify Tomcat

Extract the downloaded Tomcat archive.

For example:

```text
C:\apache-tomcat-10.1\
```

The directory should contain folders such as:

```text
bin/
conf/
lib/
logs/
webapps/
```

We will use this directory later when deploying the WAR file.

---

# 7. Creating a Maven Web Application

Create a project directory:

```text
hello-maven
```

Inside it, create this structure:

```text
hello-maven/
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/
        │       └── example/
        │           └── HelloServlet.java
        └── webapp/
            └── index.html
```

You can create the directories manually or use your IDE.

## Step 1: Create `pom.xml`

Create a file named `pom.xml` directly inside the `hello-maven` directory.

Do not create a file named `pom.xml.txt`.

## Step 2: Create the Servlet

Create the Java package directory:

```text
src/main/java/com/example/
```

Inside it, create:

```text
HelloServlet.java
```

## Step 3: Create the web page

Inside `src/main/webapp`, create:

```text
index.html
```

Next, we will add the complete code for each file.

---

# 8. Understanding `pom.xml`

The `pom.xml` file is the heart of a Maven project.

Here is the complete configuration for our application.

## Complete `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>hello-maven</artifactId>
    <version>1.0.0</version>
    <packaging>war</packaging>

    <name>Hello Maven Web Application</name>

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
        <finalName>hello-maven</finalName>

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
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>

        </plugins>
    </build>

</project>
```

## Understanding the important elements

### 8.1 `<modelVersion>`

```xml
<modelVersion>4.0.0</modelVersion>
```

Specifies the POM model version understood by Maven.

It is not the version of your application.

### 8.2 `<groupId>`

```xml
<groupId>com.example</groupId>
```

Identifies the organization or group associated with the project.

In real projects, this commonly uses a reverse-domain naming convention.

### 8.3 `<artifactId>`

```xml
<artifactId>hello-maven</artifactId>
```

Specifies the project's artifact name.

It helps identify the project in Maven repositories.

### 8.4 `<version>`

```xml
<version>1.0.0</version>
```

Specifies the application version.

### 8.5 `<packaging>`

```xml
<packaging>war</packaging>
```

**This is one of the most important lines in this tutorial.**

It tells Maven to package the project as a WAR rather than the default JAR.

Without this setting, Maven normally uses JAR packaging.

### 8.6 `<properties>`

```xml
<properties>
    <maven.compiler.release>17</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

The compiler release property configures Java 17 language and API compatibility for compilation.

The encoding property sets UTF-8 as the project's source encoding.

### 8.7 `<dependencies>`

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.0.0</version>
    <scope>provided</scope>
</dependency>
```

This declares the Jakarta Servlet API dependency.

It provides the Servlet interfaces and annotations needed to compile our Java code.

### 8.8 What does `provided` mean?

```xml
<scope>provided</scope>
```

It means the dependency is required to compile the application but is expected to be supplied by the runtime environment.

Tomcat provides the Servlet API when it runs the application.

Therefore, we generally do not bundle the Servlet API JAR inside our WAR.

Do not confuse this with a normal application library, which may need to be packaged inside `WEB-INF/lib`.

### 8.9 `<build>`

The build section configures how Maven builds the application.

```xml
<build>
    <finalName>hello-maven</finalName>
</build>
```

This specifies the final artifact name, excluding the extension.

The resulting WAR will be:

```text
hello-maven.war
```

### 8.10 Maven plugins

The compiler plugin compiles Java code.

The WAR plugin packages the web application.

```xml
<artifactId>maven-compiler-plugin</artifactId>
```

```xml
<artifactId>maven-war-plugin</artifactId>
```

These plugins help Maven perform the build.

### 8.11 Why is `failOnMissingWebXml` set to `false`?

```xml
<failOnMissingWebXml>false</failOnMissingWebXml>
```

Older Servlet projects often relied on a `web.xml` deployment descriptor.

Modern Servlet applications can use annotations such as `@WebServlet`.

This configuration permits WAR packaging when the project does not contain a `web.xml` file.

It does not disable the use of `web.xml`; you can still add one when required.

---

# 9. Writing the Complete Servlet Application

We will create a simple Servlet that responds with a welcome message.

## 9.1 Complete `HelloServlet.java`

File location:

```text
src/main/java/com/example/HelloServlet.java
```

Code:

```java
package com.example;

import java.io.IOException;
import java.io.PrintWriter;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();

        out.println("<!DOCTYPE html>");
        out.println("<html>");
        out.println("<head>");
        out.println("<title>Maven WAR Tutorial</title>");
        out.println("</head>");
        out.println("<body>");

        out.println("<h1>Hello from Maven and Tomcat!</h1>");
        out.println("<p>Your WAR file is running successfully.</p>");

        out.println("</body>");
        out.println("</html>");
    }
}
```

## Understanding the code

### Package declaration

```java
package com.example;
```

The package corresponds to the directory structure under `src/main/java`.

### Servlet imports

```java
import jakarta.servlet.http.HttpServlet;
```

`HttpServlet` is the base class used to create HTTP Servlets.

We also import the request and response types to process HTTP communication.

### Servlet annotation

```java
@WebServlet("/hello")
```

This maps the Servlet to the `/hello` URL within the web application.

### Extending `HttpServlet`

```java
public class HelloServlet extends HttpServlet
```

This creates an HTTP Servlet by extending `HttpServlet`.

### Handling GET requests

```java
protected void doGet(
        HttpServletRequest request,
        HttpServletResponse response)
```

Tomcat invokes `doGet()` when the Servlet receives an appropriate HTTP GET request.

### Setting the content type

```java
response.setContentType("text/html;charset=UTF-8");
```

This tells the browser that the response contains HTML encoded using UTF-8.

### Writing the response

```java
PrintWriter out = response.getWriter();
```

The writer sends text into the HTTP response body.

The browser renders the HTML returned by the Servlet.

## 9.2 Complete `index.html`

File location:

```text
src/main/webapp/index.html
```

Code:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Maven WAR Tutorial</title>
</head>
<body>

    <h1>Welcome to Maven</h1>

    <p>This page is part of our Maven web application.</p>

    <a href="hello">Open Hello Servlet</a>

</body>
</html>
```

The `index.html` file is a static page. Tomcat can serve it directly without executing a Servlet.

The link points to `hello`, which resolves relative to the current web application URL.

---

# 10. Understanding the Maven Build Lifecycle

Maven provides a standard build lifecycle.

The lifecycle consists of phases that perform different tasks.

Some important phases are:

| Phase | Purpose |
|---|---|
| `validate` | Checks whether the project information is valid |
| `compile` | Compiles main Java source code |
| `test` | Runs tests |
| `package` | Packages the compiled application |
| `verify` | Performs additional checks and verification |
| `install` | Installs the artifact into the local Maven repository |
| `deploy` | Publishes an artifact to a configured remote Maven repository |

**Important:** Maven's `deploy` lifecycle phase does not automatically mean deploying a WAR into Tomcat's `webapps` directory.

That is a separate application deployment step.

## How the build works

When you run:

```bash
mvn package
```

Maven executes the necessary preceding phases in order.

A simplified flow is:

```text
validate
   |
   v
compile
   |
   v
test
   |
   v
package
   |
   v
hello-maven.war
```

The exact plugins and tasks executed depend on the project configuration.

---

# 11. Creating the WAR File

Now we will build the project.

## Step 1: Open the project directory

Open a terminal in the directory containing `pom.xml`.

Example:

```text
hello-maven/
```

Run:

```bash
mvn clean package
```

## Step 2: Understand the command

### `mvn`

Runs Maven.

### `clean`

Removes build output from a previous build, normally the `target` directory.

### `package`

Runs the build lifecycle through the packaging phase.

For this project, Maven compiles the Servlet and packages the web application as a WAR.

## Step 3: Check the output

After a successful build, the project should contain:

```text
hello-maven/
├── pom.xml
├── src/
└── target/
    ├── classes/
    │   └── com/
    │       └── example/
    │           └── HelloServlet.class
    │
    ├── hello-maven/
    │   ├── index.html
    │   └── WEB-INF/
    │       └── classes/
    │           └── com/
    │               └── example/
    │                   └── HelloServlet.class
    │
    └── hello-maven.war
```

Some intermediate directories may differ by plugin version and configuration.

The important output is:

```text
target/hello-maven.war
```

**Congratulations! You have created your first WAR file using Maven.**

## What happened internally?

Maven performed these operations:

1. Read `pom.xml`.
2. Resolved the required dependencies and plugins.
3. Compiled `HelloServlet.java`.
4. Copied web resources such as `index.html`.
5. Created the required WAR directory structure.
6. Packaged the application into `hello-maven.war`.

You do not need to manually create the WAR archive.

---

# 12. Deploying the WAR File to Tomcat

We now have the WAR file, but it is not yet running in Tomcat.

We need to deploy it.

## Step 1: Locate the WAR file

Find the generated file:

```text
hello-maven/target/hello-maven.war
```

## Step 2: Locate Tomcat's `webapps` directory

For example, if Tomcat is extracted to:

```text
C:\apache-tomcat-10.1\
```

Its deployment directory will be:

```text
C:\apache-tomcat-10.1\webapps\
```

On Linux, a manually extracted Tomcat installation might be located at:

```text
/opt/tomcat/webapps/
```

Your actual installation directory may be different.

## Step 3: Copy the WAR file

Copy:

```text
hello-maven.war
```

Into:

```text
apache-tomcat-10.1/webapps/
```

The resulting structure should look like:

```text
apache-tomcat-10.1/
└── webapps/
    ├── ROOT/
    ├── docs/
    ├── examples/
    └── hello-maven.war
```

## Step 4: Start Tomcat

### Windows

Open Command Prompt and execute:

```bat
cd C:\apache-tomcat-10.1\bin
startup.bat
```

Use your actual Tomcat installation path.

If you installed Tomcat as a Windows service, you can start that service instead.

### Linux

For a manually extracted Tomcat installation:

```bash
cd /opt/tomcat/bin
./startup.sh
```

Use the correct installation path and ensure that the script has the necessary execution permissions.

## Step 5: Check deployment

If automatic deployment is enabled, Tomcat will typically detect the WAR and deploy it.

It may create an expanded directory:

```text
webapps/
├── hello-maven.war
└── hello-maven/
    ├── index.html
    └── WEB-INF/
        └── classes/
            └── com/
                └── example/
                    └── HelloServlet.class
```

Tomcat manages this expanded directory. Do not manually edit its deployed contents as your primary development workflow.

If Tomcat was already running when you copied the WAR, wait for deployment to finish and check the logs if needed.

---

# 13. Accessing the Application in a Browser

With Tomcat running on the default HTTP port `8080`, open the following URL.

## Home page

```text
http://localhost:8080/hello-maven/
```

Expected page:

**Welcome to Maven**

This is the static HTML page from `index.html`.

## Servlet page

```text
http://localhost:8080/hello-maven/hello
```

Expected output:

**Hello from Maven and Tomcat!**

Your WAR file has now been deployed and is serving HTTP requests.

## Understanding the URL

```text
http://localhost:8080/hello-maven/hello
```

| Part | Meaning |
|---|---|
| `http` | Protocol |
| `localhost` | Your own computer |
| `8080` | Tomcat's configured HTTP port |
| `/hello-maven` | Web application context path |
| `/hello` | Servlet mapping |

The context path normally comes from the WAR filename when deploying a WAR with Tomcat's standard automatic deployment configuration.

For example:

```text
hello-maven.war
```

normally gives:

```text
/hello-maven
```

The `@WebServlet("/hello")` annotation defines the Servlet path within that application.

Together, these form the complete URL.

---

# 14. Updating and Redeploying the Application

Suppose you change the message in `HelloServlet.java`.

For example:

```java
out.println("<h1>Welcome to the updated application!</h1>");
```

You need to rebuild and redeploy the WAR.

## Step 1: Rebuild

From the project directory, run:

```bash
mvn clean package
```

This creates a new file:

```text
target/hello-maven.war
```

## Step 2: Replace the deployed WAR

Copy the new WAR into Tomcat's `webapps` directory, replacing the old version.

When necessary, follow your environment's safe redeployment procedure. For a simple local tutorial, you can stop Tomcat, replace the WAR, and start it again.

## Step 3: Refresh the browser

Visit:

```text
http://localhost:8080/hello-maven/hello
```

You should see the updated message.

**Best practice:** Keep your source code in your Maven project. Treat the generated WAR and Tomcat's expanded deployment directory as build and runtime artifacts, not as the main source files.

---

# 15. Common Errors and Solutions

## Error 1: `mvn is not recognized`

Example:

```text
'mvn' is not recognized as an internal or external command
```

**Cause:** Maven is not installed correctly or its `bin` directory is missing from `PATH`.

**Solution:**

1. Download and extract Maven.
2. Add the Maven `bin` directory to `PATH`.
3. Open a new terminal.
4. Run:

```bash
mvn -version
```

## Error 2: `JAVA_HOME` is not set correctly

**Cause:** Maven cannot locate a suitable JDK.

**Solution:** Install the JDK and configure `JAVA_HOME` to point to its installation directory.

Then check:

```bash
java -version
mvn -version
```

## Error 3: `package jakarta.servlet does not exist`

**Cause:** The Servlet API dependency may be missing, Maven dependencies have not resolved, or the code is using the wrong Servlet namespace.

**Solution:** Check that the dependency in `pom.xml` is correct:

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.0.0</version>
    <scope>provided</scope>
</dependency>
```

Then rebuild:

```bash
mvn clean package
```

## Error 4: `ClassNotFoundException` or `NoClassDefFoundError`

**Cause:** A required class cannot be found at runtime.

Possible reasons include:

- A required application library is missing from the WAR.
- The WAR was built incorrectly.
- The Servlet API and Tomcat versions are incompatible.
- The application uses the wrong `javax.servlet` or `jakarta.servlet` namespace.

**Solution:** Inspect the build output, dependencies, and Tomcat logs.

This tutorial uses `jakarta.servlet` with Tomcat 10.1.

Older Tomcat versions, such as Tomcat 9, use the `javax.servlet` namespace and require compatible dependencies and code.

## Error 5: HTTP 404 Not Found

**Cause:** Tomcat cannot find the requested application or Servlet mapping.

Check the following:

1. Is Tomcat running?
2. Is the WAR in the correct `webapps` directory?
3. Did Tomcat successfully deploy it?
4. Are you using the correct context path?
5. Does the Servlet annotation contain `@WebServlet("/hello")`?
6. Is the URL spelled correctly?

Try:

```text
http://localhost:8080/hello-maven/
```

Then:

```text
http://localhost:8080/hello-maven/hello
```

## Error 6: HTTP 500 Internal Server Error

**Cause:** The server encountered an error while processing the request.

Check Tomcat's logs, especially the `logs` directory.

Look for exceptions or stack traces corresponding to the time of the request.

## Error 7: WAR file is not generated

**Cause:** The build may have failed, or the project may not be configured for WAR packaging.

Confirm that `pom.xml` contains:

```xml
<packaging>war</packaging>
```

Then run:

```bash
mvn clean package
```

Read the build output and resolve any errors before trying again.

## Error 8: Port 8080 is already in use

**Cause:** Another process is already listening on the configured port.

**Solution:** Stop the conflicting process or configure Tomcat to use another available port.

The configured HTTP connector is normally defined in:

```text
conf/server.xml
```

Do not change the port without checking which service is using it.

---

# 16. Maven Commands Cheat Sheet

| Command | Purpose |
|---|---|
| `mvn -version` | Displays Maven and Java information |
| `mvn validate` | Validates the project |
| `mvn compile` | Compiles main source code |
| `mvn test` | Runs tests |
| `mvn package` | Packages the project |
| `mvn clean` | Removes build output |
| `mvn clean package` | Cleans and rebuilds the project |
| `mvn verify` | Runs the lifecycle through verification |
| `mvn install` | Installs the artifact into the local Maven repository |
| `mvn dependency:tree` | Displays the dependency hierarchy |

For this tutorial, the most important command is:

```bash
mvn clean package
```

It generates the WAR file in the `target` directory.

---

# 17. Practice Exercises

Try these exercises after successfully deploying the application.

### Exercise 1: Change the Servlet message

Update the message returned by `HelloServlet`.

Rebuild the project and redeploy it.

### Exercise 2: Add a second Servlet

Create another Servlet:

```java
@WebServlet("/about")
```

Return an HTML response describing the application.

Test:

```text
http://localhost:8080/hello-maven/about
```

### Exercise 3: Add a CSS file

Create:

```text
src/main/webapp/style.css
```

Link it from `index.html` and verify that the WAR contains the CSS file.

### Exercise 4: Change the WAR filename

Change:

```xml
<finalName>hello-maven</finalName>
```

to:

```xml
<finalName>student-portal</finalName>
```

Rebuild the project.

What is the new WAR filename? What context path will Tomcat normally use when that WAR is deployed?

### Exercise 5: Inspect the WAR

Open the WAR file using an archive viewer.

Identify:

- The HTML file.
- The compiled Servlet class.
- The `WEB-INF` directory.
- The `WEB-INF/lib` directory, if present.

### Exercise 6: Understand dependency scope

Temporarily research the difference between `provided` and `compile` dependency scopes.

Why is the Servlet API normally declared with `provided` in a Tomcat application?

---

# 18. Interview Questions

**Q1. What is Maven?**

Maven is a build automation and dependency management tool commonly used in Java projects.

**Q2. What is `pom.xml`?**

It is the Maven Project Object Model file that describes project metadata, dependencies, and build configuration.

**Q3. What is a WAR file?**

A WAR file packages a Java web application for deployment to a compatible Servlet container.

**Q4. What is the purpose of `<packaging>war</packaging>`?**

It tells Maven to produce a WAR artifact instead of the default JAR artifact.

**Q5. What is the `target` directory?**

It contains generated build output, including compiled classes and packaged artifacts.

**Q6. What does `mvn clean package` do?**

It removes previous build output and executes the Maven build lifecycle through the packaging phase.

**Q7. What is the purpose of Tomcat's `webapps` directory?**

It is a conventional directory for deploying web applications in a standard Tomcat installation.

**Q8. Why do we use `provided` for the Servlet API dependency?**

Because the Servlet API is supplied by the Servlet container at runtime.

**Q9. Does `mvn package` automatically deploy the application to Tomcat?**

No. It creates the artifact. Deployment is a separate step unless an additional deployment workflow has been configured.

**Q10. Does Maven's `deploy` phase mean copying the WAR into Tomcat's `webapps` directory?**

No. Maven's standard `deploy` phase publishes artifacts to a configured remote Maven repository. Tomcat deployment is a separate operation.

**Q11. Why does the URL contain `/hello-maven/hello`?**

The first path segment identifies the web application's context path, and the second is the Servlet mapping.

**Q12. What is the difference between `javax.servlet` and `jakarta.servlet`?**

They are different package namespaces used by different generations of Servlet APIs. Tomcat 10.1 uses `jakarta.servlet`, whereas Tomcat 9 uses `javax.servlet`.

---

# Final Summary

You have learned how to create a Maven web application, declare dependencies, compile Java Servlet code, generate a WAR file, deploy it to Tomcat, and access it through a browser.

The complete workflow is:

```text
Create Maven project
        |
        v
Configure pom.xml
        |
        v
Write Servlet and HTML
        |
        v
Run mvn clean package
        |
        v
Generate target/hello-maven.war
        |
        v
Copy WAR into Tomcat/webapps
        |
        v
Tomcat deploys the application
        |
        v
Open application in browser
```

Remember these three essential concepts:

1. **Maven** builds and packages the project.
2. **WAR** is the packaged web application.
3. **Tomcat** deploys and runs the application.

Once this workflow is clear, you can apply the same concepts to larger Servlet, JSP, Hibernate, and other Java web projects.

---

## Official References

- Maven: https://maven.apache.org/
- Maven POM Reference: https://maven.apache.org/pom.html
- Maven Build Lifecycle: https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
- Maven WAR Plugin: https://maven.apache.org/plugins/maven-war-plugin/
- Apache Tomcat 10.1 Documentation: https://tomcat.apache.org/tomcat-10.1-doc/
- Jakarta Servlet Specification: https://jakarta.ee/specifications/servlet/6.0/
