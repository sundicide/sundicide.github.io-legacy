# Week3-1: Type-Directed Programming
지금까지 봤듯이 compiler는 values로 부터 types을 유추할 수 있다.

```scala
val x = 12
```
compiler는 x를 Int로 유추한다. 왜냐하면 값이 12이므로
아래와 같이 복잡한 표현에서도 이는 적용된다.

```scala
val y = x + 3
```
compiler는 y 또한 Int로 유추한다.

<br />
이번에는 반대로 compiler가 types로 부터 values를 유추하는 과정을 볼 것이다.
왜 이것이 유용하냐? 확실한 하나는 compiler가 value를 찾아서 줄 수 있기 때문이다.

이번 레슨의 나머지는 이런 메카니즘의 motivation을 소개하고 다음 번 레슨은 how to use it을 설명할 것이다.

## Motivating Example
parameter로 List[Int]를 받아서 정렬한 결과를 List[Int]로 리턴하는 함수를 생각해보자.

```scala
def sort(xs: List[Int]): List[Int] = {
  ...
  ... if (x < y) ...
  ...
}
```
상세 코드는 여기에서 필요가 없기에 생략했다. 위 코드는 Int에 대해서만 적용 가능하므로 general하게 모든 타입에 대해서도 동작하게 하고 싶다.
이에 대한 straightforward approach는 polymorphic type을 사용하는 것이다.

```scala
def sort[A](xs: List[A]): List[A] = ...
```

하지만 이것만으로는 부족하다. 왜냐하면 각 type별로 compare를 다르게 해야 하기 때문이다.
그래서 이번엔 각 compare 함수를 parameter로 받도록 해보자.

```scala
def sort[A](xs: List[A])(lessThan: (A, A) => Boolean): List[A] = {
  ...
  ... if (lessThan(x, y)) ...
  ...
}
```

그렇게 되면 아래와 같이 가능하다'

```scala
val xs = List(-5, 6, 3, 2, 7)

val strings = List("apple", "pear", "orange", "pineapple")

sort(xs)((x, y) => x < y)

sort(strings)((s1, s2) => s1.compareTo(s2) < 0)
```

## Refactoring With Ordering
scala는 standard library 에서 comparing 하는 함수를 기본으로 제공한다.

```scala
package scala.math

trait Ordering[A] {
  def compare(a1: A, a2: A): Int
  def lt(a1: A, a2: A): Boolean = compare(a1, a2) <= 0
  ...
}
```
compare 함수는 2개의 parameter를 받아서 첫 번째 값이 클 경우 양수, 작을 경우 음수, 동일한 경우 0을 리턴한다.

이를 사용하면 아래와 같이 변경 가능하다.
```scala
def sort[A](xs: List[A])(ord: Ordering[A]): List[A] = {
  ...
  ... if (ord.lt(x, y)) ...
  ...
}
```
```scala
import scala.math.Ordering

sort(xs)(Ordering.Int)
sort(strings)(Ordering.String)
```

여기에서 사용 중인 Int와 String은 `types`이 아니고 `values`임을 알아야 한다.
scala에서는 types과 values에 동일한 symbol을 사용하는 것이 가능하다.

```scala
object Ordering {
  val Int = new Ordering[Int] {
    def compare(x: Int, y: Int) = if (x > y) 1 else if (x < y) -1 else 0
  }
}
```

## Reducing Boilerplate
지금까지 정의한 것을 따르면 잘 동작한다.
하지만 모든 경우에 대해 boilerplate가 존재하게 된다. Int를 비교할 때마다 `Ordering.Int`를 반복적으로 사용해야 한다.

```scala
sort(xs)(Ordering.Int)
sort(ys)(Ordering.Int)
sort(strings)(Ordering.String)
```

## Implicit Parameters

`implicit`을 명시함으로서 compiler가 argument `ord`를 support를 하게 할 수 있다.
```scala
def sort[A](xs: List[A])(implicit ord: Ordering[A]): List[A] = ...
```

```scala
sort(xs)
sort(ys)
sort(strings)
```

위와 같이 하면 컴파일러가 value에 맞춰 type을 결정한다.

컴파일러가 수행하는 과정을 자세히 살펴보자.
```scala
sort(xs)
```

xs 가 List[Int] 타입이므로 컴파일러는 위의 코드를 아래와 같이 변환한다.
```scala
sort[Int](xs)
```

그리고 컴파일러는 candidate definition중 Ordering[Int] 타입에 맞는 것을 찾는다. 위의 케이스에서는 Ordering.Int와 only matching되고 컴파일러는 method sort로 이를 전달한다.

```scala
sort[Int](xs)(Ordering.Int)
```

candidate values가 어떻게 정의도어있는 지를 살펴 보기 전에 implicit 키워드에 대해 자세히 알아보자.

1. method는 오직 하나의 implicit parameter list를 가질 수 있으며 이는 마지막 paramter가 되야 한다.
1. At call site, the arguments of the given clause are usually left out, although it is possible to explicitly pass them:
```scala
// Argument inferred by the compiler
sort(xs)

// Explicit argument
sort(xs)(Ordering.Int.reverse)
```

## Candidates for Implicit Parameters
컴파일러가 type T에 대해 어떤 candidate definition를 찾을까?
컴파일러는 아래 definition을 찾는다.

- have type T,
- are marked implicit,
- are visible at the point of the function call, or are defined in a companion object associated with T.

