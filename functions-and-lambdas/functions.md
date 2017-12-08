函数
===

函数声明
---

Kotlin 使用关键词 `fun` 声明一个函数：

```kotlin
fun double(x: Int): Int {
    return 2 * x
}
```

函数使用
---

普通函数调用方式（the traditional approach）：

```kotlin
val result = double(2)
```

成员函数调用方式（the dot notation）：

```kotlin
Sample().foo()  // create instance of class Sample and call foo
```
### 参数

参数的定义符合 **Pascal notation**，例如：`name: type`。参数之间通过都好分隔，每个参数都必须指明类型。

```kotlin
fun powerOf(number: Int, exponent: Int) {
  // ...
}
```

### 默认参数

函数参数可以有默认值，因此调用时可以省略对应的参数。这样可以减少函数重载。

```kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) {
  // ...
}
```

默认值定义在类型右侧的 `=` 之后。

重写函数总是使用同样的默认参数。所以重写一个带默认参数的函数时，函数签名中必须要删除默认值：

```kotlin
open class A {
    open fun foo(i: Int = 10) { /* ... */ }
}

class B : A {
    // no default value allowed
    override fun foo(i: Int) { /* ... */ }
}
```

如果默认参数后面的参数没有默认值，只能通过命名参数（named arguments）的方式使用默认值：

```kotlin
fun foo(bar: Int = 0, baz: Int) { /* ... */ }

// The default value bar = 0 is used
foo(baz = 1)
```

如果把 lambda 作为最后一个参数传递给函数（位于圆括号之外），默认参数无需传值：

```kotlin
fun foo(bar: Int = 0, baz: Int = 1, qux: () -> Unit) { /* ... */ }

foo(1) { println("hello") } // Uses the default value baz = 1
foo() { println("hello") } // Uses the default value bar = 0 and baz = 1
```

### 命名参数

函数调用时可以指定参数名。当函数有很多参数或者带默认参数时，这个特性很方便。

例如下面的函数：

```kotlin
fun reformat(str: String,
             normalizeCase: Boolean = true,
             upperCaseFirstLetter: Boolean = true,
             divideByCamelHumps: Boolean = false,
             wordSeparator: Char = ' ') {
  // ...
}
```

可以用默认默认值调用：

```kotlin
reformat(str)
```

如果不使用参数默认值：

```kotlin
reformat(str, true, true, false, '_')
```

如果使用命名参数，代码会变得更可读：

```kotlin
reformat(str,
    normalizeCase = true,
    upperCaseFirstLetter = true,
    dividebyCamelHumps = false,
    wordSeparator = '_'
)
```

如果不需要所有的参数：

```kotlin
reformat(str, wordSeparator = '_')
```

如果同时使用位置参数和命名参数来调用函数，那么位置参数必须位于命名参数之前。例如，`f(1, y = 2)` 合法，`f(x = 1, 2)` 不合法。

可变数量参数（`vararg`）可以利用 `spread` 操作符通过命名的形式传递：

```kotlin
fun foo(vararg strings: String) { /* ... */ }

foo(strings = *arrayOf("a", "b", "c"))
```

注意，命名参数的语法无法用于 java 函数，因为 Java 字节码不一定保有参数名。

### 返回 Unit 的函数

如果函数的返回值无用，返回值类型可以为 `Unit`。`Unit` 类型只有一个值 - `Unit`。无需显示返回这个值。

```kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")

    // `return Unit` or `return` is optional
}
```

`Unit` 的返回值可以不用声明。上面的代码等价于：

```kotlin
fun printHello(name: String?) {
    // ...
}
```

### 单表达式函数
如果函数只返回一个单表达式的值，那么可以省略花括号，函数体放在等号右侧。

```kotlin
fun double(x: Int): Int = x * 2
```

如果编译器能够推导出返回值类型，无需显示声明。

```kotlin
fun double(x: Int) = x * 2
```

