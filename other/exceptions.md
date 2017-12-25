异常
===

异常类
---

Kotlin 的所有异常类都继承自 `Throwable`。每个异常都包含消息，堆栈和可选的异常原因。

使用 `throw` 表达式抛出异常对象：

```kotlin
throw MyException("Hi There!")
```

使用 `try` 表达式捕获异常：

```kotlin
try {
    // some code
}
catch (e: SomeException) {
    // handler
}
finally {
    // optional finally block
}
```

可能会有一个或多个 `catch` 块。`finally` 块可以省略。但是至少需要一个 `catch` 或 `finally` 块。

`try` 是表达式
---

`try` 是表达式，它可以有返回值：

```kotlin
val a: Int? = try { parseInt(input) } catch (e: NumberFormatException) { null }
```

`try` 表达式的返回值要么是 `try` 块的最后一个表达式，要么是 `catch` 快（可以有多个）的最后一个表达式。`finally` 块的内容不会影响表达式的值。

Checked Exceptions
---
Kotlin 没有 checked exceptions。原因有很多，我们通过一个例子看一下。

下面是 JDK 提供的一个接口，`StringBuilder` 实现了它：

```java
Appendable append(CharSequence csq) throw IOException;
```

签名说明了每次给某个东西（`StringBuilder`、某种 log、控制台等）附加字符串时，必须捕获所有的 `IOException`。因为可能会有 IO 操作（`Writer` 也实现了 `Appendable`），如果到处都是下面这样的代码：

```java
try {
    log.append(message)
} catch (IOException e) {
    // Must be safe
}
```

这样做不好，参见 Effective Java 第 65 条：*不要忽视异常*.

Bruce Eckel says in [Does Java need Checked Exceptions?](http://www.mindview.net/Etc/Discussions/CheckedExceptions):

> Examination of small programs leads to the conclusion that requiring exception specifications could both enhance developer productivity and enhance code quality, but experience with large software projects suggests a different result – decreased productivity and little or no increase in code quality.

Other citations of this sort:
- [Java's checked exceptions were a mistake](http://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html) (Rod Waldhoff)
- [The Trouble with Checked Exceptions](http://www.artima.com/intv/handcuffs.html) (Anders Hejlsberg)

`Nothing` 类型
---

`throw` 在 Kotlin 中是一个表达式，所以可用作一个 Elvis expression 的一部分：

```kotlin
val s = person.name ?: throw IllegalArgumentException("Name required")
```

`throw` 表达式的类型是一个特殊类型 - `Nothing`。这个类型没有值，只用于标记永远不会执行到的代码位置。我们可用 `Nothing` 标记一个永远不会 return 的函数：

```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

当调用 `fail` 时，编译器会知道调用完成之后执行不会继续。

```kotlin
val s = person.name ?: fail("Name required")
println(s)  // 's' is known to be initialized at this point
```

还有一个碰到这个类型的场景是类型推断（type inference）。这个类型的可空变体是 `Nothing?`，只有一个取值 `null`。如果用 `null` 初始化一个推断类型的值而且没有其他信息可以决定具体的类型，编译器的推断结果是 `Nothing?` 类型：

```kotlin
val x = null            // 'x' has type `Nothing?`
val l = listOf(null)    // 'l' has type `List<Nothing>`
```

Java 互操作
---
具体可参考 [Java 互操作](../java-interop/calling-java-from-kotlin.md)。

