---
title: 3. Spring REST
category: 05. HTTP
order: 3
exclude: false
excluded_in_search: false
---
<h2>Contents</h2>
* toc
{:toc}
## Introduction
Here, we are going to build a RESTful application in Spring. With the term RESTful, we mean a web service that adheres to the REST architectural style.

In Spring, everything comes easy. Let's make our Students App (the one we built in the <a href="{{ site.baseurl }}/spring-2/application-example">Spring Part II</a> lesson) RESTful.
### Students App
In the previous lesson, we built our first Spring Controller:
{% highlight java %}
@RestController
public class StudentController {
    @Autowired
    private StudentService studentService;

    @GetMapping("/students")
    public String getStudents() {
        return studentService.findAll().toString();
    }

    @GetMapping("/students/add")
    public String addStudent(@RequestParam String id, @RequestParam String name, @RequestParam String surname) {
        studentService.add(name, surname, id);
        return "OK";
    }
}
{% endhighlight %}
However, this is not RESTful: REST puts the focus on resources, and resources should be clearly identifiable by identifiers and accessed by URLs. The action we want to perform on a resource is expressed by the HTTP method, not by the URL.  
Our simple example has some problems:
1. We perform a GET to insert a new student. GET should only *retrieve* information: it must not change the state of the server. Moreover, `/students/add` is not a resource, it is an action.
2. We pass all the student's info in the query string, instead of sending a representation of the student in the request body.
3. We always return an "OK" with status 200. But what happens if the student already exists in the application? What happens if we have some problems? The client has no way to know it.
4. `GET /students` returns the `toString()` of a list: this is a format that we invented, and a client would have to parse it by hand.

Let's fix them.

## Spring REST
We now present some annotations and classes that Spring uses to build REST controllers.
### @RestController
We already know this annotation: it tells Spring that our `StudentController` class is a REST Controller. `@RestController` is a specialization of `@Controller` that automatically writes the value returned by a method into the response body.
### Mapping annotations
To map specific requests to the appropriate request handlers (i.e., methods), we use mapping annotations. We already used `@GetMapping`, which maps the GET requests to a particular path to the annotated method. In the same way, we have `@PostMapping`, `@PutMapping`, `@PatchMapping`, and `@DeleteMapping` (all of them are shortcuts of the more general `@RequestMapping`, which lets you specify the method explicitly, e.g. `@RequestMapping(path = "/students", method = RequestMethod.GET)`).  
Note that two methods can share the same path, as long as they handle different HTTP methods: in a moment, we will have both `GET /students` (retrieve all the students) and `POST /students` (create a new student).
### ResponseEntity class
We want our API to return JSON, a format that every client can understand.
`ResponseEntity` is a Spring class that represents a whole HTTP response: status code, headers, and body. It is generic in the type of the object that we wish to return as the body. In our case, we want to return a list of Students:
{% highlight java %}
    @GetMapping("/students")
    public ResponseEntity<List<Student>> getStudents() {
        return ResponseEntity.ok(studentService.findAll());
    }
{% endhighlight %}
`ResponseEntity.ok(...)` builds a response with status 200 (OK) and the given body. Spring (through the <a target="_blank" rel="noopener noreferrer" href="https://github.com/FasterXML/jackson">Jackson</a> library, included in `spring-boot-starter-webmvc`) converts the list into JSON automatically, using the getters of the `Student` class, and sets the `Content-Type: application/json` header. Restart the application, add some students, and call `GET /students`: we now have a JSON array as a response. Wonderful!  
You can see how to build different responses in the <a target="_blank" rel="noopener noreferrer" href="https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/ResponseEntity.html">documentation</a>. Using `ResponseEntity`, you can, for example, set a header or return another status code without too much effort.
### @RequestBody
Now we are going to fix the creation of a student: we want to receive the student as a JSON body and throw away all the query string parameters. We can use the `@RequestBody` annotation, which works like `@RequestParam` but, instead of mapping query string parameters to the method arguments, it maps the content of the body (again, Jackson converts the JSON into a `Student` object for us). Firstly, let's change `@GetMapping` to `@PostMapping`, and change the path to identify the collection of students, `/students`. Then, we change the method signature:
{% highlight java %}
    @PostMapping("/students")
    public ResponseEntity<Void> addStudent(@RequestBody Student student)
{% endhighlight %}
Here, we are going to POST a new resource (a new Student) inside our application. If everything goes well, we return a 201 (Created) status, and we use the `Location` header to tell the client where this newly created resource is located. Since our service does not have a method that takes a `Student` yet, add it to `StudentService`:
{% highlight java %}
    public void add(Student student) {
        studentRepository.create(student);
    }
{% endhighlight %}
And then complete the controller method:
{% highlight java %}
    @PostMapping("/students")
    public ResponseEntity<Void> addStudent(@RequestBody Student student) {
        studentService.add(student);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest() // http://localhost:8080/students
                .path("/{id}")                                         // http://localhost:8080/students/{id}
                .buildAndExpand(student.getId())                       // http://localhost:8080/students/999
                .toUri();
        return ResponseEntity.created(location).build();
    }
{% endhighlight %}
`ServletUriComponentsBuilder.fromCurrentRequest()` starts from the URL of the current request, and we append the id of the new student to it. `ResponseEntity.created(location)` builds a 201 response with the `Location` header, and `build()` tells Spring that there is no body.  
Now we are ready to do a test using Insomnia (remember to select JSON as the body type, so that the `Content-Type: application/json` header is sent):
![HTTP Spring]({{ site.baseurl }}/images/http_springrest_1.png)
Or, with cURL:
{% highlight bash %}
curl -i -X POST -H "Content-Type: application/json" \
  -d '{"name": "Homer", "surname": "Simpsons", "id": "999"}' \
  http://localhost:8080/students
{% endhighlight %}
As you can see, `Location` contains the URL from which we can access the resource. However, the method that we wrote is not quite correct: can you tell why? Spoiler: what happens if we POST a new Student with the same ID? The existing student is silently overwritten! We would like to catch this case and return a 409 (Conflict) status code.
### Handle Status Codes
Add a new class that models an error response, in the `models` package:
{% highlight java %}
package com.example.studentsapp.models;

