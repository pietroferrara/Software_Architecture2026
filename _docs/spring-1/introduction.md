---
title: 1. Introduction
category: 03. Java Spring (Part I)
order: 1
exclude: false
excluded_in_search: false
---
![Spring logo]({{ site.baseurl }}/images/spring_logo.png)
<h2>Contents</h2>
* toc
{:toc}
<iframe width="100%" height="315" src="https://www.youtube.com/embed/wAAJ8eS-CzA?si=oCBZVTMpJ9D7p-ZM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
## The Spring framework
<a target="_blank" rel="noopener noreferrer" href="https://spring.io/">Spring</a> is a free and open-source framework for building Java applications, mainly web applications and REST APIs. It takes care of the "plumbing" that almost every application needs: it creates your objects and connects them together, receives HTTP requests and routes them to your code, talks to databases, handles security, and so on. This way, you can focus on the logic of your application. Spring is also an ecosystem: around its core there are many projects (Spring Boot, Spring Data, Spring Security, ...) that you add depending on what you need.  

Before going further, let's briefly define what a framework is.

### What is a framework
Like a library, a framework is a bunch of code packed together. The main difference is that it offers a runtime execution environment. The idea of frameworks is that a developer specifies some components, and under the hood, the framework drives the execution. This concept is called Inversion of Control (IoC): the execution is not managed directly by developers but by the framework itself.  
Usually, a Java framework like Spring consists of a runtime environment plus a library.  

Do you remember JUnit? JUnit is a framework: you specify test methods, but **not** how to run them. You don't have to define a main function that calls the test methods. The framework's responsible for managing how to run it (IoC). Let's recall the simple test that we wrote in one of the previous lessons:  

{% highlight java %}
import org.example.Main;
import org.junit.jupiter.api.Test;

@Test 
public void testAdd() {
    if (Main.add(12, 34) != 46) {
        throw new ArithmeticException("Wrong!");
    }
}
{% endhighlight %}
The JUnit framework understands that the testAdd method is a test because of the annotation @Test. Annotations are one of the ways that permit an interaction with a framework.

### Spring Boot and its dependencies
In our examples, we use **Spring Boot** (version 4). Spring Boot sits on top of the Spring framework and simplifies the creation of applications: it comes with sensible default configurations, and it embeds a web server on which our application is deployed automatically. Spring Boot is not a dependency you pick: it is the base of every project. On top of it, we add:
1. **Spring Web**: adds the Model-View-Controller (MVC) capabilities to Spring, allowing us to build web applications and REST APIs. It also brings the embedded <a target="_blank" rel="noopener noreferrer" href="https://tomcat.apache.org/">Tomcat</a> server.
2. **Spring Boot DevTools**: a development-only helper that, among other things, automatically restarts the application when the code changes.

## Initialize a Gradle Spring project
Let's see now how to initialize a Java Spring Application. There are different ways to do that.
### Using Spring Initializr
<a target="_blank" rel="noopener noreferrer" href="https://start.spring.io">Spring Initializr</a> is an online tool that helps you create a Spring Project. Fill out the form (choose **Gradle - Kotlin** as project type and **Java** as language), add the previously mentioned dependencies (Spring Web, Spring Boot DevTools), and click the "Generate" button. This action will download a .zip file containing your empty Gradle project with Spring dependencies. After unzipping the .zip, you can open the project with your desired IDE and start working on it.  
![Spring intro]({{ site.baseurl }}/images/spring_intro_3.png)

Most IDEs offer the same wizard directly in their "New Project" view: for example, <a target="_blank" rel="noopener noreferrer" href="https://marketplace.visualstudio.com/items?itemName=vmware.vscode-boot-dev-pack">VS Code with the Spring Boot Extension Pack</a> (command *Spring Initializr: Create a Gradle Project...*) or IntelliJ IDEA (Spring support requires a subscription, which is free for students). Under the hood, they all call start.spring.io.
### By Hand
You can also initialize a Spring Web application by configuring plugins and dependencies in your build.gradle.kts:
{% highlight kotlin %}
plugins {
    java
    id("org.springframework.boot") version "4.1.2"
    id("io.spring.dependency-management") version "1.1.7"
}

