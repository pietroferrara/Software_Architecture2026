---
title: 1. Introduction 
category: 02. Gradle
order: 1
---
![Gradle logo]({{ site.baseurl }}/images/gradle_logo.png)
<h2>Contents</h2>
* toc
{:toc}
<iframe width="100%" height="315" src="https://www.youtube.com/embed/JCCOMnGtjrY?si=63LBNhAWYYIS6fsJ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
## What is Gradle
Gradle is an open-source build automation tool, that is, a tool that allows developers to automate tasks such as compiling code, running tests, generating documentation, and deploying applications.
It is one of the most popular build systems for the Java Virtual Machine (JVM), and the default one for Android.
What makes Gradle special is its own high-level, declarative build language, which makes it easy to write build logic. In this course, we will focus only on using Gradle to build Java applications.

## Installation
You can install Gradle in many different ways:
1. Installing <a target="_blank" rel="noopener noreferrer" href="https://www.jetbrains.com/idea/">IntelliJ IDEA</a>: Gradle is included by default (**Recommended**).
2. Installing the <a target="_blank" rel="noopener noreferrer" href="https://code.visualstudio.com/docs/java/java-build">Java Extension Pack</a> for <a target="_blank" rel="noopener noreferrer" href="https://code.visualstudio.com/">Visual Studio Code</a>.
3. <a target="_blank" rel="noopener noreferrer" href="https://gradle.org/install/">Manual installation</a> (just follow the documentation).  

In this course, we will use Gradle with IntelliJ IDEA. Whatever you choose, make sure to use JDK 17 or newer: this is the minimum version required by recent versions of Gradle and JUnit (the testing library that we will use later).

