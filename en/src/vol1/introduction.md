# Introduction to the Lane Language

Lane is a functional programming language.

Unlike many industrial programming languages, Lane never promises to run correctly on every common platform, and it never promises backward compatibility. The Lane language and the lane tool are open source under the MIT license, so anyone may use, modify, and even commercialize them. However, every design decision in Lane and every implementation decision in the lane tool are dictated by the original author, MilkyNatas. Lane is something I make for fun, so I am not here to pamper users.

## Getting Started

You can use Lane by installing the lane tool, or you can try it in the official [playground](https://lanelang.github.io/playground).

The traditional first step in learning programming or a programming language is to write a Hello World program, which prints `hello, world`. First, write the following text in a local file or in the playground editor:

```
module Hello

import Basic.Io.*

pub fn hello() -> Unit ! Io {
  println!("hello, world")
}
```

How to run this program depends on the context. In a Unix environment with the lane tool installed, suppose we save the text above as `hello.lane`. The content of that file is the program's source code. Run the following command:

```
lane run hello.lane:hello --lib-dir $LANE_BASIC
```

This runs the program and prints the following output in the terminal:

```
hello, world
```

The command entered in the terminal also shows that we need to pass the `$LANE_BASIC` environment variable, which points to the Basic library directory. In general, installing the toolchain may also install the Basic library to a default path and set the `$LANE_BASIC` environment variable to that path. Basic is only a convention in Lane, not a standard library or a privileged language component. Therefore, it is normal to fail to find the correct `$LANE_BASIC` or Basic library files. To avoid that, we can write another file named `io.lane` in the current directory:

```
module Basic.Io

pub effect Io {
  println(String) -> Unit
}
```

Then run:

```
lane run hello.lane:hello --lib io.lane
```

This prints normally.

By comparison, completing this task in the playground is much easier. The web page includes several necessary Basic library files, including the `io.lane` file we need. Ideally, after the hello program has been entered, the output box will show:

```
hello, world
```

It is worth explaining the command and program text we used.

- Line 1 declares a module named `Hello`. The first statement in any Lane file must be a module name declaration, and each file can declare only one module.
- Line 3 imports another module. The wildcard `*` imports every name under the `Basic.Io` module. Here, we use only one name: `println`.
- Lines 5-7 define a function.
  - At the beginning of line 5, `pub` declares the function's visibility. It means the function can be accessed externally, for example by the `lane run` command or by other modules.
  - `fn` is the keyword for declaring a function, and `hello` after it is the function's name.
  - The parentheses after the function name are the function's parameter list. The `hello` function has no parameters, so the parentheses are empty.
  - After the arrow `->` is the function's return type. `Unit ! Io` means the function returns a value of type `Unit` and produces an effect of type `Io`.
- The function body on line 6 is enclosed in braces, and it contains only one expression. That expression uses the `println` operation to produce an `Io` effect.
- The part enclosed in double quotes `"` is a string.

The concepts that appear here, including modules, strings, functions, visibility, types, and effects, are explained in detail in the main chapters.

Now look at the command we ran. `lane run` is the main entry point for running a Lane file. It takes a positional argument that specifies the function to run and the file that contains it. In our example, we run the `hello` function in `hello.lane`. It is easy to imagine defining multiple functions in one file and running whichever one is needed.

`--lib <file>` adds another Lane source file as a library, while `--lib-dir <path>` adds every Lane file under a directory as a library. Because we passed one of these options, `hello.lane` can import the `Basic.Io` module defined in another file.

## Bindings, Integers, Functions, and Builtin Functions

With Lane's builtin integer type and builtin functions, we can use Lane to do some simple computation. For example, we can compute the sum of two integers:

```
// integer.lane
module Integer

let add : (Int, Int) -> Int = builtin("%i64_add")

let sum : Int = add(1, 2)

import Basic.Io.*

pub fn print_sum() -> Unit ! Io {
  println!("sum is " + to_string(sum))
}
```

Run `lane run integer.lane:print_sum --lib io.lane`, and you should see `sum is 3` printed in the console.

The first line of this program is a comment. A comment starts with two consecutive slashes `//`, and the rest of the line is only used to explain the meaning of the program; it does not affect the program's behavior. Comments are ignored by the compiler, which in this case is what `lane run` invokes, so they may contain any text.

This time, first look at line 6. It binds the value `add(1, 2)` to the name `sum`. After we bind a value to a name, we can use that name anywhere to stand for the original value. For example, we use `sum` on line 11; if we replaced that `sum` with `add(1, 2)`, the program would produce the same result.

After the colon in a `let` binding is the type of the binding. The type of `sum` is `Int`, which means `sum` is an integer, more precisely a 64-bit integer.

The expression used to compute `sum` is the function application `add(1, 2)`. We can understand this by common sense: the result of `add(1, 2)` is the integer `3`.

Line 4 binds the builtin function `builtin("%i64_add")` to the name `add`. Here, the type of `add` is `(Int, Int) -> Int`, which means `add` is a function. Both of its parameters are integers, and its return type is also an integer. Therefore, after passing two integers as arguments to `add`, the result, which is the value of the whole expression, is also an integer.

The right-hand side of the binding on line 4 is a builtin function. For now, we do not need to care about the details of this function. We only need to know that implementing integer addition inside the Lane language itself would be very troublesome; by using a builtin function, we can hand this function to the compiler, and the compiler will implement the correct operation for us.

Lane's `Int` type is a 64-bit integer, which means its range is from -2^63 to 2^63-1. In general, we will not need numbers that large. Besides `Int`, Lane has other builtin types, such as `Bool`, `String`, and `Unit`.

We can slightly modify line 6 of the program above:

```
let sum : Bool = add(1, 2)
```

That is, we change the type of `sum` to the Boolean type `Bool`. If we run `lane run` or `lane check` again, the compiler will immediately report an error:

```
expected `Bool`, found `Int`
```

This means that we need `sum` to be a `Bool`, but the type of the right-hand side of `=` is `Int`.

Types can help us avoid many errors before the program runs. In the following chapters, we will see how Lane helps us do this.

## Local Bindings and Scope

Bindings can be written at the top level of a file, but they are more commonly written inside function definitions. These are local bindings. For example, the integer-printing program above can be written like this:

```
// scope.lane
module Scope

let add : (Int, Int) -> Int = builtin("%i64_add")

import Basic.Io.*

pub fn print_sum() -> Unit ! Io {
  let sum = add(1, 2)
  println!("sum is " + to_string(sum))
}
```

This program produces the same result as before. The difference is that local bindings can omit their types. Notice that we did not write `: Int` here; the compiler automatically infers that the type of `sum` is `Int`. This feature is very useful when a type is hard to write down.

There is another difference: now only the inside of the `print_sum` function can access `sum`. More precisely, `sum` can be accessed only after the local `let` binding on line 9 and before the function definition ends on line 11. This region is called a scope. Each local binding can be accessed only by expressions inside its scope. For a top-level binding, its scope is the whole file. For a binding marked with the `pub` visibility marker, other modules can also access it by importing the module.

Inside the scope of a binding, if a new binding with the same name appears, the new binding shadows the old one. For example:

```
pub fn print_sum() -> Unit ! Io {
  let sum = add(1, 2)
  let sum = add(3, 4)
  println!("sum is " + to_string(sum))
}
```

This program prints `sum is 7`, not `sum is 3`, because the second binding shadows the first binding. Now consider another example. In this example, we define a local function. Like a local binding, it can be accessed by later expressions inside its scope:

```
pub fn print_sum() -> Unit ! Io {
  let sum = add(1, 2)
  fn add3(a : Int, b : Int, c : Int) -> Int {
    let sum = add(a, b)
    add(sum, c)
  }
  let sum = add3(sum, sum, sum)
  println!("sum is " + to_string(sum))
}
```

This program prints `sum is 9`. The reason is that the `sum` on line 2 has the result 3; the `sum` on line 5 refers to the binding on line 4, not the shadowed binding on line 2. When `add3(sum, sum, sum)` is computed on line 7, the argument `sum` refers to the binding on line 2, because the scope of the `sum` bound on line 4 is only inside the body of `add3`, and it ends when that function body ends on line 6. Therefore, the final result is `add3(3, 3, 3) => add(add(3, 3), 3) => add(6, 3) => 9`.

Scope limits where a binding can be accessed; shadowing ensures that the meaning of a name is determined by the nearest effective binding with that name. With these two rules, we can compose code with more confidence, without worrying that some distant binding with the same name will accidentally change the meaning of the current code.

## Booleans, Enums, and Pattern Matching

In Lane, we can compare two numbers and enter different program branches based on the result of the comparison. The `print_less_one` function defined below prints the smaller of its two arguments:

```
// compare.lane
let less : (Int, Int) -> Bool = builtin("%i64_lt")

fn print_less_one(a : Int, b : Int) -> Unit ! Io {
  match less(a, b) {
    true => println!(to_string(a))
    false => println!(to_string(b))
  }
}
```

To avoid repetition, from now on we will no longer write out the module declaration and module import statements every time.

In the program above, we bind the builtin function `%i64_lt` to `less`. It compares the sizes of two integers. When `a < b`, `less(a, b)` returns the Boolean value `true`. There are only two Boolean values: `true` and `false`. The next three lines of code match on the comparison result: if the result is `true`, the program prints `a`; otherwise, it prints `b`.

For Boolean values, besides pattern matching, we can also use a conditional expression, also known as an if-then-else expression. The following function has the same behavior as the one above:

```
fn print_less_one(a : Int, b : Int) -> Unit ! Io {
  if less(a, b) {
    println!(to_string(a))
  } else {
    println!(to_string(b))
  }
}
```

If the expression after `if` evaluates to `true`, then the expression inside the first pair of braces, also called the then branch, becomes the result of the whole conditional expression; otherwise, the expression inside the second pair of braces, also called the else branch, becomes the result of the whole conditional expression.

Like `Bool`, we can also define our own types whose values can be described by listing all possible cases. For example, we can define the two sides of a coin:

```
enum Coin {
  head()
  tail()
}
```

Then we can construct the corresponding value with `TypeName::constructor()`. For example:

```
let first_coin : Coin = Coin::head()
```

When there is no ambiguity, the type name can be omitted:

```
let second_coin : Coin = tail()
```

With pattern matching, we can use these values:

```
match coin {
  Coin::head() => println!("head")
  tail() => println!("tail")
}
```

Enum values can also carry data. For example, we can define a shape type. A shape can be a circle or a rectangle. If it is a circle, we need a floating-point number `Double` to describe its radius; if it is a rectangle, we need two floating-point numbers to describe its length and width:

```
enum Shape {
  circle(Double)
  rectangle(Double, Double)
}
```

Defining values of the shape type works the same way as before: we only need to put the carried data inside the parentheses:

```
let shape1 : Shape = circle(4.0)

let shape2 : Shape = rectangle(2.0, 3.0)
```

We can write a function that computes the area of a shape:

```
fn area(shape : Shape) -> Double {
  match shape {
    circle(r) => r * r * pi
    rectangle(x, y) => x * y
  }
}
```

We will introduce the floating-point type `Double`, the multiplication sign `*`, and the constant `pi` later, but that does not stop us from understanding this program. Pattern matching enters the branch corresponding to how the value was constructed and binds the data carried by the value to the names inside the parentheses. For example, `circle(r)` binds the circle's radius to `r`, and `rectangle(x, y)` binds the rectangle's length and width to `x` and `y`.

## Structs

A struct is another kind of user-defined type, different from an enum type.

Sometimes the data type we need has only one shape, such as a point in two-dimensional space:

```
enum Point {
  point(Int, Int)
}

let p : Point = point(1, 2)
```

In this situation, it is more convenient to use a Lane struct. The syntax for defining a struct is similar to the syntax for defining an enum type, except that both the name and the type of each field must be written out. The syntax for constructing a struct literal is also similar to the syntax for constructing an enum value, except that each field name must be written out, followed by the corresponding value after a colon:

```
struct Point {
  x : Int
  y : Int
}

let p : Point = Point::{ x: 1, y: 2 }
```

Structs are used in a way similar to enum types, and their fields can also be accessed through pattern matching:

```
fn squared_distance(p : Point) -> Int {
  match p {
    Point::{ x: first, y: second } => first * first + second * second
  }
}
```

When a struct field name is the same as the name we want to bind in the pattern, we can use shorthand syntax:

```
fn squared_distance(p : Point) -> Int {
  match p {
    Point::{ x, y } => x * x + y * y
  }
}
```

Besides pattern matching, struct fields can also be accessed directly with dot notation:

```
fn squared_distance(p : Point) -> Int {
  p.x * p.x + p.y * p.y
}
```

## Lists and Generics

Lane also supports a special type and its corresponding literal syntax: lists. A list is an ordered collection of elements, and all elements must have the same type. We can use square brackets `[]` to write list literals, with elements separated by commas `,`. For example:

```
let int_list : List[Int] = [1, 2, 3]

let bool_list : List[Bool] = [true, false, true]
```

As we can see, the list type `List` itself is not a concrete type, but a generic type. `List[Int]` means a list whose element type is `Int`, and `List[Bool]` means a list whose element type is `Bool`. We can use `List[T]` to mean a list whose element type is some type `T`.

User-defined enums and structs can also be generic types like this. For example, we can define a generic binary tree type:

```
enum Tree[T] {
  leaf(T)
  node(Tree[T], Tree[T])
}
```

We can also define a simple boxed type:

```
struct Box[T] {
  value : T
}
```

We can understand this type as follows: for any type `T`, `Box[T]` represents a box whose carried value has type `T`.

Therefore, we can put values of any type into a box of type `Box[T]`:

```
let int_box : Box[Int] = Box::{ value: 42 }

let bool_box : Box[Bool] = Box::{ value: true }
```

In fact, lists are defined as a generic enum type in the `Basic.Data.List` module:

```
module Basic.Data.List

enum List[T] {
  empty()
  cons(T, List[T])
}
```

The list literal `[1, 2, 3]` written with square brackets is actually shorthand for `cons(1, cons(2, cons(3, empty())))`.

Besides structs and enums, functions can also be generic. For example, we can define a function that accepts a list and returns its length:

```
fn[T] length(list : List[T]) -> Int {
  match list {
    empty() => 0
    cons(_, tail) => 1 + length(tail)
  }
}

let int_list : List[Int] = [1, 2, 3]

fn print_length(list : List[Int]) -> Unit ! Io {
  println!("length is " + to_string(length(list)))
}
```

The program above prints `length is 3`. We use `[T]` in the function definition on line 1, which means `length` is a generic function. It can accept a list with any element type as its argument. No matter what the element type of the list is, the `length` function can correctly compute the length of the list. This lets us use the same function to compute the length of lists with different element types, without writing a new function for each type.

## Recursive Functions and Higher-Order Functions

In the program that prints the length of a list, the `length` function calls itself. Such a function is a recursive function. A recursive function is a function that directly or indirectly calls itself in its function body. Recursion is a common programming technique, especially suitable for processing data types with recursive structure, such as lists and trees.

Basically, all list-related operations are suitable for implementation with recursive functions. For example, we can define a function that computes the sum of all integers in a list:

```
fn sum(list : List[Int]) -> Int {
  match list {
    empty() => 0
    cons(head, tail) => head + sum(tail)
  }
}
```

Suppose we have a list `[1, 2, 3]`. We can observe the computation process of this function:

```
  sum([1, 2, 3])
= 1 + sum([2, 3])
= 1 + (2 + sum([3]))
= 1 + (2 + (3 + sum([])))
= 1 + (2 + (3 + 0))
= 6
```

Lists have a common operation called mapping, or map. A mapping operation applies a function to each element in a list and returns a new list; the new list contains the result of applying the function to each corresponding element in the original list. For example, we can define a function that doubles every integer in a list:

```
fn double_list(list : List[Int]) -> List[Int] {
  match list {
    empty() => empty()
    cons(head, tail) => cons(head * 2, double_list(tail))
  }
}
```

This function returns a new list where each element is twice the corresponding element in the original list. For example, `double_list([1, 2, 3])` returns `[2, 4, 6]`.

Suppose we have a function for checking whether an integer is even:

```
fn is_even(n : Int) -> Bool {
  n % 2 == 0
}
```

We can define a function that checks whether each element in a list is even:

```
fn is_even_list(list : List[Int]) -> List[Bool] {
  match list {
    empty() => empty()
    cons(head, tail) => cons(is_even(head), is_even_list(tail))
  }
}
```

`is_even_list([1, 2, 3])` returns `[false, true, false]`.

As we can see, `double_list` and `is_even_list` have very similar structures. They both use recursion and pattern matching to process lists. To avoid repeated code, we can define a more general higher-order function `map`, which accepts a function and a list as parameters and returns a new list:

```
fn[T, U] map(f : (T) -> U, list : List[T]) -> List[U] {
  match list {
    empty() => empty()
    cons(head, tail) => cons(f(head), map(f, tail))
  }
}
```

In this function, `T` and `U` are type parameters. They represent the element type of the input list and the element type of the output list, respectively. `f` is a function that accepts a parameter of type `T` and returns a result of type `U`. The `map` function applies `f` to each element in the list and returns a new list.

This ability to pass functions as arguments lets us write more general and more reusable code. We can use `map` to implement `double_list` and `is_even_list`:

```
fn is_even_list(list : List[Int]) -> List[Bool] {
  map(is_even, list)
}
```

To implement `double_list`, we can define a simple function `double` and pass it to `map`:

```
fn double(n : Int) -> Int {
  n * 2
}

fn double_list(list : List[Int]) -> List[Int] {
  map(double, list)
}
```

Of course, since the `double` function is very simple, we can also use an anonymous function directly:

```
fn double_list(list : List[Int]) -> List[Int] {
  map(fn(n : Int) -> Int { n * 2 }, list)
}
```

An anonymous function is itself an expression. It can be used directly wherever a function is needed, without first giving it a name. Usually, we use anonymous functions when we need to pass a function as an argument. This reduces the cost of naming and makes the code more concise. Of course, because an anonymous function is an ordinary expression, it can also be bound to a name for use elsewhere, or returned as a return value.
