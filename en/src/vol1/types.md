# Types

Types are an extremely important concept in the Lane programming language. The organization, underlying representation, and behavioral abstraction of data all rely on types.

## Builtin Types

Lane provides several important builtin types:

```
Int
Double
String
Unit
Bool
Void
```

Their definitions are not exposed to programmers. However, we can roughly understand `Void`, `Bool`, and `Unit` as having definitions like these:

```
enum Void {}
enum Bool {
  true()
  false()
}
enum Unit {
  unit()
}
```

`Void` cannot be constructed; `Unit` always has exactly one value, `unit()`; and `Bool` has two values, `true()` and `false()`.

In actual Lane programs, there is no need to import the definitions of these types. The actual forms of `unit()`, `true()`, and `false()` are `()`, `true`, and `false`, respectively.

The integer type `Int` is a 64-bit integer, covering every integer from `-(2^63)` to `2^63 - 1`.

The floating-point type `Double` is a 64-bit floating-point number conforming to the IEEE 754 standard. It includes positive infinity, `double_inf`; negative infinity, `double_neg_inf`; and not-a-number, `double_nan`.

Lane's string type is a sequence of ASCII codes.

## Function Types

Function types use arrows: `(A) -> B` denotes a function that accepts one parameter of type `A` and returns a value of type `B`. For example:

```
fn f(x : Int) -> Int {
  x * 2
}
```

Its type is:

```
f : (Int) -> Int
```

For a function `f` of type `(A) -> B` and a value `x` of type `A`, the expression `f(x)` always has type `B`.

The number of parameters of a function is called its arity. Here is an example of a binary function:

```
fn f(x : Int, y : Int) -> Int {
  x + y
}
```

A function can also have arity zero, for example:

```
fn f() -> Int {
  42
}
```

If the parameters of a function `f` include another function, then `f` is a higher-order function. If its parameters do not include functions, then it is a first-order function. Ordinary values can be regarded as zeroth-order. If the highest-order parameter of `f` is of order `n`, then `f` is an `n + 1`-order function.

For example, the following function is second-order:

```
fn[A, B] apply(f : (A) -> B, x : A) -> B {
  f(x)
}
```

Function types can also carry type variables. For example, the type of the `apply` function above can be written as:

```
apply : [A, B]((A) -> B, A) -> B
```

This type can be read as follows: if `A` and `B` are types, `f` has type `(A) -> B`, and `x` has type `A`, then `apply[A, B](f, x)` has type `B`.

Here, type parameters can usually be inferred automatically by the compiler, so they need not always be written. Normally, `apply[A, B](f, x)` can be written simply as `apply(f, x)`.

We can also read only half of the type above: if `A` and `B` are types, then `apply[A, B]` has type `((A) -> B, A) -> B`.

This way of understanding types is essential for the existential types discussed below.

## Existential Types

We have already seen an example of a generic type: the list type `List`. It can be written as `type List : [Type] -> Type`, which reads: if `A` is a type, then `List[A]` is also a type. In other words, `List` itself is not a type.

We can define a simplest generic type, `Box`, which stores only one value:

```
enum Box[T] {
  box(T)
}
```

We can regard `box` as having type `[T](T) -> Box[T]`. This common kind of polymorphic type is called a universal type. But sometimes we need a box, `Hide`, that can store any content without reflecting the type of that content in the type of the box itself. In other words, we need a constructor of type `[T](T) -> Hide` and the corresponding definition of the type `Hide`. In Lane, we can write this as follows:

```
enum Hide {
  hide[T](T)
}
```

Notice that the type variable `T` is no longer introduced by the enum declaration; it is introduced by the constructor `hide`. In this example, `Hide` itself is a type, namely `Hide : Type`; while `hide` needs a type parameter and has type `[T](T) -> Hide`. Its type has the same left-hand side as the type of `box` and differs only on the right-hand side, but the difference is fundamental.

How can we use such a type? We can use pattern matching, or a binding specifically for irrefutable patterns:

```
let v : Hide = hide[Int](10)
...
let hide[T](x) = v
// or
match v {
  hide[T](x) => ...
}
```

Note, however, that because `v` has type `Hide`, using it does not tell us that it stores an integer. We know only that after unpacking it, `x` has type `T`. Therefore, neither the value `x` nor the type `T` can escape its scope. For example, the following program is illegal:

```
fn[T] escape(v : Hide) -> T {
  let hide[T](x) = v
  x
}
```

This is because unpacking `v` essentially matches the hidden type to the type variable `T`. This `T` shadows the type variable `T` introduced by the function's type parameter in the surrounding context. Thus, the program above is no different from the following program:

```
fn[R] escape(v : Hide) -> R {
  let hide[T](x) = v
  x
}
```

This makes the problem clear: `x` has type `T`, while the function must return `R`. There is clearly no evidence that these two types are the same, so the function cannot compile.

## Structs as Interfaces
