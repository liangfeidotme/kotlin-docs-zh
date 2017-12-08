高阶函数和lambda
===

高阶函数
---
高阶函数是一个函数，它可以接收函数类型的参数，也可以返回一个函数。`lock()` 是一个很好的例子，它接收一个 lock 对象和一个函数，获取 lock，执行函数，释放 lock：

```kotlin
fun <T> lock(lock: Lock, body: () -> T): T {
    lock.lock()
    try {
        return body()
    }
    finally {
        lock.unlock()
    }
}
```

上面的代码中，`body` 是一个函数类型：`() -> T`，无参并且返回值类型是 `T` 的函数。在 try-block 中调用，被 `lock` 保护，由 `lock()` 提供返回值。

`lock()` 可接收另一个函数作为它的参数：

```kotlin
fun toBeSynchronized() = sharedResource.operation()
val result = lock(lock, ::toBeSynchronized)
```

传入一个 lambda 表达式会更简单：

```kotlin
lock(lock, { sharedResource.operation() })
```

下文有关于 lambda 表达式更详细的描述，先看一下概览：

- lambda 表达式由花括号包裹
- 参数在 `->` 之前声明（参数类型可省略）
- body 在 `->` 之后（如果有的话）

如果一个函数的最后一个参数是一个函数，我们可以在函数括号之外传入一个 lambda 表达式：

```kotlin
lock (lock) {
    sharedResource.operation()
}
```

另一个高阶函数的例子是 `map()`：

```kotlin
fun <T, R> List<T>.map(transform: (T) -> R): List<R> {
    val result = arrayListOf<R>()
    for (item in this)
        result.add(transform(item))
    return result
}
```

用法如下：

```kotlin
val doubled = inis.map { value -> value * 2 }
```

如果 lambda 是函数唯一的参数，括号可以省略。

### it: 单个参数的隐式名字

另外一个比较有用的约定是：如果一个函数只有一个参数，那么参数的声明连带 `->` 可以省略，参数名用 `it` 指代：

```kotlin
ints.map { it * 2 }
```

这些约束允许使用 LINQ-style 的代码：

```kotlin
strings.filter { it.length == 5 }.sortedBy { it }.map { it.toUpperCase() }
```

### 下划线指代无用变量（1.1开始）

如果 lambda 的参数无用，可以用下划线代替：

```kotlin
map.forEach { _, value -> println("$value!") }
```

### lambda 解构

参考解构声明。

内联函数
---
使用内联函数可以提升高阶函数性能。

lambda 表达式和匿名函数
---
一个 lambda 表达式或匿名函数是一个 **function literal**，例如，一个函数没有声明，但是直接作为表达式被传递。如下：

```kotlin
max(strings, { a, b -> a.length < b.length })
```

`max` 是高阶函数，它的第二个参数是一个函数。第二个参数是一个表达式，这个表达式本身是一个函数（function literal）。其等价于如下代码：

```kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

### 函数类型

对于接收函数作为参数的函数，我们需要为那个参数指定函数类型。例如，上述提到的 `max`，其定义如下：

```kotlin
fun <T> max(collection: Collection<T>, less: (T, T) -> Boolean): T? {
    var max: T? = null
    for (it in collection)
        if (max == null || less(max, it))
            max = it
    return max
}
```

参数 less 的类型是 `(T, T) -> Boolean`，两个 T 类型参数，返回值是 `Boolean`：如果第一个参数小于第二个，返回 true。

函数体的第四行，`less` 是一个函数调用，传入了两个类型为 `T` 的参数。

函数类型的写法如上，也可以用命名参数来增加可读性：

```kotlin
val compare: (x: T, y: T) -> Int = ...
```

如果要声明一个可为空的函数类型的变量，可以用括号包裹整个函数类型，然后在后面加一个问号：

```kotlin
var sum: ((Int, Int) -> Int)? = null
```

### lambda 表达式的语法

lambda 表达式（i.e. literals of function types）完整的语法形式如下：

```kotlin
val sum = { x: Int, y: Int -> x + y }
```

lambda 表达式由大括号包裹，在完整的语法格式下，参数声明在括号内，并且可以带有可选的类型标记，lambda 体在 `->` 符号之后。如果编译器推测出来返回值类型不是 `Unit`，lambda 体内的最后一个表达式（也有可能是一个单表达式，也就说只有一个表达式）会被当做返回值。

如果去掉所有的可选标记，剩下的代码如下所示：

```kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

