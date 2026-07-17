# Lane 语言简介

Lane 是一门函数式编程语言。

和众多工业级语言不同，Lane 永远不保证能在所有常见平台上正常运行，也永远不保证向后兼容。Lane 语言和 lane 工具以 MIT 协议开源，任何人都可以使用、修改，甚至用于商业用途。但是，Lane 的所有设计以及 lane 工具的实现决策，都由原作者 MilkyNatas 独裁。因为 Lane 是我自己做着玩儿的，我可不惯着你们这些用户！

## 入门

Lane 语言可以通过安装 lane 工具来使用，也可以在官方 [playground](https://lanelang.github.io/playground) 网页上使用。

学习程序设计或程序设计语言的传统第一步，是编写 Hello World 程序，也就是「打印出 hello, world」。我们首先在本地文件或 playground 的编辑区域中编写如下文本：

```
module Hello

import Basic.Io.*

pub fn hello() -> Unit ! Io {
  println("hello, world")
}
```

如何运行这个程序取决于具体场景。在安装了 lane 工具的 Unix 环境中，假设我们将上述文本保存为 `hello.lane` 文件，文件内容就叫作程序的源代码。使用如下命令：

```
lane run hello.lane:hello
```

即可运行该程序，并在终端中看到如下信息：

```
hello, world
```

从刚才在终端中输入的命令也能看出，我们需要手动传入 Basic 库所在的目录。一般来说，安装工具链时会提示把环境变量 `$LANE_HOME` 设置为合适的位置，而 Basic 库默认安装在 `$LANE_HOME/basic` 路径下。Basic 是 Lane 语言的标准库，但这个库本身并没有任何特殊权限。因此，我们也可以在当前目录编写另一个文件 `io.lane`，用它临时提供所需的标准库内容，如下：

```
module Basic.Io

pub let println : (String) -> Unit ! Io = extern("println")
```

然后执行如下命令：

```
lane run hello.lane:hello --lib io.lane --no-basic
```

即可正常打印。

相比之下，在 playground 网页上完成这个任务要容易得多。网页中内置了运行示例所需的 Basic 库文件，包括我们需要用到的 `io.lane`。理想情况下，当 hello 程序的代码输入完毕后，输出框就能呈现如下信息：

```
hello, world
```

下面有必要对我们用到的命令和程序文本做一些说明。

- 代码的第 1 行声明了一个模块 `Hello`。任何 Lane 文件的第一条语句都必须是模块名称声明，并且一个文件只能声明一个模块。
- 第 3 行引入了另一个模块。通配符 `*` 表示引入 `Basic.Io` 模块下的所有名字。这里我们只用到了一个名字，也就是 `println`。
- 第 5-7 行定义了一个函数。
  - 其中，第 5 行开头声明了这个函数的可见性：`pub` 表示函数可以被外部访问，例如被 `lane run` 命令或其他模块访问。
  - `fn` 是声明函数的关键字，后面的 `hello` 是函数的名字。
  - 函数名后的括号是函数的参数列表。`hello` 函数没有参数，因此括号内是空的。
  - 箭头 `->` 后面是函数的返回值类型，`Unit ! Io` 表示函数返回一个 `Unit` 类型的结果，并声明可能产生 `Io` 效应。
- 第 6 行的函数体被大括号包裹，函数体内只有一个表达式。这个唯一的表达式调用 `println`，将字符串输出到终端。
- 用双引号 `"` 包裹的部分是字符串。

这里出现的诸多概念，包括模块、字符串、函数、可见性、类型、效应等，在正文章节都会进行详细解释。

下面再来看我们执行的命令。`lane run` 是运行 Lane 文件的主要入口，后面接一个位置参数，表示需要运行的函数以及它所处的文件。在我们的例子中，运行的是 `hello.lane` 文件中的 `hello` 函数。可以想象，我们可以在一个文件中定义多个函数，并按需执行其中某一个函数。

`--lib <file>` 用于添加作为库使用的其他 Lane 源代码文件，而 `--lib-dir <path>` 可以一次性把一个目录下的所有 Lane 文件都作为库添加进去。正因为添加了这两个参数，我们才能够在 `hello.lane` 中引入其他文件定义的 `Basic.Io` 模块。

## 绑定、整数、函数和内置函数

借助 Lane 提供的内置整数类型和内置函数，我们可以使用 Lane 做一些简单的计算。例如，我们可以计算两个整数的和：

```
// integer.lane
module Integer

import Basic.Io.*

let add : (Int, Int) -> Int = builtin("%i64_add")

let sum : Int = add(1, 2)

pub fn print_sum() -> Unit ! Io {
  println("sum is " + to_string(sum))
}
```

执行 `lane run integer.lane:print_sum --lib io.lane`，应该就能在控制台看到输出 `sum is 3`。

这个程序的第一行是注释。注释用两个连续的斜杠 `//` 开头，后续内容直到行尾，都只是用于解释程序的含义，不影响程序的行为。注释会被编译器，也就是这里使用的 `lane run` 忽略，因此可以在其中写任何内容。

这次我们先读程序的第 6 行，它将 `add(1, 2)` 这个值绑定到了名字 `sum`。当我们将一个值绑定到一个名字后，就可以用这个名字在任何地方代替原来的值。例如，我们在第 11 行就用到了 `sum`；如果把这里的 `sum` 替换为 `add(1, 2)`，程序的结果是一样的。

`let` 绑定的冒号后面是绑定的类型。`sum` 的类型是 `Int`，意思是 `sum` 是一个整数，准确地说是 64 位整数。

用于计算 `sum` 的式子是一个函数应用 `add(1, 2)`。我们可以根据常识来理解这一点：`add(1, 2)` 的结果就是整数 `3`。

程序的第 4 行将一个内置函数 `builtin("%i64_add")` 绑定到了名字 `add`。在这里，`add` 的类型是 `(Int, Int) -> Int`，表示 `add` 是一个函数。这个函数的两个参数类型都是整数，返回值类型也是整数。正因如此，给 `add` 函数传入两个整数作为参数后，得到的结果，也就是整个表达式的值，也是一个整数。

第 4 行绑定中 `=` 的右手侧是一个内置函数。我们暂时不用关心这个函数的细节，只需要知道，想要在 Lane 语言内部为整数类型实现加法运算非常麻烦；通过内置函数的方式，我们可以把这种函数交给编译器处理，由编译器为我们实现正确的运算。

Lane 的 `Int` 类型是 64 位整数，这意味着它的取值范围在 -2^63 到 2^63-1 之间。一般情况下，我们不会用到那么大的数字。除了 `Int`，Lane 还有其他内置类型，例如 `Bool`、`String` 和 `Unit`。

我们可以稍微修改上述程序的第 6 行：

```
let sum : Bool = add(1, 2)
```

也就是将 `sum` 的类型改成布尔型 `Bool`。这时再执行 `lane run` 或 `lane check`，编译器会立刻报错：

```
expected `Bool`, found `Int`
```

意思是，这里我们需要 `sum` 是一个 `Bool`，但 `=` 右手侧的类型是 `Int`。

类型可以帮助我们在程序运行前就规避很多错误。在接下来的章节中，我们会看到 Lane 如何帮助我们做到这一点。

## 局部绑定和作用域

绑定除了可以写在文件的顶层，更常见的情况是写在函数定义内部，也就是局部绑定。例如，上述打印整数的程序可以写成这样：

```
// scope.lane
module Scope

import Basic.Io.*

let add : (Int, Int) -> Int = builtin("%i64_add")

pub fn print_sum() -> Unit ! Io {
  let sum = add(1, 2)
  println("sum is " + to_string(sum))
}
```

这个程序的运行结果和之前相同。不同点在于，局部绑定可以省略绑定的类型。注意，此处我们没有写 `: Int`，编译器会自动帮我们推断出 `sum` 的类型为 `Int`。当类型不易书写时，这个功能会非常有用。

还有一个不同点：此时只有 `print_sum` 函数内部可以访问 `sum`。准确地说，`sum` 只能在第 9 行的 `let` 局部绑定之后、第 11 行函数定义结束之前被访问。这个区间叫作作用域。每个局部绑定都只能被其作用域内的表达式访问。对于顶层绑定来说，它的作用域是整个文件。对于添加了 `pub` 可见性标识的绑定，其他模块也可以通过模块引入的方式访问。

在一个绑定的作用域内，如果出现了同名的新绑定，那么新绑定会遮蔽原来的绑定。例如：

```
pub fn print_sum() -> Unit ! Io {
  let sum = add(1, 2)
  let sum = add(3, 4)
  println("sum is " + to_string(sum))
}
```

这个程序会打印 `sum is 7`，而不是 `sum is 3`，因为第二个绑定遮蔽了第一个绑定。再看一个例子。在这个例子中，我们定义了一个局部函数，它和局部绑定一样，可以被作用域内后续的表达式访问：

```
pub fn print_sum() -> Unit ! Io {
  let sum = add(1, 2)
  fn add3(a : Int, b : Int, c : Int) -> Int {
    let sum = add(a, b)
    add(sum, c)
  }
  let sum = add3(sum, sum, sum)
  println("sum is " + to_string(sum))
}
```

这个程序会打印 `sum is 9`。原因是，第 2 行的 `sum` 结果是 3；第 5 行的 `sum` 指向的是第 4 行的绑定，而不是被遮蔽的第 2 行。在第 7 行计算 `add3(sum, sum, sum)` 时，这里的参数 `sum` 指向第 2 行的绑定，因为第 4 行绑定的 `sum` 的作用域只在 `add3` 函数体内部，到第 6 行函数体结束时就结束了。因此，最终结果就是 `add3(3, 3, 3) => add(add(3, 3), 3) => add(6, 3) => 9`。

作用域限制了一个绑定可以被访问的范围；遮蔽则让同名绑定的含义始终由最近的有效绑定决定。有了这两条规则，我们就可以更放心地组合代码，而不用担心某个远处的同名绑定意外改变当前代码的语义。

## 布尔、枚举和模式匹配

在 Lane 中，我们可以对两个数进行比较，并根据比较的结果进入不同的程序分支。下面定义的 `print_less_one` 函数会打印两个参数中更小的一个：

```
// compare.lane
let less : (Int, Int) -> Bool = builtin("%i64_lt")

fn print_less_one(a : Int, b : Int) -> Unit ! Io {
  match less(a, b) {
    true => println(to_string(a))
    false => println(to_string(b))
  }
}
```

为避免重复，从现在起，我们将不再反复写出模块声明和模块引入语句。

在上述程序中，我们把内置函数 `%i64_lt` 绑定到了 `less`。它用于比较两个整数的大小。当 `a < b` 时，`less(a, b)` 会返回布尔值 `true`。布尔值只有两种，分别是 `true` 和 `false`。接下来的三行代码对比较结果进行匹配：如果结果为 `true`，就打印 `a`；否则打印 `b`。

对于布尔值，除了使用模式匹配，还可以使用条件表达式，也就是 if-then-else 表达式。下面这个函数在功能上和上面的定义相同：

```
fn print_less_one(a : Int, b : Int) -> Unit ! Io {
  if less(a, b) {
    println(to_string(a))
  } else {
    println(to_string(b))
  }
}
```

如果 `if` 后面的表达式结果是 `true`，那么第一个花括号内的表达式，也叫 then 分支，就是整个条件表达式的结果；否则，第二个花括号内的表达式，也叫 else 分支，就会成为整个条件表达式的结果。

像 `Bool` 这样可以通过列举所有取值来描述的类型，我们也可以自己定义。例如，我们可以定义硬币的正反面：

```
enum Coin {
  head()
  tail()
}
```

然后就可以用 `TypeName::constructor()` 的方式构造相应的值。例如：

```
let first_coin : Coin = Coin::head()
```

没有歧义时，可以省略前面的类型名：

```
let second_coin : Coin = tail()
```

借助模式匹配，我们可以使用这些值：

```
match coin {
  Coin::head() => println("head")
  tail() => println("tail")
}
```

枚举值也可以承载数据。例如，我们可以定义一个图形类型。一个图形可以是圆，也可以是矩形。如果是圆，我们需要用一个浮点数 `Double` 来描述它的半径；如果是矩形，我们需要用两个浮点数来描述它的长和宽：

```
enum Shape {
  circle(Double)
  rectangle(Double, Double)
}
```

定义图形类型的值和前面一样，只需要在圆括号中填入它承载的数据：

```
let shape1 : Shape = circle(4.0)

let shape2 : Shape = rectangle(2.0, 3.0)
```

我们可以写一个函数，用于求图形的面积：

```
fn area(shape : Shape) -> Double {
  match shape {
    circle(r) => r * r * pi
    rectangle(x, y) => x * y
  }
}
```

我们在后文才会介绍浮点数类型 `Double`、乘号 `*` 和圆周率 `pi` 是怎么来的，但这不妨碍我们理解这个程序。模式匹配会根据值的构造方式进入对应分支，并把值中携带的数据绑定到括号里的名字。例如，`circle(r)` 会把圆的半径绑定到 `r`，`rectangle(x, y)` 会把矩形的长和宽分别绑定到 `x` 和 `y`。

## 结构体

结构体是另一种自定义类型，它和枚举类型不同。

有时，我们需要构造的数据类型只有一种形态，比如表示二维空间中的一个点：

```
enum Point {
  point(Int, Int)
}

let p : Point = point(1, 2)
```

在这种情况下，更方便的做法是使用 Lane 语言的结构体。定义结构体的语法和定义枚举类型类似，只是字段的名字和类型都需要写出来；构造结构体字面量的语法也和构造枚举值类似，只是需要写出每个字段名，并在冒号后面写出对应的值：

```
struct Point {
  x : Int
  y : Int
}

let p : Point = Point::{ x: 1, y: 2 }
```

结构体的使用方式和枚举类型类似，也可以通过模式匹配来访问其字段：

```
fn squared_distance(p : Point) -> Int {
  match p {
    Point::{ x: first, y: second } => first * first + second * second
  }
}
```

当结构体的字段名和模式匹配中想要绑定的名字相同时，可以使用简写：

```
fn squared_distance(p : Point) -> Int {
  match p {
    Point::{ x, y } => x * x + y * y
  }
}
```

除了模式匹配，也可以直接通过点号访问结构体字段：

```
fn squared_distance(p : Point) -> Int {
  p.x * p.x + p.y * p.y
}
```

## 列表和泛型

Lane 语言还支持一种特殊的类型及其对应的字面量：列表。列表是一个有序的元素集合，所有元素的类型必须相同。我们可以用方括号 `[]` 表示列表字面量，列表中的元素用逗号 `,` 分隔。例如：

```
let int_list : List[Int] = [1, 2, 3]

let bool_list : List[Bool] = [true, false, true]
```

可以看到，列表类型 `List` 本身并不是一个具体的类型，而是一个泛型类型。`List[Int]` 表示元素类型为 `Int` 的列表，`List[Bool]` 表示元素类型为 `Bool` 的列表。我们可以用 `List[T]` 表示元素类型为某个类型 `T` 的列表。

自定义的枚举和结构体也可以是这样的泛型类型。例如，我们可以定义一个泛型二叉树类型：

```
enum Tree[T] {
  leaf(T)
  node(Tree[T], Tree[T])
}
```

也可以定义一个简单的装箱类型：

```
struct Box[T] {
  value : T
}
```

我们可以这样理解这个类型：「对于任意类型 `T`，`Box[T]` 表示一个装箱，它承载的值的类型是 `T`。」

因此，我们可以在 `Box[T]` 类型的箱子中放入任意类型的值：

```
let int_box : Box[Int] = Box::{ value: 42 }

let bool_box : Box[Bool] = Box::{ value: true }
```

事实上，列表正是 `Basic.Data.List` 模块中定义的一个泛型枚举类型：

```
module Basic.Data.List

enum List[T] {
  empty()
  cons(T, List[T])
}
```

用方括号表示的列表字面量 `[1, 2, 3]`，实际上是 `cons(1, cons(2, cons(3, empty())))` 的简写形式。

除了结构体和枚举，函数也可以是泛型的。例如，我们可以定义一个函数，让它接受一个列表，并返回该列表的长度：

```
fn[T] length(list : List[T]) -> Int {
  match list {
    empty() => 0
    cons(_, tail) => 1 + length(tail)
  }
}

let int_list : List[Int] = [1, 2, 3]

fn print_length(list : List[Int]) -> Unit ! Io {
  println("length is " + to_string(length(list)))
}
```

上述程序会打印 `length is 3`。我们在第 1 行的函数定义中使用了 `[T]`，表示 `length` 是一个泛型函数，它可以接受任意元素类型的列表作为参数。无论列表中元素的类型是什么，`length` 函数都能正确计算出列表的长度。这样一来，我们就可以用同一个函数计算不同类型列表的长度，而不需要为每种类型都写一个新的函数。

## 递归函数和高阶函数

在打印列表长度的程序中，`length` 函数调用了自身，这样的函数就是递归函数。递归函数是指在函数体内直接或间接调用自身的函数。递归是一种常见的编程技巧，尤其适合处理具有递归结构的数据类型，例如列表和树。

基本上，所有列表相关的操作都适合用递归函数实现。例如，我们可以定义一个函数，用于计算列表中所有整数的和：

```
fn sum(list : List[Int]) -> Int {
  match list {
    empty() => 0
    cons(head, tail) => head + sum(tail)
  }
}
```

假设有一个列表 `[1, 2, 3]`，我们可以观察这个函数的计算过程：

```
  sum([1, 2, 3])
= 1 + sum([2, 3])
= 1 + (2 + sum([3]))
= 1 + (2 + (3 + sum([])))
= 1 + (2 + (3 + 0))
= 6
```

列表有一种常见操作，叫作映射（map）。映射操作会对列表中的每个元素应用一个函数，并返回一个新的列表；新列表中包含原列表每个元素经过函数处理后的结果。例如，我们可以定义一个函数，将列表中的每个整数加倍：

```
fn double_list(list : List[Int]) -> List[Int] {
  match list {
    empty() => empty()
    cons(head, tail) => cons(head * 2, double_list(tail))
  }
}
```

这个函数会返回一个新的列表，其中每个元素都是原列表中对应元素的两倍。例如，`double_list([1, 2, 3])` 会返回 `[2, 4, 6]`。

假设我们有一个用于判断整数是否为偶数的函数：

```
fn is_even(n : Int) -> Bool {
  n % 2 == 0
}
```

我们可以定义一个函数，用于判断列表中每个元素是否为偶数：

```
fn is_even_list(list : List[Int]) -> List[Bool] {
  match list {
    empty() => empty()
    cons(head, tail) => cons(is_even(head), is_even_list(tail))
  }
}
```

`is_even_list([1, 2, 3])` 会返回 `[false, true, false]`。

可以看到，`double_list` 和 `is_even_list` 的结构非常相似。它们都使用递归和模式匹配来处理列表。为了避免重复代码，我们可以定义一个更通用的高阶函数 `map`，它接受一个函数和一个列表作为参数，并返回一个新的列表：

```
fn[T, U] map(f : (T) -> U, list : List[T]) -> List[U] {
  match list {
    empty() => empty()
    cons(head, tail) => cons(f(head), map(f, tail))
  }
}
```

在这个函数中，`T` 和 `U` 是类型参数，分别表示输入列表的元素类型和输出列表的元素类型。`f` 是一个函数，它接受一个类型为 `T` 的参数，并返回一个类型为 `U` 的结果。`map` 函数会对列表中的每个元素应用函数 `f`，并返回一个新的列表。

这种将函数作为参数传递的能力，使我们可以编写更通用、更可复用的代码。我们可以使用 `map` 函数来实现 `double_list` 和 `is_even_list`：

```
fn is_even_list(list : List[Int]) -> List[Bool] {
  map(is_even, list)
}
```

要实现 `double_list`，我们可以定义一个简单的函数 `double`，然后将其传递给 `map`：

```
fn double(n : Int) -> Int {
  n * 2
}

fn double_list(list : List[Int]) -> List[Int] {
  map(double, list)
}
```

当然，由于 `double` 函数非常简单，我们也可以直接使用匿名函数：

```
fn double_list(list : List[Int]) -> List[Int] {
  map(fn(n : Int) -> Int { n * 2 }, list)
}
```

匿名函数本身也是一种表达式，它可以在需要函数的地方直接使用，而不必先为它命名。通常情况下，我们会在需要把函数作为参数传递时使用匿名函数，这样可以减少命名开销，并使代码更加简洁。当然，既然匿名函数是普通表达式，它也可以被绑定到一个名字上，以便在其他地方使用，或者作为返回值返回。

## 上下文解析和运算符重载

我们终于有机会揭开加法运算符 `+` 的神秘面纱了。前面我们使用 `+` 来计算整数的和，也使用 `+` 来拼接字符串、计算浮点数的和。借助前文对泛型的介绍，我们可以把 `+` 理解成一个泛型函数：它的参数类型和返回值类型都依赖于传入的参数类型。它的声明大致可以写成如下形式：

```
fn[T] +(a : T, b : T) -> T { ... }
```

当然，这不是合法的 Lane 语法。在 Lane 中，`+` 不是普通函数名，而是由编译器解析为对 `op_add` 函数的调用。`op_add` 的真正声明大致是这样：

```
module Basic.Ops

import Basic.Builtins.*

pub fn[T] op_add(a : T, b : T, auto op : Add[T]) -> T {
  op.add(a, b)
}

pub offer int_add_ops : Add[Int] = Add::{ add: int_add }
```

也就是说，`op_add` 会调用第三个参数 `op` 中的 `add` 字段来完成真正的加法计算。先不管新的关键字 `auto` 和 `offer`，我们可以看到，`op_add` 函数有一个额外的参数 `op`，它的类型是 `Add[T]`。这个类型的定义如下：

```
module Basic.Ops

struct Add[T] {
  add : (T, T) -> T
}
```

它只是对函数类型的一层包装。也就是说，`op_add` 函数的第三个参数不是函数本身，而是一个包含 `add` 字段的结构体值；这个 `add` 字段才是函数，它接受两个类型为 `T` 的参数，并返回一个类型为 `T` 的结果。

这一切的关键在于，在 `op_add` 被调用的同一个上下文中，必须存在一个类型为 `Add[T]` 的值。这个值会被自动传入 `op_add` 函数的第三个参数。对于整数加法来说，真正执行计算的是 `Basic.Builtins` 模块中的内置函数：

```
module Basic.Builtins

pub let int_add : (Int, Int) -> Int = builtin("%i64_add")
```

这个 `builtin("%i64_add")` 就是我们在前文中提到的内置函数。随后，`Basic.Ops` 模块会把它包装成一个类型为 `Add[Int]` 的值：

```
module Basic.Ops

pub offer int_add_ops : Add[Int] = Add::{ add: int_add }
```

`int_add_ops` 绑定不是用 `let` 声明的，而是用 `offer` 声明的。`offer` 声明的绑定会进入一个「候选上下文」。当函数用 `auto` 声明了某个「自动参数」时，调用该函数的上下文中不必显式传递这个参数，而是可以在「候选上下文」中查找是否存在一个类型匹配的候选值。

也就是说，当我们在某个上下文中调用 `op_add` 函数，也就是使用 `+` 运算符时，如果该上下文中存在一个类型为 `Add[T]` 的值，那么这个值就会被自动传入 `op_add` 的第三个参数。通常情况下，只要我们引入了 `Basic.Ops` 模块中相应的绑定，`int_add_ops` 就会进入「候选上下文」，从而让整数类型的加法运算正常工作。

借助这种方式，`Double` 类型的加法运算也可以用类似方式实现。只要在 `Basic.Builtins` 模块中定义一个 `double_add` 内置函数，并在 `Basic.Ops` 模块中提供一个 `double_add_ops` 绑定，就可以让 `+` 运算符支持浮点数类型的加法运算。

再回顾一次，当我们在程序中使用 `+` 运算符时：

```
fn add_numbers(a : Int, b : Int) -> Int {
  a + b
}
```

编译器会把它当作 `op_add` 函数的调用：

```
fn add_numbers(a : Int, b : Int) -> Int {
  op_add(a, b)
}
```

而 `op_add` 有一个隐式的参数 `op`，它的类型是 `Add[Int]`。编译器会在当前上下文中查找是否存在一个类型为 `Add[Int]` 的值。如果找到了，就会把它传入 `op_add` 函数的第三个参数，从而完成整数加法运算。

```
fn add_numbers(a : Int, b : Int) -> Int {
  op_add(a, b, op=int_add_ops)
}
```

而当我们使用 `+` 运算符来拼接字符串时，编译器会查找类型为 `Add[String]` 的值，从而完成字符串拼接操作。看起来 `op_add` 的行为会根据传入参数的类型而有所不同，但事实上这是两个不同的调用，它们自动补全的参数并不相同。这个特性叫作「上下文解析」（Contextual Resolution），它允许我们在不同上下文中为同一个函数提供不同的默认实现。借助上下文解析，运算符重载就很容易实现：同一个运算符可以在不同类型的操作数上表现出不同的行为。

## 输入/输出和效应

我们在最早的输出字符串程序中是这样定义 `println` 函数的：

```
pub let println : (String) -> Unit ! Io = extern("println")
```

这个绑定使用 `extern("println")` 引用了一个由执行环境提供的外部函数。`extern` 和 `builtin` 不同：`builtin` 表示由编译器直接理解和实现的内置操作，而 `extern` 表示需要在程序加载时由运行时提供的外部符号。`println` 的类型携带 `! Io`，这表明它在返回 `Unit` 的同时可能与外部环境发生可观察的交互。没有效应的 Lane 函数仍然是纯函数；像 `println` 这样会改变控制台内容的函数，则必须在类型中声明 `Io`。

`Io` 和 `Int` 一样，是 Lane 内置的类型，不由 `Basic.Io` 模块声明。它没有可以被程序处理的效应操作，而是统一表示终端、文件、时钟、随机数、环境变量和网络等外部交互。`Basic.Io` 只是一个普通的库模块，它在这里导出了绑定到 `println` 运行时符号的 `println` 函数；编译器不会因为模块名或符号名而给予它特殊待遇。

那么其它效应呢？在 Lane 中，效应和自定义类型一样，也可以由用户自己进行定义：

```
effect Exception {
  raise(String) -> Unit
}
```

这里我们定义了一个异常效应，这是最典型的效应用途。当程序执行到一个我们无法正常处理的情况下，我们就可以抛出一个异常，让调用该程序的程序来处理这类特殊情况。例如，我们有一个从列表中取出第一个元素的函数 `first`，它的返回值类型是列表的元素类型。如果列表中一个元素都没有，这个函数应该返回什么呢？我们可以使用如下枚举类型来表达返回类型：

```
enum Option[T] {
  some(T)
  none()
}
```

但也可以让函数产生一个 `Exception` 效应：

```
fn[T] first(ls : List[T]) -> T ! Exception {
  match ls {
    empty() => raise!("no elements in the list")
    some(head, tail) => head
  }
}
```

产生效应的语法是 `operation!(arguments)`，这个 `operation` 可以是一个定义在自定义效应中的分支，也可以是一个会产生效应的函数。就像 `some` 可以看成一个 `(T) -> Option[T]` 类型的函数一样，`raise` 也可以看成是一个 `(String) -> Unit ! Exception` 类型的函数。当一个表达式内出现了效应，那么整个表达式要么会把这个效应传递到外侧，例如函数的签名处；要么需要用一个 `handle-with` 表达式来处理效应。

在上面的例子中，`first` 函数只是产生了 `raise` 效应而没有处理，因此函数的执行就会同样产生，或者严格地说，可能产生 `Exception` 效应。我们可以在调用处来处理这个效应。处理效应的方式和枚举值的模式匹配类似，根据不同的效应操作进入不同的分支。由于 `Exception` 只有一个操作 `raise`，所以只有一个分支：

```
fn print_first_integer() -> Unit ! Io {
  let list = [1, 2, 3, 4]
  handle first(list) with {
    raise(msg, _resume) => println("got exception: " + msg)
  } final v {
    println("first integer is: " + to_string(v))
  }
}
```

注意到两个新事物：`raise` 模式匹配处，还有第二个没有被使用的参数 `resume`；以及，除了用 with 块匹配所有 `Exception` 类型的操作，还需要一个 `final` 用于函数调用正常返回时的兜底。例如，在调用上面这个函数，就会打印 `first integer is: 1`。也就是说，`first([1, 2, 3, 4])` 正常返回的 `1` 被绑定到了 final 块的名字 `v`。

以下程序表达这样一种逻辑：当调用 `first` 函数时遇到 `raise`，以 0 作为默认调用结果。

```
handle first(list) with {
  raise(_msg, _resume) => 0
} final v {
  v
}
```

但是，效应的处理逻辑经常和产生效应不在同一个程序。例如，很有可能一个巨大的程序经过很深层次的嵌套，最终调用了 `first`，但是 `first` 却并不知道这个调用者打算如何处理效应。我们可以假设这样一种情况：

```
handle {
  ...
  let fst = first(some_list)
  ...
} with {
  ...
} final v {
  ...
}
```

会产生效应的 `first` 函数只是千万行程序中的一行。此时，我们只是希望 `first` 抛出异常时会把 0 绑定到 `fst`，而不希望整个表达式的结果是 0. 我们可以使用 `resume` 来做到这一点：

```
handle {
  ...
  let fst = first(some_list)
  ...
} with {
  raise(_msg, resume) => resume(0)
} final v {
  ...
}
```

`resume` 是一个函数，代表程序执行到 `first` 调用处的延续（continuation）。当调用 `resume(0)` 时，程序会回到执行 `first` 的地方，以 `0` 为值填充 `first(some_list)` 表达式，继续剩余程序的执行。

## 总结

到目前为止，我们已经介绍了 Lane 语言中大部分常用的语言要素。借助这些要素，已经可以编写大型程序。后续章节会逐一展开介绍各种基本要素。
