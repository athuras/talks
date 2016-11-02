## Implicit Programming in Scala

[@athuras](https://www.twitter.com/athuras)


Take a deep breath...

```scala
def implicitly[T](implicit t: T): T = t
```



## Motivation

1. Code is dangerous.
2. Type Systems prevent bugs before they happen.
3. Compilers don't make mistakes.


This talk is intended to demystify implicit programming, and provide vectors for curiosity.




## Let's talk about Polymorphism

1. Subtype Polymorphism
2. Parametric Polymorphism
3. Ad-Hoc Polymorphism



## Subtype Polymorphism

Relies on the [Liskov Substitution Principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle)

> If A is a subtype of B, then objects of type A may be substituted for those of B


### Example
```scala
class Foo() {
  def sayHi: Unit = println("Foo says hi!")
}

class Bar() extends Foo {
  override def sayHi = println("Bar says hi!")
}

def sayHi(foo: Foo): Unit = foo.sayHi

sayHi(new Bar())
"Bar says hi!"
```



## Parametric Polymorphism

a.k.a. "Generics" in inferior languages.


### Example
```scala
// Contains things.
case class Container[T](value: T)

Container(1)
"Container[Int](1)"

Container("Power Overwhelming!")
"Container[String](Power Overwhelming!)"
```


We can exercise a little more control by using type bounds.
```scala
class Baz() extends Bar {
  override def sayHi = println("Baz says hi!")
}

def sayHi[T <: Bar](atMostBar: T): Unit = atMostBar.sayHi

sayHi(new Bar)
"Bar says hi!"
sayHi(new Foo)
error: inferred type arguments [Foo] do not conform to method
sayHi's type parameter bounds [T <: Bar]
...
```



## Ad Hoc Polymorphism

In Scala we have two choices.
1. Overloading a.k.a. "Bad"
2. Context Bounds a.k.a. "Good"



Bad Polymorphism (via overloaded methods).

```scala
object Print {
  def apply(integer: Int): Unit = println(integer.toString)
  def apply(foo: Foo): Unit = foo.sayHi
}

Print(1)
"1"
Print(List(1, 2, 3))
error: overloaded method value apply with alternatives;
  (integer: Int)Unit  and
  (foo: Foo)Unit
cannot be applied to (List[Int])
  Print(List(1, 2, 3))
        ^
```


### The problems with Overloading.

1. Doesn't play well with inheritance.
2. Doesn't play well with default arguments.
3. Makes [&eta;-reduction]("https://en.wikipedia.org/wiki/Lambda_calculus#Reduction") difficult.
4. Inflexible.
5. Opaque (compiler-generated method names)
6. Annoying.


We can do better.



### Segue: User Defined Behaviour

In certain cases, we can _inject_ a contextual argument that makes our implementations more flexible.
```java
public static class Collections {
  ...
  static <T> void sort(List<T> list, Comparator<? super T> c)
}
```
```scala
object Collections {
  def sort[T](list: mutable.Seq[T], ord: Ordering[T]): Unit
}
```

This is a necessary evil in Java, because the type system sucks.


There are exactly two reasons why we'd do this in scala:
1. Because we don't know any better.
2. To tell the users of our library that we don't value their time.



### Enter Context Bounds

```scala
def sort[T : Ordering](ts: Seq[T]): Seq[T] =
  ts.sorted

// Which is (effectively) syntactically equivalent to:
def sort[T](ts: Seq[T])(implicit ord: Ordering[T]): Seq[T] =
  ts.sorted
```


The next section of this talk will focus on understanding how this works.



## Implicit Search

```scala
def implicitly[T](implicit t: T): T = t
```


1. First look in current scope:
 - implicit definitions
 - explicit imports (import Ordering.Long )
 - wildcard imports (import Ordering.\_)

2. Look Elsewhere:
 - Companion object of desired type.
 - implicit scope of arguments type.
 - outer objects for nested types (i.e. embedded classes).

[This is well explained here]("http://stackoverflow.com/questions/5598085/where-does-scala-look-for-implicits") and all uses of implicits rely on clear, deliberate application of these principles.


## Scope. Is. Everything.
The easiest way to remember this is with a tattoo.

WARNING: Esoteric - [Confluence and Coherence in Scala]("http://blog.ezyang.com/2014/07/type-classes-confluence-coherence-global-uniqueness/")



The rest of this talk will be about using implicit scope for personal gain.



## Introducing 'Show'

```scala
/** First-class `ToString`. **/
trait Show[T] { def show(t: T): String }

object Show {
  def apply[T : Show](t: T): String =
    implicitly[Show[T]].show(t)

  // Lets define some primitive instances for Int, and String
  implicit val IntShow = new Show[Int] {
    def show(t: Int) = t.toString
  }

  implicit val StringShow = new Show[String] {
    def show(t: String) = t
  }
}
```


And here is its use:
```
Show(1)
"1"
Show("Foo")
"Foo"
Show(List(1))

error: could not find implicit value for evidence
parameter of type Show[List[Int]]
```
Hmmmmm...


Superficially, we have two solutions.
1. Force the user to "deal with it".
2. Provide `Show` instances for all types (as in `java`).


### 1. Write a bad library
```scala
object DealWithIt {
  // *claps hands*, well _that_ was easy!
  def show[T](t: T, s: Show[T]): String = s.show(t)
}
```


### 2. Boilerplate our way out

If this was java, we'd just implement every instance of `Show`.

```scala
implicit val ListIntShow = new Show[Seq[Int] {
  def show(t: Seq[Int]) = t match {
    case Seq() => "Empty"
    case Seq(a, _@_*) => s"Seq($a, ...)"
  }
}

implicit val ListStringShow = ...
implicit val ListFloatShow = ...
```
This is bad idea for a variety of reasons.


### 3. Use Implicit Resolution ... Again!
```scala
implicit def seqShow[T : Show]: Show[Seq[T]] = new Show[Seq[T]] {
  private[this] val elementShow = implicitly[Show[T]]
  def show(t: Seq[T]) = t match {
    case Seq() => "Empty"
    case Seq(x, _@_*) => s"Seq(${elementShow.show(x)}, ...)"
  }
}

Show(Seq(1, 2, 3))
"Seq(1, ...)"
```

This also works recursively!
```
Show(Seq(Seq(Seq(1, 2, 3))))
"Seq(Seq(Seq(1, ...), ...), ...)"
```



# Scope Is Everything
