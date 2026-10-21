---
title: 3. Testing Java Applications
category: 02. Gradle
order: 3
---
<h2>Contents</h2>
* toc
{:toc}

## Testing
Testing is a program analysis technique that aims to check the correctness of a piece of code by observing its executions.
Suppose that we have this Java application:  

{% highlight java %}
package org.example;

public class Main {

    public static int add(int a, int b) {
        return Math.addExact(a, b);
    }

    public static void main(String[] args) {
        int a = Integer.parseInt(args[0]);
        int b = Integer.parseInt(args[1]);

        int result = Main.add(a, b);

        System.out.println("The result is "+result);
    }
}
{% endhighlight %}
We want to check whether the function `add` behaves correctly. It should return the sum of two integers: for example, given 3 and 4, `Main.add` should return 7.   
We can check it manually by running the `main` method, which takes the two numbers to add as arguments. With Gradle's `run` task, we can pass arguments using the `--args` flag:
{% highlight bash %}
~/IdeaProjects/Gradle-GettingStarted ./gradlew run --args="3 4"

> Task :run
The result is 7

BUILD SUCCESSFUL in 799ms
2 actionable tasks: 1 executed, 1 up-to-date
~/IdeaProjects/Gradle-GettingStarted 
{% endhighlight %}
The function seems to behave correctly. But what about all the other integers? We don't know. Let's try some other cases:
{% highlight bash %}
~/IdeaProjects/Gradle-GettingStarted ./gradlew run --args="-3 -10"

> Task :run
The result is -13

BUILD SUCCESSFUL in 810ms
2 actionable tasks: 1 executed, 1 up-to-date
~/IdeaProjects/Gradle-GettingStarted ./gradlew run --args="56 -1" 

> Task :run
The result is 55

BUILD SUCCESSFUL in 728ms
2 actionable tasks: 1 executed, 1 up-to-date
~/IdeaProjects/Gradle-GettingStarted ./gradlew run --args="452 123"

> Task :run
The result is 575

BUILD SUCCESSFUL in 770ms
2 actionable tasks: 1 executed, 1 up-to-date
~/IdeaProjects/Gradle-GettingStarted ./gradlew run --args="-452 123"

> Task :run
The result is -329

BUILD SUCCESSFUL in 717ms
2 actionable tasks: 1 executed, 1 up-to-date
~/IdeaProjects/Gradle-GettingStarted 
{% endhighlight %}
What we are doing here is called manual testing: a person (the tester) puts themselves in the shoes of an end user and tries the application's features to make sure that they behave correctly. However, testing an application this way is time-consuming, costly, and prone to human error.  

Automated testing comes to our rescue: instead of checking the program by hand, we write software that checks it for us. Rather than manually trying some combinations of numbers and verifying each sum, we define a set of cases that we want to test, and then let a program automatically verify that each of them produces the correct result. In this lesson, we will see how to do this with <a target="_blank" rel="noopener noreferrer" href="https://junit.org/">JUnit</a>, a Java framework for writing and running tests.  

The benefits of automated testing become clear with more complex examples. Suppose that you wrote a function and you want to test it on 100 different inputs. You start testing by hand and, at the 99th input, you find a bug. You fix the code... and now you need to redo all the tests from the beginning! With automated tests, you can rerun all of them at once, in a few seconds.

**NOTE**: *this doesn't mean that manual testing is useless. It is needed in the early phases of software development, before automating, to check whether automation is feasible. Moreover, when you download and try the beta version of an application, you are actually performing a form of manual testing.*
### Unit Testing and Integration Testing
Before going into details, let's say something about the different kinds of testing. Here, we will focus on unit testing, that is, testing the correctness of a software component (for example, a function or a class) in isolation. This is the first phase of testing, and it is considered white-box testing: these tests are written by developers who know the internal design of the software.  

If we want to test how two or more components interact with each other, we talk about integration testing. Integration tests come after unit tests: rather than the internal logic of each unit, they check the correctness of the interfaces between them (for example, testing that our application correctly saves data to a real database and reads it back). Going further, end-to-end tests check the whole application from the user's point of view, without any knowledge of its internal design (black-box testing): for example, testing that if you log in to a website you are redirected to your account page.
## JUnit
<a target="_blank" rel="noopener noreferrer" href="https://junit.org/">JUnit</a> is a unit testing framework for the Java programming language. Let's see how it works.  