most specific한 정의를 찾게 되면 그것을 사용하고 없다면 error를 report한다.

### implicit Definition
implicit definition이란 implicit 키워드와 함께 정의된 것을 말한다.

```scala
object Ordering {
  implicit val Int: Ordering[Int] = ...
}
```
위의 코드는 Int라는 이름을 가진 Ordering[Int] 타입의 implicit value를 정의한 것이다.

Any val, lazy val, def, or object definition can be marked implicit.

마지막으로 implicit definitions는 type parameters와 implicit parameters를 가질 수 있다.

```scala
implicit def orderingPair[A, B](implicit
  orderingA: Ordering[A],
  orderingB: Ordering[B]
): Ordering[(A, B)] = ...
```

### Implicit Search Scope
type T의 implicit value를 찾기 위해 첫 번째로 visible(inherited, imported, or defined in an enclosing scope)한 모든 implicit definitions를 찾는다.

만약 컴파일러가 lexcial scope에서 implicit instance와 매칭되는 type T를 찾지 못하면, T와 관련된 companion objects에서 이어서 찾는다. (companion objects와 types는 other types와 연관있다.)

A companion object is an object that has the same name as a type. 예로 object scala.math.Ordering is the companion of the type scala.math.Ordering.

The types associated with a type T are:

- if T has parent types T₁ with T₂ ... with Tₙ, the union of the parts of T₁, ... Tₙ as well as T itself,
- if T is a parameterized type S[T₁, T₂, ..., Tₙ], the union of the parts of S and T₁, ..., Tₙ,
- otherwise, just T itself.

As an example, consider the following type hierarchy:

```scala
trait Foo[A]
trait Bar[A] extends Foo[A]
trait Baz[A] extends Bar[A]
trait X
trait Y extends X
```

만약 Bar[Y] 타입의 implicit value가 필요하다면 compiler는 아래와 같은 companion object에서 implicit definition을 찾을 것이다.

- Bar, because it is a part of Bar[Y],
- Y, because it is a part of Bar[Y],
- Foo, because it is a parent type of Bar,
- and X, because it is a parent type of Y.
- However, the Baz companion object will not be visited.

### Implicit Search Process
search process는 no candidate found 혹은 매칭되는 최소한 하나의 candidate를 결과를 만들어 낸다.

만약 no no available implicit definition matching 이라면 에러가 repot 된다.

```scala
scala> def f(implicit n: Int) = ()

scala> f
       ^
error: could not find implicit value for parameter n: Int
```

반대로 둘 이상의 implicit definition이 eligibale 하다면 ambiguity가 report 된다.

```scala
scala> implicit val x: Int = 0

scala> implicit val y: Int = 1

scala> def f(implicit n: Int) = ()

scala> f
       ^
error: ambiguous implicit values:
  both value x of type => Int
```

same type에 매칭되는 several implicit definitions가 있어도 하나를 특정 할 수 있다면 괜찮다.

A definition a: A is more specific than a definition b: B if:

- type A has more “fixed” parts,
- or, a is defined in a class or object which is a subclass of the class defining b.

Let’s see a few examples of priorities at work.

Which implicit definition matches the Int implicit parameter when the following method f is called?

```scala
implicit def universal[A]: A = ???
implicit def int: Int = ???
def f(implicit n: Int) = ()
f
```

위의 경우에서 universal은 type paramter를 지니고 int는 아니기에, int가 more fixed parts를 갖고 이는 universal보다 먼저 고려된다. 그렇기 때문에 컴파일러가 int를 선택함에 있어 ambiguity가 없다.

아래와 같이 있을 때 implicit Int 파라미터를 갖는 f method는 어느 implicit definition에 매치 될까?

```scala
trait A {
  implicit val x: Int = 0
}
trait B extends A {
  implicit val y: Int = 1
  def f(implicit n: Int) = ()
  f
}
```

y가 A를 extend하는 trait이므로 y가 A보다 more specific 하다. 그러므로 컴파일러가 y를 선택하는 것에 ambiguity는 없다.


## Context Bounds
Syntactic sugar allows the omission of the implicit parameter list:

```scala
def printSorted[A: Ordering](as: List[A]): Unit = {
  println(sort(as))
}
```

Type parameter A has one context bound: Ordering. This is equivalent to writing:

```scala
def printSorted[A](as: List[A])(implicit ev1: Ordering[A]): Unit = {
  println(sort(as))
}
```

More generally, a method definition such as:

```scala
def f[A: U₁ ... : Uₙ](ps): R = ...
```

Is expanded to:
```scala
def f[A](ps)(implicit ev₁: U₁[A], ..., evₙ: Uₙ[A]): R = ...
```


## Implicit Query
At any point in a program, one can query an implicit value of a given type by calling the implicitly operation:

```scala
scala> implicitly[Ordering[Int]]
res0: Ordering[Int] = scala.math.Ordering$Int$@73564ab0
```

Note that implicitly is not a special keyword, it is defined as a library operation:

```scala
def implicitly[A](implicit value: A): A = value
```


## Summary
In this lesson we have introduced the concept of type-directed programming, a language mechanism that infers values from types.

There has to be a unique (most specific) implicit definition matching the queried type for it to be selected by the compiler.

Implicit values are searched in the enclosing lexical scope (imports, parameters, inherited members) as well as in the implicit scope of the queried type.

The implicit scope of type is made of implicit values defined in companion objects of types associated with the queried type.