group = "com.example"
version = "0.0.1-SNAPSHOT"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-webmvc")
    developmentOnly("org.springframework.boot:spring-boot-devtools")
    testImplementation("org.springframework.boot:spring-boot-starter-webmvc-test")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

tasks.withType<Test> {
    useJUnitPlatform()
}
{% endhighlight %}
The `toolchain` block tells Gradle to compile and run the project with JDK 17. If Gradle can't find a JDK 17 on your machine, the build fails with *"Cannot find a Java installation on your machine matching: {languageVersion=17...}"*: either install it, or let Gradle download it automatically by adding this plugin at the top of your settings.gradle.kts:
{% highlight kotlin %}
plugins {
    id("org.gradle.toolchains.foojay-resolver-convention") version "1.0.0"
}
{% endhighlight %}
## Getting Started
If you choose to initialize the project directly by editing the build.gradle.kts file, you need to add this class in the file **src/main/java/com/example/spring/SpringGettingStartedApplication.java**:
{% highlight java %}
package com.example.spring;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringGettingStartedApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringGettingStartedApplication.class, args);
    }
}
{% endhighlight %}
Create also a class Hello.java in the folder **src/main/java/com/example/spring/controller/** and insert this content:
{% highlight java %}
package com.example.spring.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
@RestController
public class Hello {

    @GetMapping("/")
    public String index() {
        return "Greetings from Spring Boot!";
    }
}
{% endhighlight %}
Now launch:
{% highlight bash %}
~/SpringGettingStarted ./gradlew bootRun

> Task :bootRun

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v4.1.2)

2026-10-28T19:47:40.768+01:00  INFO 98146 --- [SpringGettingStarted] [  restartedMain] c.e.s.SpringGettingStartedApplication    : Starting SpringGettingStartedApplication using Java 17.0.21 with PID 98146 (/home/student/SpringGettingStarted/build/classes/java/main started by student in /home/student/SpringGettingStarted)
2026-10-28T19:47:40.775+01:00  INFO 98146 --- [SpringGettingStarted] [  restartedMain] c.e.s.SpringGettingStartedApplication    : No active profile set, falling back to 1 default profile: "default"
2026-10-28T19:47:40.833+01:00  INFO 98146 --- [SpringGettingStarted] [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2026-10-28T19:47:40.833+01:00  INFO 98146 --- [SpringGettingStarted] [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
2026-10-28T19:47:41.624+01:00  INFO 98146 --- [SpringGettingStarted] [  restartedMain] o.s.boot.tomcat.TomcatWebServer          : Tomcat initialized with port 8080 (http)
2026-10-28T19:47:41.639+01:00  INFO 98146 --- [SpringGettingStarted] [  restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2026-10-28T19:47:41.639+01:00  INFO 98146 --- [SpringGettingStarted] [  restartedMain] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/11.0.27]
2026-10-28T19:47:41.679+01:00  INFO 98146 --- [SpringGettingStarted] [  restartedMain] b.w.c.s.WebApplicationContextInitializer : Root WebApplicationContext: initialization completed in 844 ms
2026-10-28T19:47:42.010+01:00  INFO 98146 --- [SpringGettingStarted] [  restartedMain] o.s.boot.tomcat.TomcatWebServer          : Tomcat started on port 8080 (http) with context path '/'
2026-10-28T19:47:42.020+01:00  INFO 98146 --- [SpringGettingStarted] [  restartedMain] c.e.s.SpringGettingStartedApplication    : Started SpringGettingStartedApplication in 1.68 seconds (process running for 2.106)
<==========---> 80% EXECUTING [11s]
> :bootRun

{% endhighlight %}
This command calls the task bootRun, coming from the Spring Gradle plugin, that launches the Spring application. As you can see from the logs, a <a target="_blank" rel="noopener noreferrer" href="https://tomcat.apache.org/">Tomcat</a> server is initialized on port 8080. The framework automatically does this: it isn't necessary to download and configure a Tomcat server! This is transparent from the developer's perspective. If you go on <a target="_blank" rel="noopener noreferrer" href="http://localhost:8080">localhost:8080</a>, you can see the server in action:
![Spring intro]({{ site.baseurl }}/images/spring_intro_4.png)
<div class="lesson-nav">
    <div>
        Next: <a href="{{ site.baseurl }}/spring-1/the-spring-framework">Java Spring - The Spring framework</a>  
    </div>
</div>
