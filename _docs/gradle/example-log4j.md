---
title: 4. Example - log4j
category: 02. Gradle
order: 4
---
<h2>Contents</h2>
* toc
{:toc}

In this section, we will see how to add a dependency to our Gradle project and use it. As an example, we will use a Java logging library: <a target="_blank" rel="noopener noreferrer" href="https://logging.apache.org/log4j/2.x/">Log4j</a>.
## Logging
When you write software, it is useful to add logs. Logs help you, for example, to record the events that happen in your application, to understand its patterns of activity, and to find the source of a problem when something goes wrong.  

Log4j is a library that improves on Java's built-in logging mechanism. To use Log4j in a Gradle project, we follow the <a target="_blank" rel="noopener noreferrer" href="https://logging.apache.org/log4j/2.x/manual/installation.html">documentation</a> and add its dependencies. Our build file should look like this:
{% highlight kotlin %}
plugins {
    id("application")
}

group = "org.example"
version = "1.0-SNAPSHOT"

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.apache.logging.log4j:log4j-api:2.26.1")
    implementation("org.apache.logging.log4j:log4j-core:2.26.1")
    testImplementation(platform("org.junit:junit-bom:6.1.3"))
    testImplementation("org.junit.jupiter:junit-jupiter")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

application {
    mainClass.set("org.example.Main")
}

tasks.test {
    useJUnitPlatform()
}
{% endhighlight %}

Then, we can start using Log4j in our application:
{% highlight java %}
package org.example;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class Main {
    private static final Logger logger = LogManager.getLogger("Main");
    public static int add(int a, int b) {
        return Math.addExact(a, b);
    }

    public static void main(String[] args) {
        // int a = Integer.parseInt(args[0]);
        // int b = Integer.parseInt(args[1]);
        // int result = Main.add(a, b);
        logger.info("Hello World from log4j");
    }
}
{% endhighlight %}
However, if you run the application, nothing is printed in the console. This is because we haven't configured Log4j yet: without a configuration, Log4j uses a default one that prints only the most severe messages (errors), so our info message is discarded. To configure it, create a file named `log4j2.xml` in `src/main/resources` (remember: this is the folder for the files that the application needs at runtime), with this content:
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
        </Console>
        <File name="File" fileName="./logs/log.log" immediateFlush="false" append="false">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </File>
    </Appenders>
    <Loggers>
        <Root level="debug">
            <AppenderRef ref="Console" />
            <AppenderRef ref="File"/>
        </Root>
    </Loggers>
</Configuration>
{% endhighlight %}
This configuration tells Log4j to write the logs both to the console and to the file `./logs/log.log`. Run the application again and check it out! The `status` attribute, instead, refers to the internal messages of Log4j itself: with `WARN`, Log4j tells us only if something is wrong with its configuration. You can find more information about the configuration file <a target="_blank" rel="noopener noreferrer" href="https://logging.apache.org/log4j/2.x/manual/configuration.html">here</a>.  

## Log level
Logs can be classified by importance, using the log level. The higher the level, the more important the message, and the more attention it requires: the highest levels are used for errors and fatal errors. These are the levels offered by Log4j, from the most to the least important:
1. Fatal: the application is in a state where one of its crucial functionalities has stopped working.
2. Error: one or more functionalities don't work properly.
3. Warn: something unexpected happened, but the application is not affected.
4. Info: something normal happened (for example, the application entered a certain state).
5. Debug: information useful for debugging.
6. Trace: the finest-grained information, used when you need full visibility of what is happening inside your application (for example, to follow its control flow step by step).

You can use log levels in the following way:

{% highlight java %}
package org.example;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class Main {
    private static final Logger logger = LogManager.getLogger("Main");
    public static int add(int a, int b) {
        return Math.addExact(a, b);
    }

    public static void main(String[] args) {
        // int a = Integer.parseInt(args[0]);
        // int b = Integer.parseInt(args[1]);
        // int result = Main.add(a, b);
        logger.trace("Hello World from log4j");
        logger.debug("Hello World from log4j");
        logger.info("Hello World from log4j");
        logger.warn("Hello World from log4j");
        logger.error("Hello World from log4j");
        logger.fatal("Hello World from log4j");
    }
}
{% endhighlight %}

{% highlight bash %}
~/IdeaProjects/Gradle-GettingStarted ./gradlew run

> Task :run
15:45:27.762 [main] DEBUG Main - Hello World from log4j
15:45:27.763 [main] INFO  Main - Hello World from log4j
15:45:27.763 [main] WARN  Main - Hello World from log4j
15:45:27.763 [main] ERROR Main - Hello World from log4j
15:45:27.764 [main] FATAL Main - Hello World from log4j

BUILD SUCCESSFUL in 1s
3 actionable tasks: 2 executed, 1 up-to-date
~/IdeaProjects/Gradle-GettingStarted 
{% endhighlight %}
Notice that the trace message is missing. In our configuration file, the root logger has `level="debug"`: this means that only messages with level debug or higher (i.e., more important) are logged, and the others are discarded. Try to change it to `"trace"` (all the messages are logged) or to `"warn"` (only warn, error, and fatal messages are logged), and run the application again.  

This is very useful in practice: you can fill your code with debug and trace messages during development, and then raise the level in production to keep only the important ones, without touching the code.

## Exercises
1. Define a class `Student` that has a name, a surname, a matriculation number, and a list of passed courses. Then, write a test that creates a `Student` and adds some courses to their list. Finally, log the created `Student` in JSON format, using Log4j and the <a target="_blank" rel="noopener noreferrer" href="https://github.com/google/gson">Gson</a> library.
<div class="lesson-nav">
    <div>
        Previous: <a href="{{ site.baseurl }}/gradle/testing-java-applications">Gradle - Testing Java Applications</a>
    </div>
</div>