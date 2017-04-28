## Aggressive Functional Programming

[@athuras](https://www.twitter.com/athuras)



### Q: What is Functional Programming?


1. Esoteric?
2. Using lots of parentheses?
3. Insanely generic?


A: Programming with Functions



### Q: What is Aggressive Functional Programming?


A: *Aggressively* programming with functions.



### FP patterns aim to achieve two things:

1. Separation of Data, and Behaviour (algorithms).
2. Ensure it is impossible to represent an illegal state.


#### Example: Separating Data, and Behavior
```scala
sealed trait Chain

final case class Link(next: Chain) extends Chain

case object End extends Chain

@tailrec def length(l: Chain, acc: Int = 0): Int = l match {
  case Link(n) => length(n, acc + 1)
  case End     => acc
}

// length(Link(Link(Link(End)))) -> 3
```


#### Example: Illegal States
```java
public class PositiveInteger {
  private int _value;

  public PositiveInteger(final int value) {
    // WHAT DID YOU EXPECT!?!
    if (value < 0) System.exit(1);
    _value = value;
  }
  public int getValue() { return _value; }
}
```



## Brief Interlude
NOTE: Analogy of writing this slideshow, and learning FP



## Regularly Scheduled Program
1. Mapping from SOLID to FP.
2. An Overview of Esoteric Nomenclature
3. The M-Word
5. Writing Higher-Kinded Interfaces
6. Property Based Testing



## S.O.L.I.D
The Laws of software design trivially map to FP


### Single Responsibility Principle
Only one potential change in the application should affect the specification of a *thing*.

Functions <!-- .element: class="fragment" -->


### Open/Closed Principle
*Things* should be open for extension, but closed for modification.

Functions <!-- .element: class="fragment" -->


### Liskov Substitution Principle
*Things* should be replaceable with instances of their subtypes without altering the correctness of the program.

Functions <!-- .element: class="fragment" -->


### Interface Substitution Principle
Many client-specific interfaces are better than one general-purpose interface.

Functions <!-- .element: class="fragment" -->


### Dependency Inversion Principle
One should depend upon abstractions, not concrete implementations.

Functions <!-- .element: class="fragment" -->



## Overview of Esoteric Nomenclature


| Term | Description |
| ------- | ------------|
|[Function](https://en.wikipedia.org/wiki/Function_%28mathematics%29)| Maps one type, to another. |
|[Type Class](https://en.wikipedia.org/wiki/Type_class) | Generic Interface. |
|[Kind](https://en.wikipedia.org/wiki/Kind_%28type_theory%29) | The type of a Type. What happens when types level-up.|
|[Category](https://en.wikipedia.org/wiki/Category_theory)| A math graph; hard to visualise.|
|[Functor](https://en.wikipedia.org/wiki/Functor)| Read the docs |
|[Monad](https://en.wikipedia.org/wiki/Monad_%28category_theory%29)| When used correctly people will get off your lawn.|



## Examples


### Functions
```scala
val f: Int => Int = _ + 1
```

```java
Function<Integer, Integer> f = (int x -> x + 1)
```

```python
f = lambda x: x + 1
```

```haskell
f :: Integer -> Integer
f x = x + 1
```


### Type Classes
```scala
trait Show[T] { def show(t: T): String }
```

```java
public interface Show<T> {
  public String show(final T t);
}
```

```python
def show(t): return str(t)
```

```haskell
class Show t where
  show :: t -> String
```


### Kind(s)
```scala
import scala.language.higherKinds
type Type // 0-th order kind. a regular type
type Constructor[T] // 1-st order kind.
type Factory[F[_]] // 2-nd ...
type Industry[F[_[_]]] // yes, this is a thing.
```

```java
// Doesn't work in java.
```

```python
# No idea how this would be done in python. Meta-classes?, probably meta-classes.
```

```haskell
*
* -> *
* -> * -> *
```


### Categories
```scala
CanBuildFrom[-From, -Elem, +To] // A base trait for builder factories.
Functor[F[_]] // Functors between categories form a category.
```


## The M-Word
```scala
// Allows us to apply a function within a context `F`.
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}

// Allows us to wrap a value in a context `A`.
trait Applicative[A[_]] extends Functor[A] {
  def apply[X](a: X): A[X]
  def ap[X, Y](ax: A[X])(f: A[X => Y]): A[Y]
}

// Allows us to 'merge' nested contexts together.
trait Monad[M[_]] extends Applicative[M] {
  def flatMap[A, B](ma: M[A])(f: A => M[B]): M[B]
  def join[A](mma: M[M[A]]): M[A]
}
```



## Writing Higher-kinded Interfaces
The ultimate abstraction:
```scala
final case class Scored[T](item: T, score: Int)
type Scorer[T, F[_]] = T => F[Scored[T]]
type Selector[T] = Set[Scored[T]]] => Option[T]

type Auctioneer[T, F[_]] = Seq[T] => F[Option[T]]
object Auctioneer {
  def apply[T, F[_] : Applicative](
    scorer: Scorer[T, F],
    selector: Selector[T]
  ): Auctioneer[T, F] = { ts =>
    val scored: Seq[F[Scored[T]]] = ts.map(scorer)(breakOut)
    Applicative.sequence(scored).map(selector)
  }
}
```


### Annoyances
The above code doesn't work, but instead forces us into using 'package' objects to export/use the types.
This is either a bug or a feature; but forces a slightly different (header-oriented) style:
```scala
package object awesome {
  type Scorer[T, F[_]] = T => F[Scored[T]]
  type Selector[T] = Set[Scored[T]] => Option[T]
  type Auctioneer[T, F[_]] = Set[T] => F[Option[T]]
}
```



## Property Based Testing

Lets imagine we have an injection type class:
```scala
// LAW:
//   reverse.compose(forward)(x) must be Some(x) forAll x in A.
final case class Injection[A, B](
  forward: A => B,
  reverse: B => Option[A]
)
```


Seems legit ...
```scala
val IntMagic = Injection[Int, Int](x => x * 4, x => Some(x / 4))
```


We could even test it:
```scala
// useless test that I see everywhere....
"IntMagic should obey the injection law" in {
  import IntMagic._
  reverse.compose(forward)(4) shouldBe (Some(4))
}
```


There is a terrible problem here, because Integer division is not invertible.
Lets just test all possible inputs (it is *so very easy*).
```scala
import org.scalacheck.Arbitrary._

"IntMagic should obey the injection law" in {
  import IntMagic._
  val roundTrip = reverse.compose(forward)
  forAll { x: Int =>
    roundTrip(x) shouldBe (Some(x))
  }
}
```


We can/should abstract this Law into a generic Property test!
```scala
import org.scalacheck.{ Arbitrary, Prop }

object InjectionLaw {
  // This is SO EASY IT HURTS, seriously PLEASE DO THIS
  def apply[A : Arbitrary, B](inj: Injection[A, B]): Prop = {
    import inj._
    val roundTrip = reverse.compose(forward)
    Prop.forAll { a: A => roundTrip(a) == Some(a) }
  }
}

// Elsewhere!
"IntMagic should obey the injection law" in {
  InjectionLaw(IntMagic)
}
```


## In Summary
1. Everything I've said is unabiguously correct
2. Type away your problems (get it?!)
3. Properties are easy to test, so please do.



## Further Reading

[The Type Classes of Cats](http://typelevel.org/cats/typeclasses.html)

[Algebraic Patterns](https://philipnilsson.github.io/Algebraic-patterns-04-2017-slides/#/)

[Functional Programming is Terrible](https://www.youtube.com/watch?v=hzf3hTUKk8U)

[System F](https://en.wikipedia.org/wiki/System_F)

[My Other Talk on Implicits](https://github.com/athuras/talks/blob/master/implicits.markdown)

[Reveal.js](https://github.com/hakimel/reveal.js)