public class ErrorResponse {
    private String message;

    public ErrorResponse(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}
{% endhighlight %}
Let's make the `create` method of the repository return `false` if the Student already exists (and `true` otherwise):
{% highlight java %}
@Repository
public class StudentRepository {
    private Map<String, Student> students;

    public StudentRepository() {
        this.students = new HashMap<>();
    }

    public boolean create(Student student) {
        if (students.containsKey(student.getId())) {
            return false;
        }
        students.put(student.getId(), student);
        return true;
    }

    // getAll, fetch, search, delete: unchanged
}
{% endhighlight %}
The same in `StudentService`, which simply forwards the result. Since the `add(name, surname, id)` method was used only by the old `/students/add` endpoint, we can remove it:
{% highlight java %}
@Service
public class StudentService {
    @Autowired
    StudentRepository studentRepository;

    public List<Student> findAll() {
        return new ArrayList<Student>(studentRepository.getAll());
    }

    public Student fetch(String id) {
        return studentRepository.fetch(id);
    }

    public boolean add(Student student) {
        return studentRepository.create(student);
    }

    // search, delete: unchanged
}
{% endhighlight %}
Now, check the result inside the controller. The method can now return two different kinds of body (nothing, or an `ErrorResponse`), so we use the wildcard `ResponseEntity<?>`:
{% highlight java %}
    @PostMapping("/students")
    public ResponseEntity<?> addStudent(@RequestBody Student student) {
        if (!studentService.add(student)) {
            ErrorResponse error = new ErrorResponse("Student with this id already exists");
            return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
        }
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(student.getId())
                .toUri();
        return ResponseEntity.created(location).build();
    }
{% endhighlight %}
If we POST the same student twice, the second time we get a 409, with a body that explains the problem:
![HTTP Spring]({{ site.baseurl }}/images/http_springrest_2.png)
### @PathVariable
The `Location` header points to `/students/{id}`, but we don't have anything that answers on this path yet. Let's add a GET to fetch a particular Student by ID:
{% highlight java %}
    @GetMapping("/students/{id}")
    public ResponseEntity<Student> getStudent(@PathVariable String id) {
        Student student = studentService.fetch(id);
        if (student == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(student);
    }
{% endhighlight %}
With the `@PathVariable` annotation, the value of the method argument is taken directly from the path of the URL: the `{id}` placeholder in the mapping is bound to the `id` argument (they must have the same name). If the student does not exist, we return a 404 (Not Found).  
![HTTP Spring]({{ site.baseurl }}/images/http_springrest_3.png)
![HTTP Spring]({{ site.baseurl }}/images/http_springrest_4.png)
### The complete controller
Here is the complete `StudentController`, with all the imports:
{% highlight java %}
package com.example.studentsapp.controllers;

import com.example.studentsapp.models.ErrorResponse;
import com.example.studentsapp.models.Student;
import com.example.studentsapp.services.StudentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.support.ServletUriComponentsBuilder;

import java.net.URI;
import java.util.List;

@RestController
public class StudentController {
    @Autowired
    private StudentService studentService;

    @GetMapping("/students")
    public ResponseEntity<List<Student>> getStudents() {
        return ResponseEntity.ok(studentService.findAll());
    }

    @GetMapping("/students/{id}")
    public ResponseEntity<Student> getStudent(@PathVariable String id) {
        Student student = studentService.fetch(id);
        if (student == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(student);
    }

    @PostMapping("/students")
    public ResponseEntity<?> addStudent(@RequestBody Student student) {
        if (!studentService.add(student)) {
            ErrorResponse error = new ErrorResponse("Student with this id already exists");
            return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
        }
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(student.getId())
                .toUri();
        return ResponseEntity.created(location).build();
    }
}
{% endhighlight %}
### Add hypermedia to Response Body
We said that the most important thing in REST is the resource. To add some sort of navigation between resources, one can add to the bodies links that refer to other resources. Considering the previous GitHub example, a **GET https://api.github.com** call returns a JSON full of URLs, which permit us to navigate all the linked resources. This idea is called HATEOAS (Hypermedia As The Engine Of Application State), and we can add this capability easily to our application with <a target="_blank" rel="noopener noreferrer" href="https://spring.io/guides/gs/rest-hateoas/">**Spring HATEOAS**</a>. Firstly, add the Spring HATEOAS starter to our build.gradle.kts, then reload the Gradle project:

{% highlight kotlin %}
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-webmvc")
    implementation("org.springframework.boot:spring-boot-starter-hateoas")
    developmentOnly("org.springframework.boot:spring-boot-devtools")
    testImplementation("org.springframework.boot:spring-boot-starter-webmvc-test")
    testImplementation("org.springframework.boot:spring-boot-starter-hateoas-test")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}
{% endhighlight %}

Then, make `Student` extend `RepresentationModel`, a Spring HATEOAS class that adds a list of links to our model, and add a *self* link (a link to the resource itself) when a student is created:
{% highlight java %}
package com.example.studentsapp.models;

import com.example.studentsapp.controllers.StudentController;
import org.springframework.hateoas.RepresentationModel;

import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.linkTo;
import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.methodOn;

public class Student extends RepresentationModel<Student> {
    private String name;
    private String surname;
    private String id;

    public Student(String name, String surname, String id) {
        this.name = name;
        this.surname = surname;
        this.id = id;
        this.add(linkTo(methodOn(StudentController.class).getStudent(id)).withSelfRel()); // add a self link.
    }

    // getters, setters and toString: unchanged
}
{% endhighlight %}
`methodOn(StudentController.class).getStudent(id)` does not really call our controller: Spring HATEOAS records which method we are pointing to, and `linkTo(...)` builds the URL that is mapped to that method (`/students/{id}`, with the actual id). This way, if we change the path in the `@GetMapping`, the links follow automatically.  
![HTTP Spring]({{ site.baseurl }}/images/http_springrest_5.png)
Since our controller returns a plain `List`, the links are rendered as a simple `links` array. Spring HATEOAS also supports standard hypermedia formats, like <a target="_blank" rel="noopener noreferrer" href="https://en.wikipedia.org/wiki/Hypertext_Application_Language">HAL</a> (in which the links appear in a `_links` object): you get it when the controller returns a `RepresentationModel` directly, as in `getStudent`, or wraps the list in a `CollectionModel`. Try to call `GET /students/{id}` and compare the two responses.  

And that's it.
## Exercises
1. Add a `DELETE /students/{id}` endpoint, which returns 204 (No Content) if the student has been deleted and 404 (Not Found) if it does not exist.
1. Add a `PUT /students/{id}` endpoint that replaces the name and surname of an existing student. What should you return if the id in the path is different from the id in the body?
1. Make `GET /students` accept an optional `name` query parameter (`@RequestParam(required = false)`) that filters the students using the `search` method of the service.
<div class="lesson-nav">
    <div>
    Previous: <a href="{{ site.baseurl }}/http/rest">HTTP - REST</a>
    </div>
</div>
