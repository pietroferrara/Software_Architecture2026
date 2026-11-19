---
title: 2. Example
category: 06. Camel
order: 2
exclude: false
excluded_in_search: false
---
<h2>Contents</h2>
* toc
{:toc}
## Export Students to FTP
You want to integrate in your Students App (the RESTful one that we built in the <a href="{{ site.baseurl }}/http/spring-rest">Spring REST</a> lesson) an external service to do some analytics. This service, unfortunately, is not managed directly by you and does not have API endpoints to communicate with.  
The external agency that manages this service told you: "don't worry, just put every student in a folder on an FTP server and we manage the import on our side. Students must be in JSON format, one student per file".  
How can we do this? Let's see an example with Camel.
### Requirements
Before writing code, we should sit and think about the overall export process. It is required to:
- **Avoid multiple exports**: a student should be exported only one time. We need to figure out how to do this: for example, we can add to our `Student` model a boolean flag that tells whether the student has already been exported or not.
- **Asynchronous export**: the company that manages the external service says that they will perform an import daily, so there is no need to export a student in a hurry. The idea is to schedule a job that every few minutes (or hours) exports the students that have not been exported yet. An asynchronous job also lets us handle errors better: for example, the application continues to work even if there are problems with the FTP server.
- **File format**: we know that we need to export students in JSON format. The company tells us that the available fields are: name, surname, id, and exportDate. This is not a problem, because our application already has the first three fields, and the exportDate can be generated on demand.

## The FTP server
We want to test the integration before going live, so we start an FTP server on our machine with Docker:
{% highlight bash %}
docker run -d --name studentsftp \
  -p 21:21 -p 21000-21010:21000-21010 \
  -e USERS="studentsapp|stUd3nts@pp" \
  delfer/alpine-ftp-server
{% endhighlight %}
This command starts an FTP server with a user named `studentsapp` (password `stUd3nts@pp`). Port 21 is the FTP control port, while the ports 21000-21010 are used to transfer the files (FTP in *passive mode* opens a new connection for every transfer).

## Gradle dependencies
Camel provides a set of *starters* for Spring Boot, which work like the Spring Boot ones: they add the dependencies and configure Camel automatically. Let's add to our build.gradle.kts the Camel BOM (Bill of Materials: it fixes the version of all the Camel dependencies, so we don't need to repeat it), and the starters that we need: Camel itself, the FTP component, and Jackson (to convert objects into JSON):
{% highlight kotlin %}
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-webmvc")
    implementation("org.springframework.boot:spring-boot-starter-hateoas")
    implementation(platform("org.apache.camel.springboot:camel-spring-boot-bom:4.23.0"))
    implementation("org.apache.camel.springboot:camel-spring-boot-starter")
    implementation("org.apache.camel.springboot:camel-ftp-starter")
    implementation("org.apache.camel.springboot:camel-jackson-starter")
    developmentOnly("org.springframework.boot:spring-boot-devtools")
    testImplementation("org.springframework.boot:spring-boot-starter-webmvc-test")
    testImplementation("org.springframework.boot:spring-boot-starter-hateoas-test")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}
{% endhighlight %}

## Changes to the Student model
We want to save the export status of a student: to do this, we add an `exported` field (with its getter and setter) to our `Student` model:
{% highlight java %}
package com.example.studentsapp.models;

import com.example.studentsapp.controllers.StudentController;
import com.fasterxml.jackson.annotation.JsonIgnore;
import org.springframework.hateoas.RepresentationModel;

import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.linkTo;
import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.methodOn;

public class Student extends RepresentationModel<Student> {
    private String name;
    private String surname;
    private String id;
    @JsonIgnore
    private boolean exported = false;

    // constructor, getters, setters and toString: unchanged

    public boolean isExported() {
        return exported;
    }