First, we need the JUnit dependencies in our build file. We already added them to the Gradle-GettingStarted project, so you can just reuse it (replace the `Main` class with the one shown above):
{% highlight kotlin %}
dependencies {
    testImplementation(platform("org.junit:junit-bom:6.1.3"))
    testImplementation("org.junit.jupiter:junit-jupiter")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}
{% endhighlight %}
- `testImplementation` declares the dependencies needed to compile and run the tests: here, the JUnit BOM (which fixes the version of all the JUnit modules) and `junit-jupiter` (the API that we use to write tests, and the engine that runs them).
- `testRuntimeOnly` declares the dependencies needed only when running the tests: here, the launcher that Gradle uses to start them.

Now, inside `src/test/java`, create a class called `UnitTests` with this content:
{% highlight java %}
import org.example.Main;
import org.junit.jupiter.api.Test;

public class UnitTests {
    
    @Test 
    public void testAdd() {
        if (Main.add(12, 34) != 46) {
            throw new ArithmeticException("Wrong!");
        }
    } 
}
{% endhighlight %}
Let's spend a few words on the method `testAdd`. The `@Test` <a target="_blank" rel="noopener noreferrer" href="https://en.wikipedia.org/wiki/Java_annotation">annotation</a> marks the method as a test. The body is self-explanatory: it checks that the value returned by `add` is correct and, if not, it throws an exception. This is exactly what we did by hand a few minutes ago. If the test throws an exception, it fails: there is something wrong with the method.  

You can run the tests from the terminal with `gradlew`:
{% highlight bash %}
./gradlew test # launch all the defined test methods
./gradlew test --tests 'UnitTests.testAdd' # launch test method testAdd of class UnitTests
{% endhighlight %}
If you are using IntelliJ, you have probably noticed a play button next to the method signature:
![Gradle testing]({{ site.baseurl }}/images/gradle_testing_1.png)
Clicking on it, you can run the test or, among other options, run it with coverage. Choose "Run with Coverage". A window will appear on the right, showing the coverage, i.e., the percentage of your code that the tests reach. In our case, we cover 100% of the classes, 50% of the methods (1 out of 2), and 20% of the lines. Open the `Main` class and notice the green mark on the left of the function `add`: it means that `testAdd` executes the function completely. Of course, we never call `main` from the test, so `main` has no coverage (highlighted in red).
![Gradle testing]({{ site.baseurl }}/images/gradle_testing_2.png)
### Assertions
Writing an `if` and throwing an exception by hand for every check works, but it quickly becomes verbose, and when the test fails, the message "Wrong!" doesn't tell us much. JUnit provides a set of <a target="_blank" rel="noopener noreferrer" href="https://docs.junit.org/current/writing-tests/assertions.html">assertions</a>, i.e., static methods of the class `org.junit.jupiter.api.Assertions` that check a condition and make the test fail if the condition does not hold. Let's rewrite our test with them:
{% highlight java %}
import org.example.Main;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

public class UnitTests {

    @Test
    public void testAdd() {
        assertEquals(46, Main.add(12, 34));
        assertEquals(-13, Main.add(-3, -10));
        assertEquals(0, Main.add(5, -5));
    }

    @Test
    public void testAddOverflow() {
        assertThrows(ArithmeticException.class, () -> Main.add(Integer.MAX_VALUE, 1));
    }
}
{% endhighlight %}
The most common assertions are:
1. `assertEquals(expected, actual)`: checks that two values are equal. Note the order of the arguments: first the value that we expect, then the value computed by our code.
2. `assertTrue(condition)` and `assertFalse(condition)`: check that a boolean condition is true (or false).
3. `assertNull(value)` and `assertNotNull(value)`: check that a value is (or is not) `null`.
4. `assertThrows(exceptionClass, code)`: checks that the given code (a lambda) throws an exception of the given class.