### 显示的返回类型
有 block body 的函数必须显示指明返回值类型，除非返回 `Unit`。Kotlin 不会推测返回值类型，因为函数体的控制逻辑可能比较复杂，返回值类型不明显。

### 可变参数（Varargs）
函数参数（通常是最后一个）可用 `vararg` 修饰：

```kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts)   // ts is an Array
        result.add(t)
    return result
}
```

可以传递任意数量的参数：

```kotlin
val list = asList(1, 2, 3)
```

在函数内，类型为 `T` 的 `vararg` 参数被视作一个 `T` 的数组，也就是说，变量 `ts` 的类型是 `Array<out T>`。

`vararg` 只能标记于一个参数，如果这个参数不是参数列表的最后一个，那么它之后的参数可以通过命名参数的语法传值，或者，如果参数是函数类型，可以在括号之外传递一个 lambda。

调用 `vararg` 函数是，可以一个一个传参，例如，`asList<1, 2, 3>`，或者，如果要传递一个数组的内容，可以用 `spread` 操作符（数组前面加个 `*`）：

```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

### 中缀符号（infix notation）
当满足如下条件时，可以使用中缀符号调用函数：

- 函数是成员函数或扩展函数
- 函数只有一个参数
- 函数用 `infix` 标记

```kotlin
// Define extension to Int
infix fun Int.shl(x: Int): Int {
    // ...
}

// call extension function using infix notation
1 shl 2

// is the same
1.shl(2)
```

函数 Scope
---
Kotlin 的函数可以直接定义在代码文件的最外层，也就是说，无需像 Java、C# 或 Scala 那样必须把函数定义在类里面。初此之外，Kotlin 的函数可以声明为本地函数、成员函数和扩展函数。

### 局部函数

Kotlin 支持本地函数，也就是说，函数可以嵌套。

```kotlin
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: Set<Vertex>) {
        if (!visisted.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }

    dfs(graph.vertices[0], HashSet())
}
```

局部函数可以方位外部函数（闭包）的局部变量，所以，上例中的 `visited` 可用局部变量表示：

```kotlin
fun dfs(graph: Graph) {
    val visited = HashSet<Vertext>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
        }
    }

    dfs(graph.vertices[0])
}
```

### 成员函数

成员函数定义在类或者对象的内部：

```kotlin
class Sample() {
    fun foo() { print("Foo") }
}
```

成员函数通过点来调用：

```kotlin
Sample().foo()  // creates instance of class Sample and calls foo
```

更多关于类和重写的姿势可参考 [类和继承](classes-and-objects/classes-and-inheritance.md)。

泛型函数
---
函数可以有泛型参数，用尖括号表示：

```kotlin
fun <T> singletonList(item: T) List<T> {
    // ...
}
```

内联函数
---
请前往[内联函数](functions-and-lambdas/inline-functions.md)。

扩展函数
---
请前往[扩展函数](classes-and-objects/extensions.md)。

高阶函数和 lambda
---
请前往[高阶函数和lambda](functions-and-lambdas/lambdas.md)。

尾递归函数（Tail recursive functions）
---

Kotlin 支持函数式编程的[尾递归](https://en.wikipedia.org/wiki/Tail_call)，尾递归可以代替循环来实现某些算法，而且不会导致栈溢出。当一个函数用 `tailrec` 来修饰并且满足形式，编译器会把递归调用优化成一个快速且高效的循环。

```kotlin
tailrec fun findFixPoint(x: Double = 1.0): Double
    = if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))
```

上面的代码用于计算一个数学常量 —— cosine 的 fixpoint。实现方式只是一直调用 `Math.cos` 直到计算结果不再变，最后产生的值是 0.7390851332151607。以上代码的传统的写法如下：

```kotlin
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (x == y) return x
        x = y
    }
}
```

`tailrec` 修饰符要求函数的最后一个操作必须是调用自己。如果递归调用之后还有其他执行代码，或者是 try/catch/finally 的代码块，都不能使用尾递归。当前只有 JVM 的后端（backend）支持尾递归。
