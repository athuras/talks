## Functional Project Layout

[@athuras](https://www.twitter.com/athuras)



## Quality


The extent to which our systems do what we think they should do without paying some terrible price.



### What is "Bad" Software?


- Complex
- Brittle
- Expensive
- Incorrect
- Non Deterministic
- Sneaky



### What is "Good" Software?


- Simple
- Robust
- Inexpensive
- Correct
- Deterministic



### Organisation


How are software systems organised?



## How is "Good" software Organised?


```noformat
src
|-- main
|   |-- java
|   |-- resources
|   |-- scala
|   `-- thrift
`-- test
    |-- resources
    `-- scala
```


lol


### The Prescription


Functions.



## FPL

This is "new age" project layout the attempts to solve the following problems:


- Interface design is hard
- Separating concerns is hard
- Testing is hard
- Examples are bad
- Exposed implementations are pathologically evil



## Lets Build a Currency Exchange.

Problem: Currency conversion is hard, frequently wrong, and errors are expensive.



### Enter: The Module

In the simplest case, a module is a scope that exposes:
1. Pure Data structures
2. Algorithms


Simple Project Structure
```noformat
forex
|--- BUILD
|--- package.scala
|--- Exchange.scala
`--- model
     |--- BUILD
     |--- Currency.scala
     |--- CurrencyPair.scala
     `--- ExchangeRate.scala
```



## package objects

Package objects define scope that is available throughout the module.
They can also help define external interfaces.


```scala
package object forex {
  /**
   * A Currency exchange is a function from a pair
   * of currencies, to a result rate. How that rate
   * is found, or whether it is, is implementation
   * dependent.
   *
   * @tparam F - The resulting context, i.e. Option, Future, etc.
   **/
  type Exchange[F[_]] =
    model.CurrencyPair => F[model.ExchangeRate]
}
```


Factories are the ONLY way to use or declare implementations.

```scala
package forex

object Exchange {
  def apply(ds: DS.FutureIface): Exchange[Future] =
    new DSExchangeImpl(ds)

  private[forex] class DSExchangeImpl(
    underlying: RevenueDataservice.FutureIface
  ) extends Exchange[Future] {

    override def apply(
      pair: model.CurrencyPair
    ): Future[model.ExchangeRate] = ???
  }
}
```



## Keeping data structures safe

I like to park them in something like "model", but this can also be stored in thrift etc.


```scala
package model

sealed class Currency private[model](val iso4217Code: String)

object Currency {
  case object USD extends Currency("USD")
  case object CAD extends Currency("CAD")
  ...
}
```



### Lawful Testing


Tendencies that are observed, but not possible to _prove_


Desirable _properties_ of all implementations



### FPL Test Layout

```noformat
forex-test
|--- BUILD
|--- ExchangeLaws.scala
`--- model
     |--- BUILD
     `--- Generators.scala
```



### Exchange Laws


The Unit Law.
```scala
package forex

object ExchangeLaws {
  import model.Generators._

  def unitLaw[F[_]: Comonad](exchange: Exchange[F]): Prop =
    Prop.forAll { pair: CurrencyPair =>
      val result = Comonad.extract(
        exchange(pair.copy(right=pair.left))
      )
      result == ExchangeRate(1f)
    }
```


The Inverse Law

```scala
  def inverseLaw[F[_]: Applicative: Comonad](
    exchange: Exchange[F]
  ): Prop =
    Prop.forAll { case CurrencyPair(l, r) =>
      val forward = exchange(CurrencyPair(l, r))
      val reverse = exchange(CurrencyPair(r, l))
      val (x, y) = Comonad.extract(
        Applicative.join(forward, reverse)
      )
      x.rate == (1 / y.rate)
    }
```



### Generators


We need to describe the complete space of objects to test, computers are good at this.


```scala
package forex.model

object Generators {
  implicit val ArbCurrency = Arbitrary {
    Gen.pick(1, Seq[Currency](USD, CAD, ...))
  }

  implicit val ArbExchangeRate = Arbitrary {
    Gen.posNum[Float].map { n => ExchangeRate(n) }
  }

  implicit val ArbCurrencyPair = Arbitrary {
    for {
      l <- ArbCurrency.arbitrary
      r <- ArbCurrency.arbitrary
    } yield CurrencyPair(l, r)
  }
}
```



## Questions?