The second test, `testAddOverflow`, deserves a few words. An `int` in Java has a maximum value (`Integer.MAX_VALUE`, i.e., 2147483647): what happens if we add 1 to it? With the `+` operator, the result silently "wraps around" and becomes a negative number, which is clearly wrong. Our `add` function uses `Math.addExact`, which instead throws an `ArithmeticException` in case of overflow, and the test checks exactly this behavior. Testing what happens with "wrong" or extreme inputs is as important as testing the normal cases.  

When an assertion fails, the test fails, and JUnit records exactly what went wrong. For example, if we write `assertEquals(47, Main.add(12, 34))`, `./gradlew test` fails and prints which test failed, together with the line of the failing assertion:
{% highlight bash %}
UnitTests > testAdd() FAILED
    org.opentest4j.AssertionFailedError at UnitTests.java:10
{% endhighlight %}
The details of the failure (`expected: <47> but was: <46>`) are shown in the IntelliJ test panel, and in the HTML report that Gradle generates at `build/reports/tests/test/index.html` (open it with your browser).
## The importance of writing tests that make sense
Our test passed and covered all the lines of our `add` function. But are we really, really, really sure that the function is bug-free? Spoiler: no. To see why, we need a more complex example. Suppose that we have this function, which checks whether two strings contain the same number of 0s (add it to the `Main` class):
{% highlight java %}
public static boolean equalZero(String a, String b) {
    int countA = 0;
    int countB = 0;
    for (int i = 0; i < a.length(); i++) {
        if (a.charAt(i) == '0') {
            countA += 1;
        }
    }
    for (int i = 0; i < a.length(); i++) {
        if (b.charAt(i) == '0') {
            countB += 1;
        }
    }

    return countA == countB;
}
{% endhighlight %}
And this test (add it to the `UnitTests` class):
{% highlight java %}
@Test
public void testEqual0() {
    assertTrue(Main.equalZero("Hell0", "Hell0 World!"));
    assertTrue(Main.equalZero("0100001111", "10000101111"));
    assertFalse(Main.equalZero("0", "1"));
    assertTrue(Main.equalZero("", "11"));
    assertTrue(Main.equalZero("", ""));
}
{% endhighlight %}
Let's run the test:
{% highlight bash %}
~/IdeaProjects/Gradle-GettingStarted ./gradlew test --tests 'UnitTests.testEqual0'

> Task :compileJava
> Task :processResources NO-SOURCE
> Task :classes
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test

BUILD SUCCESSFUL in 1s
3 actionable tasks: 3 executed
~/IdeaProjects/Gradle-GettingStarted 
{% endhighlight %}
The test passed. We tried empty strings, strings of different lengths, and strings without zeros, so we may be convinced that the function is correct. But... let's add this test case:
{% highlight java %}
assertFalse(Main.equalZero("0011", "11000"));
{% endhighlight %}
Run the test again and notice the failure. This simple example shows that a bug can hide very well in the code: even with multiple test cases that cover all the lines of a method, bugs can go unnoticed. To write meaningful tests, it is important to reason about all the edge cases.

## Exercises
1. Fix the `equalZero` function. Can you also make it more modular?
2. You have this Java class:
{% highlight java %}
package org.acme;

class QuickSort {

    static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

    static int partition(int[] arr, int low, int high) {
        int pivot = arr[high];
        int i = (low - 1);

        for (int j = low; j <= high - 1; j++) {
            if (arr[j] < pivot) {
                i++;
                swap(arr, i, j);
            }
        }
        swap(arr, i + 1, high);
        return (i + 1);
    }

    static void sort(int[] arr, int low, int high) {
        if (low < high) {
            int pi = partition(arr, low, high);

            sort(arr, low, pi - 1);
            sort(arr, pi + 1, high);
        }
    }
}
{% endhighlight %}
Create a `Main` class that uses the method `sort`, and write a build file for the project. Then, write some tests.  
<div class="lesson-nav">
    <div>
        Previous: <a href="{{ site.baseurl }}/gradle/build.gradle">Gradle - build.gradle</a>
    </div>
    <div>
        Next: <a href="{{ site.baseurl }}/gradle/example-log4j">Gradle - Example: log4j</a>  
    </div>
</div>