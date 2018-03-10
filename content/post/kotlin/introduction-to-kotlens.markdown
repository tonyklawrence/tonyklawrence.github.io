---
date: "2017-01-31T20:13:09Z"
title: "An Introduction to Kotlens"

categories: [kotlin, kotlens, optics, lens, functional_programming]
keywords: "kotlin, kotlens, optics, lens, monocle, functional programming"

draft: true
---

something about category theory, isomorphism, monocle scala library and fun stuff.  something about category theory, isomorphism, monocle scala library and fun stuff.  something about category theory, isomorphism, monocle scala library and fun stuff.  something about category theory, isomorphism, monocle scala library and fun stuff.  something about category theory, isomorphism, monocle scala library and fun stuff.  something about category theory, isomorphism, monocle scala library and fun stuff.  something about category theory, isomorphism, monocle scala library and fun stuff.
something about category theory, isomorphism, monocle scala library and fun stuff.

explain isomorphisms and their uses, little bit of category theory (maybe not).  Can then refer to Monocle and show the Scala interface.  We can then show the kotlin interface and then some example uses.

Have nice image that shows the iso a->s->B

* convert from a -> b // ƒ → ⇒ ⇨ ⇰ ⇾ 
* modify on the fly
* compose multiple Isos (such as measurement example)

https://github.com/julien-truffaut/Monocle

## Isomorphism

``` kotlin
data class Iso<S, A>(val get: (S) -> A, val reverseGet: (A) -> S) {
  fun modify(f: (A) -> A): (S) -> S = { s -> reverseGet(f(get(s))) }
  fun reverse(): Iso<A, S> = Iso(reverseGet, get)
  infix fun <B> compose(other: Iso<A, B>): Iso<S, B> = Iso(
    get = { s -> other.get(get(s)) },
    reverseGet = { b -> reverseGet(other.reverseGet(b)) }
  )
}
```
