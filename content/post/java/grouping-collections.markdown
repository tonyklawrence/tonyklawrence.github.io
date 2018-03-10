---
title: Grouping collections in Java
date: 2012-08-15
comments: true
categories: [java, functional_programming, tdd]
keywords: "java, functional programming, guava, tdd, test driven development, grouping, enumerable"

aliases: 
  - /blog/2012/08/15/grouping-collections-in-java/
  - /2012/08/15/grouping-collections-in-java/
---

Recently I came across some code that was iterating over collections in order to group them by certain fields.  This code was repeated a few times as it was grouping more than once.  To me this seemed very verbose and a little hard to understand.  As Guava was the available library and one that does not include any grouping I decided to have a go myself.

<!-- more -->

## The iterative approach

``` java
Map<Character, List<String>> group = newHashMap();
List<String> strings = asList("one", "two", "three", "four");

for (String string : strings) {
    Character firstCharacter = string.charAt(0);
    
    if (group.containsKey(firstCharacter)) {
        group.get(firstCharacter).add(string);
    } else {
        group.put(firstCharacter, asList(string));
    }
}
```

## The idea

Some functional languages do allow you to group items based upon a field such as the group by in Scala.  Here's a quick example.

``` java
val strings = "one" :: "two" :: "three" :: Nil
val groups = strings groupBy (_.charAt(0))
```

## The specification

To design the grouping I drove this from how I wanted to use it.  The simplest way to do this is from a test.  This allows me to flesh out the design as well as prove it works (and also to know when I am done.)  Simple JUnit tests will suffice.

> I'm using enumerable-java to reduce noise in my Java code (see [Lambda Magic?](/2012/01/10/lambda-magic/)) along with this λ trick for Guava

``` java
@NewLambda
private static <F, T> Function<F, T> λ(F from, T to) {
    throw new LambdaWeavingNotEnabledException();
}
```

This defines a method named `λ` which can be used as a lambda function with guava as the default enumerable implementation returns a `Callable` for [Totally Lazy](https://github.com/bodar/totallylazy)

### The Test

``` java
@Test
public void groupAListOfStrings_byTheirFirstLetter() {
    List<String> strings = asList("apples", "apricots", "oranges");

    Map<Character, Collection<String>> grouping = group(strings, λ(s, s.charAt(0));

    assertThat(grouping.size(), is(2));
    assertThat(grouping.keySet(), contains('a', 'o'));
    assertThat(grouping.get('a'), contains("apples", "apricots"));
    assertThat(grouping.get('o'), contains("oranges"));
}
```

It's pretty obvious to see that we are grouping these strings by their first letter, `a` for apples and apricots, `o` for oranges.  We have a number of assertions to make sure that the group is how we expected.

## The implementation

Now we have our specification we can start to implement the function.  Our test has already given us our signature.

``` java
Map<Character, Collection<String>> group(Collection<String> strings, Function<Character, String> function);
```

Now we can simply iterate over the given list, applying the given function and placing them in the result (using Guava's [ListMultimap](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/ListMultimap.html) for ease.

``` java
ListMultimap<Character, String> groups = ArrayListMultimap.create();

for (String string : string)
    groups.put(function.apply(string), string);

return groups.asMap();
```

Great, we have a simple function that can group our list of strings by their first character.  Unfortunately this isn't very generic and if we wanted a different type of grouping this would not work.  Let's try another test to help us make this more usable.

``` java
private static Person Rod = aPerson("Rod", 50);
private static Person Jane = aPerson("Jane", 21);
private static Person Freddy = aPerson("Freddy", 50);

@Test
public void groupPeople_byAge() {
    List<Person> people = asList(Rod, Jane, Freddy);

    Map<Integer, Collection<Person>> groups = group(people, λ(p, p.age()));

    assertThat(groups.size(), is(2));
    assertThat(groups.keySet(), contains(21, 50));
    assertThat(groups.get(21), contains(Jane));
    assertThat(groups.get(50), contains(Rod, Freddy));
}

class Person {
    public static Person aPerson(String name, Integer age) {
        return new Person(name, age);
    }

    private Person(String name, Integer age) {
        this.name = name,
        this.age = age;
    }

    public Integer age() { return age };
}
```

Now we need a generic signature for our group function.  Here's what I came up with.

``` java
<Group, Item> Map<Group, Collection<Item>> group(Collection<Item> items, Function<Item, Group> grouping);
```

It's a shame that trying to make this readable makes a huge long signature.  I admit I could have just used `G` and `I` instead of `Group` and `Item` however I do feel this explains the usage more.

Also, I was quite pleased at how close my Java implementation came out to the original Scala.  It wasn't the intention, I just wanted to make it readable.

* Group a list of strings by their first character in Java

``` java
group(strings, λ(s, s.charAt(0));
```

* Group a list of string by their first character in Scala

``` scala
strings groupBy (_.charAt(0))
```

## Conclusion

I found the group function very useful, especially when creating reports.  I do realise that I wanted to remove iteration/duplication and the new `group()` function iterates, maybe this could be further improved with tail recursion.
