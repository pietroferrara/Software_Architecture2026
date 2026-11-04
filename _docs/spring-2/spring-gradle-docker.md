---
title: 3. Spring + Gradle + Docker
category: 04. Java Spring (Part II)
order: 3
exclude: false
excluded_in_search: false
---
<h2>Contents</h2>
* toc
{:toc}
Before proceeding, we want to connect all the dots seen so far and see how we can deploy a Spring application (the `StudentsApp` from the previous section) as a Docker container.

## The Dockerfile
Let's follow what the Spring Boot <a target="_blank" rel="noopener noreferrer" href="https://spring.io/guides/gs/spring-boot-docker">documentation</a> says. The idea is to copy and execute the `.jar` file produced by Gradle inside a Docker container.  

First, we need to build the application:
{% highlight bash %}
./gradlew build
{% endhighlight %}
This command generates the application's `.jar` files. Where? As we saw in the Gradle lesson, they are located inside `build/libs`:
{% highlight bash %}
~/IdeaProjects/StudentsApp ls build/libs
StudentsApp-0.0.1-SNAPSHOT-plain.jar StudentsApp-0.0.1-SNAPSHOT.jar
~/IdeaProjects/StudentsApp 
{% endhighlight %}
As you can see, we have two `.jar` files:
1. `StudentsApp-0.0.1-SNAPSHOT.jar` is produced by the `bootJar` task of the Spring Boot plugin. It is an executable jar that contains our code **and** all its dependencies (Spring, Tomcat, ...), so it can be launched on its own with `java -jar`.
2. `StudentsApp-0.0.1-SNAPSHOT-plain.jar` is produced by the standard `jar` task of Gradle, and it contains only our classes, without the dependencies: it can't be launched on its own.

We don't want the `-plain.jar` one. To tell Gradle not to produce this file, we add this to our `build.gradle.kts`:
{% highlight kotlin %}
tasks.jar {
    enabled = false
}
{% endhighlight %}
Run the `clean` task and then the `build` task again (`./gradlew clean build`): now you should see only one `.jar` file.  

Then, we create our Dockerfile in the root folder of our project:
{% highlight docker %}
FROM eclipse-temurin:17-jre
COPY build/libs/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
{% endhighlight %}
Let's see it line by line:
1. `FROM eclipse-temurin:17-jre`: the base image contains Java 17, the same version we chose for our project. We only need to **run** the application (Gradle already compiled it on our machine), so the JRE (Java Runtime Environment) is enough: we don't need the full JDK, and the resulting image is much smaller.
2. `COPY build/libs/*.jar app.jar`: copies our executable jar into the image (this is why we wanted only one `.jar` file in `build/libs`).
3. `ENTRYPOINT ["java","-jar","/app.jar"]`: launches the application when the container starts.

After that, we can build the image and run a container:
{% highlight bash %}
docker build -t unive/spring/student .
{% endhighlight %}
{% highlight bash %}
docker run -p 8081:8080 unive/spring/student
{% endhighlight %}
Inside the container, Spring listens on port 8080, as usual; with `-p 8081:8080`, we map it to port 8081 of our machine. So, go to <a target="_blank" rel="noopener noreferrer" href="http://localhost:8081/students">localhost:8081/students</a>:
![Spring docker]({{ site.baseurl }}/images/spring_docker_1.png)
Remember: the image contains the `.jar` file that existed when you ran `docker build`. If you change the code, you need to run `./gradlew build` and `docker build` again.
<div class="lesson-nav">
    <div>
    Previous: <a href="{{ site.baseurl }}/spring-2/application-example">Java Spring - Example of a Spring Application</a>
    </div>
</div>