lambda 只有一个参数是很常见的。如果 Kotlin 本身能够识别出 lambda 的函数签名，我们就不需要手动去声明这个唯一的参数，因为 Kotlin 会隐士地帮我们把这个参数声明为 `it`：

```kotlin
ints.filter { it > 0 }
```

我们可以用 [qualified return](../basics/returns-and-jumps.md) 的语法从 lambda 中显示地返回一个值。否则，最后一个表达式的值会被隐式返回。因此，下面两个代码片段是等价的：

```kotlin
ints.filter {
    val shouldFilter = it > 0
    shouldFilter
}

ints.filter {
    val shouldFilter = it > 0
    return@filter shoudFilter
    }
```

注意：如果一个函数的最后一个参数是函数类型，那么使用 lambda 表达式的实参（argument）可以在实参列表的括号之外传递。这个语法叫 *callSuffix*。

### 匿名函数

上面所展示的 lambda 表达式的语法缺少了一项能力：无法指定返回值类型。多数情况下，类型可以自动推导出，所以也没有必要指定。但是，如果一定要指定类型的话，可以利用另一个语法：*匿名函数*。

```kotlin
fun(x: Int, y: Int): Int = x + y
```

除了没有函数名之外，匿名函数的声明和常规函数并无二致。函数体既可以是上例的表达式（expression）也可以是区块（block）：

```kotlin
fun(x: Int, y: Int): Int {
    return x + y
}
```

形参和返回值类型的指定方式与常规函数一致，唯一不同的是，能够根据上下文推导出的参数类型可以直接省略掉。

```kotlin
ints.filter(fun(item) = item > 0)
```

匿名函数的返回值类型推导和常规函数一样：

- 如果函数体是一个表达式（expression），返回类型可以自动推导
- 如果函数体是一个区块（block），返回必须显示指定（或者假定为 `Unit`）

注意：匿名函数作为参数时必须在圆括号内。在圆括号外部传参的简化语法只适用于 lambda 表达式。

lambda 表达式和匿名函数的另一个不同体现在 **non-local return** 的行为。不带标签的 `return` 语句返回的是带有 `fun` 关键字的函数。这就意味着 lambda 表达式内的 `return` 返回的是它的外围函数（enclosing function）。那么匿名函数内的 `return` 返回的是这个匿名函数本身（因为匿名函数也需要 `fun` 关键字修饰）。

### 闭包

lambda 表达式或者匿名函数（以及局部函数和对象表达式）可以访问它的闭包，例如，外围区域定义的变量。与 Java 不同的是，在闭包内捕获（captured）的变量可以被修改：

```kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
```

### 带接收者的 function literal

Kotlin 可以为 `function literal` 的调用指定一个 *receiver* 对象。在 function literal 的内部，可以不带任何修饰符地访问接收者对象的方法。这点和扩展函数类似，在函数体也可以访问接收者对象的成员。最重要的用途示例之一是 [Type-safe Groovy-style builders](../basics/type-safe-builders.md)。

这种 function literal 的类型是带 receiver 的函数类型：

```kotlin
sum: Int.(other: Int) -> Int
```

function literal 的调用就好像它是接收者对象的一个方法：

```kotlin
1.sum(2)
```

匿名函数的语法允许我们直接指定 function literal 的接收者类型。因此，我们可以事先定义一个待接收者的函数变量，以备后用。

```kotlin
val sum = fun Int.(other: Int): Int = this + other
```

// TODO

```kotlin
val represents: String.(Int) -> Boolean = { other -> toIntOrNull() == other }
println("123".represents(123)) // true

fun testOperation(op: (String, Int) -> Boolean, a: String, b: Int, c: Boolean) = 
    assert(op(a, b) == c)

testOperation(represents, "100", 100, true) // OK
```

> 上例中，`represents` 是一个类型为 `String.(Int) -> Boolean` 的 lambda 变量。
> 它的值是 `{ other -> toIntOrNull() == other }`：
>   * 大括号表示 lambda 的定义
>   * lambda 的 receiver object 的类型是 `String`
>   * toIntOrNull() 是 `String` 的方法，调用对象是 receiver object。


lambda 表达式可用作带 receiver 的 function literal（前提是 receiver 的类型可以通过上下文推导出）。

```kotlin
class HTML {
    fun body() { ... }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()   // Create the receiver object
    html.init()         // pass the receiver object to the lambda
    return html
}

html {      // lambda with receiver begins here
    body()  // calling a method on the receiver object
}
```

