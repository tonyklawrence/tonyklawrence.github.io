---
title: "Modern Java - The Totally Lazy Way"
slug: "the-totally-lazy-way"
date: 2012-01-01
comments: true
categories: [java, functional_programming]
keywords: "guava, java, functional programming, totallylazy"

aliases:
  - /2012/01/01/modern-java-the-totally-lazy-way/
  - /blog/2012/01/01/modern-java-the-totally-lazy-way/
---

A few weeks ago I blogged about [Modern Java]({{< relref "modern-java.markdown" >}}) using Googles Guava to write functional programs. One of the comments on that blog was by a good friend on mine Franck Rasolo. He suggested I took a look at [Totally Lazy](https://github.com/bodar/totallylazy) by [Daniel Bodart](http://dan.bodar.com) as an alternative to Guava. So I decided to implement the same functional calculator I wrote in the previous post using totally lazy instead and here are my findings.

<!-- more -->

### How does TotallyLazy compare to Guava?

Assuming we have a list of numbers defined:

``` java
List<Integer> numbers = asList(2, 4, 6, 8, 10);
```

#### Guava

Imagine we want to halve each of the values, using the Guava API we can transform the values simply with an anonymous implementation of the `Function<F, T>` interface.

``` java
transform(numbers, new Function<Integer, Integer>() {
    public Integer apply(Integer from) {
        return from / 2;
    }
});
```

Obviously this isn't very pretty but we can hide this away in a named implementation with a suitably named factory method to give us something like:

``` java
transform(numbers, intoHalfTheirValue());
```

I won't go into detail here on how you can write a named function with a descriptive factory method but please check out the previous post if you'd like to know this.

As you can see the Guava solution is simple, neat and very easy to understand. So how can Totally Lazy improve on this?

#### Totally Lazy

One of the first changes when using Totally Lazy is that we no longer work with standard Java `Lists`, but with `Sequences`. This might sound like a bad thing to you but using a sequence offers us a more natural functional style to the code, and it's also very easy to convert from a `List` into a `Sequence` and back.

``` java
Sequence<Integer> sequence = sequence(numbers);
List<Integer> list = sequence.toList();
```

Using this sequence we can now map our numbers into new numbers in a similar way to Guava however instead of implementing the `Function` interface we now implement Totally Lazy's `Callable1<Input, Output>` interface.

``` java
sequence.map(new Callable1<Integer, Integer>() {
    public Integer call(Integer from) throws Exception {
        return from / 2;
    }
});
```

And showing a cleaner call with a factory

``` java
sequence.map(intoHalfTheirValue());
```

As you'll have noticed that the implementation of `Function<T, F>` and `Callable1<Input, Output>` are almost identical, only the method name differs from `apply()` to `call()`.  The main benefit of using the `Sequence` api is that we can now use method chaining (and lambdas which I'll describe in a following post.)  

So to conclude with a Totally Lazy implementation of the functional calculator from the previous post showing method chaining 

``` java
public class TotallyLazyFunctionalCalculator implements Calculator {

    public List<Integer> calculate(List<Integer> input) {
        return calculate(sequence(input)).toList();
    }
    
    private Sequence<Integer> calculate(Sequence<Integer> input) {
        return input
            .map(intoEvenNumber())
            .map(halfIfMultipleOfFour())
            .map(addThreeIfContainsNumberTwo());
    }
}
```

I think going forward I will be using Totally Lazy over Guava as it feels more natural, nesting transforms in Guava is a little noisy.  

In my next post I'll talk about using [Enumerable Java](https://github.com/hraberg/enumerable) to allow us to use lambda expressions directly in Java.