    public void setExported(boolean exported) {
        this.exported = exported;
    }
}
{% endhighlight %}
Note the `@JsonIgnore` annotation: it tells Jackson to ignore this field, so it is not shown in REST responses (and a client cannot set it in a POST). A new student starts as not exported.  
Then, we add two methods to `StudentService`: one that returns the students that have not been exported yet, and one that marks a student as exported:
{% highlight java %}
    public List<Student> findNotExported() {
        List<Student> result = new ArrayList<>();
        for (Student s : studentRepository.getAll()) {
            if (!s.isExported())
                result.add(s);
        }
        return result;
    }

    public void markExported(String id) {
        studentRepository.fetch(id).setExported(true);
    }
{% endhighlight %}

## Export model
We want a specific class for the export: we keep the model used by the application separate from the model used by the export, so that we can change one of them without affecting the other (for example, the export has an exportDate, which makes no sense in our application). Our `ExportStudent` model (in the `models` package) is just a POJO, with no annotations: it takes the data from a `Student`, and sets the exportDate to the current date:
{% highlight java %}
package com.example.studentsapp.models;

import java.util.Date;

public class ExportStudent {
    private String name;
    private String surname;
    private String id;
    private Date exportDate;

    public ExportStudent(Student student) {
        this.name = student.getName();
        this.surname = student.getSurname();
        this.id = student.getId();
        this.exportDate = new Date();
    }

    public String getName() {
        return name;
    }

    public String getSurname() {
        return surname;
    }

    public String getId() {
        return id;
    }

    public Date getExportDate() {
        return exportDate;
    }
}
{% endhighlight %}

## Camel Route
What remains to do is to define our Camel Route. Create a new `camel` package, with a `StudentsExport` class:
{% highlight java %}
package com.example.studentsapp.camel;

import com.example.studentsapp.models.ExportStudent;
import com.example.studentsapp.models.Student;
import com.example.studentsapp.services.StudentService;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.model.dataformat.JsonLibrary;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class StudentsExport extends RouteBuilder {
    @Autowired
    private StudentService studentService;

    @Value("${camel.ftp.uri}")
    private String ftpURI;

    @Override
    public void configure() {
        from("timer:export?period=60000")
            .bean(studentService, "findNotExported")
            .choice()
                .when(simple("${body.isEmpty()}"))
                    .log("No students to export...")
                .otherwise()
                    .split(body())
                        .setHeader("StudentID", simple("${body.id}"))
                        .setHeader("CamelFileName", simple("${body.id}.json"))
                        .log("Exporting student ${header.StudentID}")
                        .process(exchange -> {
                            Student student = exchange.getIn().getBody(Student.class);
                            exchange.getIn().setBody(new ExportStudent(student));
                        })
                        .marshal().json(JsonLibrary.Jackson)
                        .to(ftpURI)
                        .bean(studentService, "markExported(${header.StudentID})")
                    .end()
            .end();
    }
}
{% endhighlight %}
Let's spend some words here. We annotated the class as a Spring `@Component`: in this way, Spring creates it, and Camel finds our route and starts it together with the application. Since it is a bean, we can also `@Autowired` our `StudentService` inside it. In the `configure()` method, all the logic is defined:
1. `from("timer:export?period=60000")`: the <a target="_blank" rel="noopener noreferrer" href="https://camel.apache.org/components/next/timer-component.html">timer</a> component starts a new message every 60000 milliseconds (every minute).
2. `.bean(studentService, "findNotExported")`: Camel calls the `findNotExported` method of our service, and its result (the list of students to export) becomes the body of the message.
3. `.choice()`: it is like an if. When the body is empty (i.e., there are no students to export), we log this fact and the route ends here.
4. `.otherwise()`: we split the body: since we can have more than one student to export, `split` creates a new message for every element of the list, and the following steps are executed once for every student.
5. We set two headers: `StudentID`, to keep track of the id of the student, and `CamelFileName`, the special header that the FTP (and File) component uses as the name of the file (studentId.json).
6. `.process(...)`: we already saw processors in the previous section. Here, we replace the body (a `Student`) with the corresponding `ExportStudent`.
7. `.marshal().json(JsonLibrary.Jackson)`: we convert the `ExportStudent` into JSON, using Jackson.
8. `.to(ftpURI)`: we upload the file to the FTP server.
9. `.bean(studentService, "markExported(${header.StudentID})")`: finally, we mark the student as exported, so that it will not be exported again in the next run. Note that we can pass parameters to the method, using the headers of the message.

