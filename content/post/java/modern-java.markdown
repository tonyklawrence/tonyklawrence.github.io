---
title: "Modern Java"
slug: "modern-java"
date: 2011-12-04
comments: true
categories: [java, functional_programming]
keywords: "java, functional programming"

aliases:
  - /2011/12/04/modern-java/
  - /blog/2011/12/04/modern-java/
---

I recently attended XPDay London 2011 organised by the great eXtreme Tuesday Club and had a great 2 days.  Many of the talks I went to were more exploratory than anything but sometimes it’s a great way to learn more.  One of the early sessions was hosted by Julian Kelsey [@scrawlings](http://www.twitter.com/scrawlings) and Andrew Parker [@aparker42](http://www.twitter.com/aparker42) and was predominately about refactoring Java into a more function style, and another by Nat Pryce [@natpryce](http://www.twitter.com/natpryce) about test driving function programming which in the end turned into something called Modern Java.

<!-- more -->

From what I understood Modern Java was about taking a functional approach to programming and utilising libraries such as Googles Guava to produce simpler, immutable, easily tested and less coupled software (albeit at a risk of noise due to the java language.)

### Why would you want to do this you might ask?

Well, I’ve been working with these libraries for a few years and have realised the potential functional programming offers even to a Java developer.  Recently, I refactored some code with one of my colleagues James Bull and that process gave me the idea to write this post.  I hope to help explain how you can do this and also try to convince you that the end result is a better place to be.

To do this we are going to need an simple example, we'll need to build a calculator which will do the following:

* Given a list of numbers...
* If the number is odd add 1 to make it even
* If the new number becomes a mutliple of 4 then halve it
* If the result contains a 2 then add 3

Hopefully this is simple enough, but lets go through a few examples (would make good test cases)

* Given 1, add 1 becomes 2, is not a mutiple of 4, does contain a 2 so add 3, result 5
* Given 4, is even, is multiple of 4 so halve it which becomes 2,  does contains a 2 so add 3, result 5
* Given 7, add 1 becomes 8, is multiple of 4 so halve it which becomes 4, does not contain a 2, result 4

### Everyday TDD approach

If we are going to take the expected imperative TDD approach we would break the requirements down into doing the simplest thing possible so we might start off with just the first part of the example which will add 1 to odd numbers.  We might start with a test something like this (you might really start with a single number before handling a list but this is an example and don’t want it to get too long.)

``` java
@Test
public void addsOneToEveryOddNumber() {
    List someNumbers = asList(1, 2, 3, 5, 17, 7, 0, 14);
    List expectedNumber = asList(2, 2, 4, 6, 18, 8, 0, 14);
    assertThat(calculator.calculate(someNumbers), is(equalTo(expectedNumber)));
}
```

And the implementation of the calculator to make this pass could be:

``` java
public List<Integer> calculate(List<Integer> input) {
    List<Integer> output = new ArrayList<Integer>();

    for (Integer number : input) {
        if (number % 2 != 0) number++;
        output.add(number);
    }

    return output;
}
```

Great, we have a working test and are a third of the way through the requirements.  Next we would go onto the next part, easy.  We add a new test to check if the number is divisible by 4 we divide by 2.

Now, if we want to use a random set of numbers as input we have to factor in the first calculation (we could just use even numbers but that’s not really a great test.)  And when testing the third case we need to factor in case 1 and 2, this is making our test cases more complicated than we need to.

Anyway, we could end up with our calculator looking something like this (with a few methods extracted, not included for conciseness.)

``` java
public List<Integer> calculate(List<Integer> input) {
    List<Integer> output = new ArrayList<Integer>();

    for (Integer number : input) {
        if (odd(number)) number++;
        if (multipleOfFour(number)) number /= 2;
        if (containsATwo(number)) number += 3;
        output.add(number);
    }

    return output;
}
```

Great, it’s a nice simple, readable piece of code that has been well tested. 

Unfortunately, the test cases are quite complicated due to having to consider all 3 stages of the calculation. You could make these extracted methods public and test them but that’s breaking our encapsulation. If we decided to add a forth, we would have to change all the other tests which is not ideal.

### So how can a functional approach help us?

Using a library such as [Guava](https://github.com/google/guava) which offers us a small amount of power of functional programming, allowing us to transform, filter and find objects in lists. The guava function interface used by transform is as follows:

``` java
public interface Function {
    T apply(@javax.annotation.Nullable F f);
    boolean equals(@javax.annotation.Nullable java.lang.Object o);
}
```

By implementing the apply method we can transform the input into the required output (not required to be of the same type.)

``` java
public Integer apply(Integer number) {
    return number % 2 == 0 ? number : number + 1;
}
```

This approach enables us to extract out each of our separate concerns into individual classes and test them alone. So starting at the beginning we would start with the first case of making all numbers even.

``` java
@Test
public void oddNumbersArePromotedToNextEvenNumber() {
    assertThat(function.apply(27), is(28));
    assertThat(function.apply(4), is(4));
}
```

This way we end up with 3 functions, each doing one thing and nice simple tests. But how do we connect them together to form the calculator? We just transform the input using the new functions we’ve written (using a factory method to help reduce noise of new.)

``` java
public List<Integer> calculate(List<Integer> input) {
    List<Integer> evenNumbers = transform(input, intoEvenNumber());
    List<Integer> halvedIfMultipleOfFour = transform(evenNumbers, halfIfMultipleOfFour());
    return transform(halvedIfMultipleOfFour, addThreeIfContainsNumberTwo());
}
```

We then have a simple test for the calculator that given a set of input we get the right output, we no longer need to test all the different edge cases, each of the processes. Adding a new condition is as simple as writing the function for it, updating the calculator test with the new output and add the transformation to the code.

I've included the full class definition of one of the functions in case you'd like to try this yourself.

``` java
import com.google.common.base.Function;

public class HalfIfMultipleOfFour implements Function<Integer, Integer> {

    public static HalfIfMultipleOfFour halfIfMultipleOfFour() {
        return new HalfIfMultipleOfFour();
    }

    public Integer apply(Integer number) {
        return multipleOfFour(number) ? number / 2 : number;
    }

    private boolean multipleOfFour(Integer number) {
        return number % 4 == 0;
    }
}
```

There have been a lot of information to cover in the post, I realise that I've been quite brief however I'm happy to cover anything in more detail if you leave a comment.