## Gradle in Action
To see Gradle in action, create a new project with IntelliJ IDEA, choose Gradle as the build system, and set the other parameters as follows:
![Gradle intro]({{ site.baseurl }}/images/gradle_intro_1.png)
Now open `build.gradle.kts` and replace its content with the following snippet (we will explain it in the next sections):
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
Then, click on the elephant icon highlighted by the arrow in the image below (if you don't see it, just reopen the project). This syncs IntelliJ with the changes we made to the build file. (The screenshot was taken with an older version of the build file, so the dependencies you see in it are slightly different from yours.)
![Gradle intro]({{ site.baseurl }}/images/gradle_intro_2.png)
Now open the Gradle view (the elephant in the sidebar on the right). Here you find the list of all the Gradle tasks that you can launch. A task is a piece of work that Gradle can do on a project: for example, the `build` task performs all the steps needed to fully build our Java application. Double-click on it.
![Gradle intro]({{ site.baseurl }}/images/gradle_intro_3.png)
A run panel opens in the lower part of the IDE, showing what the task does. You should see something like this:

{% highlight gradle %}
20:22:13: Executing 'build'...

> Task :compileJava
> Task :processResources NO-SOURCE
> Task :classes
> Task :jar
> Task :startScripts
> Task :distTar
> Task :distZip
> Task :assemble
> Task :compileTestJava NO-SOURCE
> Task :processTestResources NO-SOURCE
> Task :testClasses UP-TO-DATE
> Task :test NO-SOURCE
> Task :check UP-TO-DATE
> Task :build

BUILD SUCCESSFUL in 328ms
5 actionable tasks: 5 executed
20:22:14: Execution finished 'build'.
{% endhighlight %}
As you can see, several tasks were executed one after the other, and a new folder named `build` appeared in your project. Inside it, you will find several subfolders (`classes`, `distributions`, `generated`, `libs`, and so on) that contain your application in different "formats": as a set of `.class` files, as a `.jar` library, or as archives (`.zip` and `.tar`) ready for distribution.  

Now double-click on the **run** task, under the `application` folder:
{% highlight gradle %}
20:49:10: Executing 'run'...

> Task :compileJava UP-TO-DATE
> Task :processResources NO-SOURCE
> Task :classes UP-TO-DATE

> Task :run
Hello and welcome!i = 1
i = 2
i = 3
i = 4
i = 5

BUILD SUCCESSFUL in 163ms
2 actionable tasks: 1 executed, 1 up-to-date
20:49:11: Execution finished 'run'.
{% endhighlight %}
This task performs all the steps needed to run the application: first, it calls the `compileJava` task, which (as the name suggests) compiles the Java code into `.class` files, and then it runs the program. This time, no `.jar`, `.zip`, or `.tar` file is generated, because we only wanted to run the application.  

The `build` folder contains only generated files, so you can safely delete it at any time: the **clean** task does exactly this, and the next build will regenerate everything from scratch.

### Project Structure
Let's take a closer look at the files that IntelliJ created for us:
{% highlight bash %}
Gradle-GettingStarted/
├── build.gradle.kts
├── settings.gradle.kts
├── gradlew
├── gradlew.bat
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
└── src/
    ├── main/
    │   ├── java/
    │   └── resources/
    └── test/
        ├── java/
        └── resources/
{% endhighlight %}
1. `build.gradle.kts`: the build file, which describes how to build the project. We will analyze it in the next section.
2. `settings.gradle.kts`: the settings file. It defines the name of the project (`rootProject.name = "Gradle-GettingStarted"`) and, in bigger projects, the list of subprojects (modules) that are part of the build.
3. `gradlew`, `gradlew.bat`, and the `gradle/wrapper` folder: the Gradle Wrapper (see below).
4. `src`: the source code. Gradle follows a convention: the Java code of the application goes in `src/main/java`, and the files that the application needs at runtime (for example, configuration files) go in `src/main/resources`. Similarly, tests go in `src/test/java`, and the files they need go in `src/test/resources`. If you follow this convention, Gradle already knows where to find everything, and you don't need to configure anything.

### Command-Line Gradle
As we have seen, the root folder of our project contains two scripts: `gradlew` and `gradlew.bat`. They are the Gradle Wrapper, and they let you launch tasks directly from the terminal. Try to run `./gradlew tasks` (on macOS / Linux) or `gradlew.bat tasks` (on Windows): this command lists all the available tasks. In general, you can launch a task with `./gradlew <task>` (or `gradlew.bat <task>`), replacing `<task>` with the name of the task. For example:  
![Gradle intro]({{ site.baseurl }}/images/gradle_intro_4.png)  

Why should we use the wrapper instead of a Gradle installed on our computer? Open the file `gradle/wrapper/gradle-wrapper.properties`:
{% highlight properties %}
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-9.2.0-bin.zip
networkTimeout=10000
validateDistributionUrl=true
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
{% endhighlight %}
(Your file may show a different Gradle version, depending on your version of IntelliJ.)  

The `distributionUrl` line pins the version of Gradle used by the project. The first time you run `./gradlew`, the wrapper downloads exactly that version (and caches it in `~/.gradle/wrapper/dists`); after that, it reuses it. This means that everyone who clones the project (your colleagues, a CI server, a Docker container) builds it with the same Gradle version, without installing Gradle at all: all they need is a JDK. For this reason, the wrapper files should always be committed to your repository.

### A Quick Note About DSL
You may have noticed that, when creating the project, you could choose the Gradle DSL (Domain Specific Language), that is, the language in which the build file is written (more details on this file later). Kotlin is the preferred choice: it is more readable and, being statically typed, it catches more errors at compile time. However, the syntax differences are minimal, so you can choose whichever you prefer.  

Note that the name of the build file depends on this choice: `build.gradle.kts` with Kotlin, and `build.gradle` with Groovy. In the rest of the course, when we write "build.gradle" we mean the build file in general, whichever DSL you chose.
<div class="lesson-nav">
    <div>
        Next: <a href="{{ site.baseurl }}/gradle/build.gradle">Gradle - build.gradle</a>
    </div>
</div>