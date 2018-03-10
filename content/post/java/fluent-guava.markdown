---
title: Fluent Guava
date: 2012-05-31
comments: true
categories: [java, functional_programming]
keywords: "guava, java, functional programming"
aliases: 
  - /blog/2012/05/31/fluent-guava/
  - /2012/05/31/fluent-guava/
---

It's been far too long since I've written anything on this site.  I've got plenty of topics, it's just finding the time to write something down.  This one comes from a comment on one of my previous posts about Guava and [Modern Java]({{< relref "modern-java.markdown" >}}).

Guava has recently been upgraded to version 12 and along with this release comes the idea of a [Fluent Interface](http://martinfowler.com/bliki/FluentInterface.html) as described by Martin Fowler and Eric Evans.  This might bring it more in line with TotallyLazy and the reason I took to the library so quickly.  I hope to introduce to you the `FluentIterable` class and show how it can improve your Java.

<!-- more -->

The first thing you'll notice is that like TotallyLazy we have to convert our dull Java collections into the rich collection offered by the fluent interface.  This is a simple step using the <code>from</code> class method on the interface itself.

``` java
import static java.util.Arrays.asList;
import static com.google.common.collect.FluentIterable.from;

List<Integer> javaList = asList(1, 2, 3, 4, 5);
FluentIterable<Integer> richList = from(javaList);
```

Now we have all the power in the `richList`.  But once again, if we need to return normal Java lists we have to convert back again.  Note: The list returned from Guava will be immutable so you can't go changing it afterwards (a good thing imho).

``` java
List<Integer> javaList = richList.toImmutableList();
```

So now we know how to get hold of a collection using the fluent interface what can we do with it?  Well pretty much all the things we could do with Guava functions/predicates before only this time the language should read more fluently.  Let's take a look at an example using both the fluent interface and the old style.

``` java
List<Integer> numberList = asList(1, 2, 3, 4, 5);

transform(numberList, byHalf()); // old style
        
FluentIterable numbers = from(numberList); // fluent interface
numbers.transform(byHalf());

// assume byHalf() returns us a Guava Function<Integer,Integer>
```

I'm sure you're thinking that the fluent interface takes more code.  And in this simple example it does because we need to enrich the standard java list.  Let's go for a few more examples of different operations.

``` java
List<Integer> onlyEvens = numbers.filter(byEvenNumbers());
List<Integer> bigNumbers = numbers.filter(bigValues());
List<Integer> smallSet = numbers.limit(5);
boolean hasLargeOnes = numbers.anyMatch(bigValues());

// one of my favourites as I always find it missing
List<Integer> first = numbers.first();
```

For me this reads better and the transformation operation is now on the object concerned instead of some global method somewhere inside Guava.  Lets take my previous example from Guava and TotallyLazy and compare them with the fluent interface.

### Guava

``` java
public List<Integer> calculate(List<Integer> input) {
    List<Integer> evenNumbers = transform(input, intoEvenNumber());
    List<Integer> halvedIfMultipleOfFour = transform(evenNumbers, halfIfMultipleOfFour());
    return transform(halvedIfMultipleOfFour, addThreeIfContainsNumberTwo());
}
```

### Totally Lazy

``` java
private Sequence<Integer> calculate(Sequence<Integer> input) {
    return input
        .map(intoEvenNumber())
        .map(halfIfMultipleOfFour())
        .map(addThreeIfContainsNumberTwo());
}
```

### Guava Fluent Interface

``` java
private FluentIterable<Integer> calculate(FluentIterable<Integer> input) {
    return input
        .transform(intoEvenNumber())
        .transform(halfIfMultipleOfFour())
        .transform(addThreeIfContainsNumberTwo());
}
```

Not bad and very similar to the TotallyLazy example.
We have a fluent syntax which we can replace all our old guava code with.

## Conclusion

So for me Guava has finally the beginning of an expressive language that allows us to apply functions to our collections.  I'm sure there's more to come but this for me is a good start and means that on projects that use Guava I can hope to improve the readability of some of the code.

### Links

* [Guava FluentIterable JavaDoc](http://docs.guava-libraries.googlecode.com/git-history/v12.0/javadoc/index.html)
* [Example Project on GitHub](https://github.com/tonyklawrence/blog-posts/tree/master/fluent-guava)
