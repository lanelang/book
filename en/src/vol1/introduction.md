# Introduction to the Lane Language

Lane is a functional programming language.

Unlike many industrial programming languages, Lane never promises to run correctly on every common platform, and it never promises backward compatibility. The Lane language and the lane tool are open source under the MIT license, so anyone may use, modify, and even commercialize them. However, every design decision in Lane and every implementation decision in the lane tool are dictated by the original author, MilkyNatas. Lane is something I make for fun, so I am not here to pamper users.

## Getting Started

You can use Lane by installing the lane tool, or you can try it in the official [playground](https://lanelang.github.io/playground).

The traditional first step in learning programming or a programming language is to write a Hello World program, which prints `hello, world`. First, write the following text in a local file or in the playground editor:

```
module Hello

import Stdlib.Io.*

pub fn hello() -> Unit ! Io {
  println!("hello, world")
}
```

How to run this program depends on the context. In a Unix environment with the lane tool installed, suppose we save the text above as `hello.lane`. The content of that file is the program's source code. Run the following command:

```
lane run hello.lane:hello --lib-dir $LANE_STD
```

This runs the program and prints the following output in the terminal:

```
hello, world
```

The command entered in the terminal also shows that we need to pass the `$LANE_STD` environment variable, which points to the standard library directory. In general, installing the toolchain also installs the standard library to a default path and sets the `$LANE_STD` environment variable to that path. However, "standard library" is only a convention in Lane, not a real standard. Therefore, it is normal to fail to find the correct `$LANE_STD` or the standard library files. To avoid that, we can write another file named `io.lane` in the current directory:

```
module Stdlib.Io

pub effect Io {
  println(String) -> Unit
}
```

Then run:

```
lane run hello.lane:hello --lib io.lane
```

This prints normally.

By comparison, completing this task in the playground is much easier. The web page includes several necessary standard library files, including the `io.lane` file we need. Ideally, after the hello program has been entered, the output box will show:

```
hello, world
```

It is worth explaining the command and program text we used.

- Line 1 declares a module named `Hello`. The first statement in any Lane file must be a module name declaration, and each file can declare only one module.
- Line 3 imports another module. The wildcard `*` imports every name under the `Stdlib.Io` module. Here, we use only one name: `println`.
- Lines 5-7 define a function.
  - At the beginning of line 5, `pub` declares the function's visibility. It means the function can be accessed externally, for example by the `lane run` command or by other modules.
  - `fn` is the keyword for declaring a function, and `hello` after it is the function's name.
  - The parentheses after the function name are the function's parameter list. The `hello` function has no parameters, so the parentheses are empty.
  - After the arrow `->` is the function's return type. `Unit ! Io` means the function returns a value of type `Unit` and produces an effect of type `Io`.
- The function body on line 6 is enclosed in braces, and it contains only one expression. That expression uses the `println` operation to produce an `Io` effect.
- The part enclosed in double quotes `"` is a string.

The concepts that appear here, including modules, strings, functions, visibility, types, and effects, are explained in detail in the main chapters.

Now look at the command we ran. `lane run` is the main entry point for running a Lane file. It takes a positional argument that specifies the function to run and the file that contains it. In our example, we run the `hello` function in `hello.lane`. It is easy to imagine defining multiple functions in one file and running whichever one is needed.

`--lib <file>` adds another Lane source file as a library, while `--lib-dir <path>` adds every Lane file under a directory as a library. Because we passed one of these options, `hello.lane` can import the `Stdlib.Io` module defined in another file.

## Bindings, Integers, Functions, and Builtin Functions

With Lane's builtin integer type and builtin functions, we can use Lane to do some simple computation. For example, we can compute the sum of two integers:

```
// integer.lane
module Integer

let add : (Int, Int) -> Int = builtin("%i64_add")

let sum : Int = add(1, 2)

import Stdlib.Io.*

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

Lane's `Int` type is a 64-bit integer, which means its range is from -2^63 to 2^63-1. In general, we will not need numbers that large. Besides `Int`, Lane has several other builtin types: `Bool`, `String`, and `Unit`.

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
