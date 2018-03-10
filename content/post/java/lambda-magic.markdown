---
title: Lambda Magic?
slug: "lambda-magic"
date: 2012-01-10
comments: true
categories: [java, functional_programming]
keywords: "guava, java, functional programming"

aliases:
  - /blog/2012/01/10/lambda-magic/
  - /2012/01/10/lambda-magic/
---

I often hear people complain about Java (I’m not excluding myself from this) about how restrictive the language can be compared to others.  We all know how noisy it is when working with functional libraries.  Wouldn’t it be great if we had a lambda style syntax in Java?

This is where [Enumerable](https://github.com/hraberg/enumerable) for Java comes in.  This has been developed by [Håkan Råberg](http://ghettojedi.org/) who describes the library as:

> "Ruby/Smalltalk style internal iterators for Java 5 using bytecode transformation to capture expressions as closures."

<!-- more -->

What does that mean?  Using post processing (either during runtime or post compilation using ([ASM](http://asm.ow2.org/)) we can write a simple single expression lambda (or closure) directly in our Java code.

### The Example

As before we start off with a sequence of numbers.  Imagine we want to increment each value by 5.  A simple operation solved in an imperative manner would be something like this:

``` java
public List<Integer> addFiveToEach(List<Integer> numbers) {
    List<Integer> result = new ArrayList<Integer>();
    for (Integer number : numbers) {
        result.add(number + 5);
    }
    return result;
}
```

Can’t really complain about the code above, nice and simple, but let’s see what we can improve on with enumerable java.

### With TotallyLazy

Let me start with my new favourite TotallyLazy.  Using enumerable we can now embed a simple lamba expression right into the transformation of the number sequence (using the λ symbol.)

``` java
public Sequence<Integer> addFiveToEach(Sequence<Integer> numbers) {
    return numbers.map(λ(i, i + 5));
}
```

Amazingly this compiles directly in IntelliJ without any messing about with settings, all you need is TotallyLazy and enumerable libraries in the project.  How can this be?  Well, with a couple of static imports of course!

``` java
import static com.googlecode.totallylazy.lambda.Lambdas.λ;
import static org.enumerable.lambda.Parameters.i;
```

This first one includes the lambda symbol into the namespace and the other includes a variable named `i` of type `int`.  There are other options for different types such as `s` for `String`, `c` for `Char` but I’ll let you have the fun finding them all out.

Here’s an example of determining the length every string in the sequence (interestingly the input and output types do not have to be the same):

``` java
public Sequence<Integer> lengthOfEach(Sequence<String> strings){
    return strings.map(λ(s, s.length()));
}
```

### The caveat

There is one gotcha in this.  Because this uses byte code manipulation to convert our lambda into an anonymous function for TotallyLazy, we have to execute the code using a javaagent argument such as:

``` bash
$ java -javaagent:enumerable-java-<version>.jar [...]
```

(or we could post process out classes to remove the runtime dependency - please refer to the libraries documentation on this [here](https://github.com/hraberg/enumerable#readme)

### With Guava

Like some of my projects, you may be using Guava and do not want to switch over to TotallyLazy.  Luckily the author has allowed for this by offering a support class that will transform functions making them suitable for guava.

``` java
import static org.enumerable.lambda.support.googlecollect.LambdaGoogleCollections.function;

private List<Integer> addFiveToEach(List<Integer> numbers) {
    return transform(numbers, function(i, i + 5));
}
```

This works in exactly the same way (and requires javaagent) but transforms the function code into an anonymous Guava function.  Still, for simple transformations the syntax is pretty tidy.  Support for predicates and suppliers are also in there.

### Conclusion

I really like this idea but I have yet to really use it for any real code.  Two small things are holding me back.  Firstly is that it’s not nice running a server with the extra javaagent to manipulate the code.  Post processing during the build would be the better option but it still feels a little dirty.  Secondly, I rarely need transformations this simple, and when adding more complicated code I would also extract it out for testability and reducing noise, thus negating the need for inline lambdas.

I still think it’s pretty cool and a great way of extending the life of Java.