Note the expressions like `${body.id}`: they are written in <a target="_blank" rel="noopener noreferrer" href="https://camel.apache.org/components/next/languages/simple-language.html">Simple</a>, a small language of Camel that permits to access the message (the body, the headers, ...).  
`ftpURI` is a class attribute, and the `@Value("${camel.ftp.uri}")` annotation means that its value is taken from the `camel.ftp.uri` property. Add it to application.properties:
{% highlight bash %}
camel.ftp.uri=ftp://studentsapp@localhost:21/students/export?password=stUd3nts@pp&passiveMode=true
{% endhighlight %}
The URI tells Camel to connect to the FTP server on localhost (port 21) with user `studentsapp`, and to put the files in the `students/export` folder (Camel creates it, if it does not exist). Keeping it in application.properties, instead of hardcoding it in the route, permits us to change the FTP server (for example, the real one of the external agency) without changing the code.  
Note that plain FTP is not secure: passwords and files travel unencrypted. That is fine for our local test, but real integrations use SFTP (FTP over SSH), which Camel supports with the same component: just change the URI to `sftp://...`.

## Testing
Let's test the integration. Start the FTP server (if it is not already running), and then our application:
{% highlight bash %}
./gradlew bootRun
{% endhighlight %}
In the logs, you should see that Camel starts our route, and every minute it tells us that there is nothing to export:
{% highlight bash %}
Apache Camel 4.23.0 (camel-1) is starting
Routes startup (total:1)
    Started route1 (timer://export)
Apache Camel 4.23.0 (camel-1) started in 357ms (build:0ms init:0ms start:357ms boot:4s356ms)
No students to export...
{% endhighlight %}
Now, add some students with a POST to `/students`, as we did in the Spring REST lesson, and wait for the next run of the route:
{% highlight bash %}
Exporting student 32412
Exporting student 859156
Exporting student 87543
Exporting student 99654
No students to export...
{% endhighlight %}
Every student is exported only once: in the following runs, there is nothing to export. We can check the exported files from Docker Desktop (Files tab of the FTP container), or from the terminal with `docker exec studentsftp ls /ftp/studentsapp/students/export`:
![Camel Example 4]({{ site.baseurl }}/images/camel_example_4.png)
Let's add a new Student:
![Camel Example 6]({{ site.baseurl }}/images/camel_example_6.png)
After at most one minute, the new student is exported too:
![Camel Example 9]({{ site.baseurl }}/images/camel_example_9.png)
We can also connect to our FTP Server using an FTP Client like, for example, <a target="_blank" rel="noopener noreferrer" href="https://filezilla-project.org/">FileZilla</a> (host localhost, user studentsapp, password stUd3nts@pp):
![Camel Example 11]({{ site.baseurl }}/images/camel_example_11.png)

Now, try to restart the application, and add the same students again: they are exported again, and the files on the FTP server are overwritten! This happens because our students (and their exported flag) live in memory, and are lost when the application stops. To solve this problem, we need to store our data in a database: we will see how in the following lessons.
## Exercises
1. Change the route so that it also writes a copy of every exported file in a local folder, using the File component that we saw in the previous section.
1. What happens if the FTP server is down? Stop the FTP container (`docker stop studentsftp`), add a student, and look at the logs. Is the student marked as exported? Then, restart the container (`docker start studentsftp`) and see what happens.
<div class="lesson-nav">
    <div>
    Previous: <a href="{{ site.baseurl }}/camel/introduction">Camel - Introduction</a>
    </div>
</div>
